---
name: DoH クエリの書き方・デバッグ方法
description: DNS over HTTPS (DoH) のクエリ構築やデバッグについて相談されたときに使用する。
---

# DoH クエリの書き方・デバッグ方法

## このスキルの出典(RFC)

- **RFC 8484** — DNS over HTTPS。GET の `dns=` パラメータは **base64url(パディング無し)**。POST はボディを `application/dns-message` で送る。
- **RFC 1035 §4.1.1 / §4.1.2** — メッセージ wire format:12バイトヘッダ + 問い合わせ部。
- **RFC 4648 §5** — base64url の定義。

## DoH の2形式

| 方式 | クエリの運び方 | 備考 |
|---|---|---|
| **GET** | `?dns=<base64url>` | キャッシュキーにクエリ文字列が乗るので CDN/Worker キャッシュで区別しやすい |
| **POST** | ボディに `application/dns-message` | ボディはキャッシュキーに乗らない(Worker 側で hash 等が必要) |

## クエリの wire format(RFC 1035)

### ヘッダ(12バイト)
| offset | 内容 |
|---|---|
| 0-1 | ID(任意) |
| 2-3 | Flags(`0x0100` = RD=1) |
| 4-5 | QDCOUNT(問い合わせ数) |
| 6-7 | ANCOUNT=0 |
| 8-9 | NSCOUNT=0 |
| 10-11 | ARCOUNT=0 |

### 問い合わせ部
QNAME は「ラベル長(1バイト)+ ASCII」の繰り返しで、最後に根 `0x00`。
`example.com A IN` の例:
```
\x07example\x03com\x00  → QNAME
\x00\x01                 → QTYPE = A (1)
\x00\x01                 → QCLASS = IN (1)
```
QTYPE 例:A=1, NS=2, CNAME=5, SOA=6, PTR=12, MX=15, TXT=16, AAAA=28(RFC 3596)。

## クエリ構築(例)
```bash
# QNAME をラベル長付きで変換するヘルパー
hexqname() { local out=""; for label in $(echo "$1" | tr '.' ' '); do
  out+="\\x$(printf '%02x' "${#label}")${label}"; done; out+="\\x00"; printf '%s' "$out"; }

# example.com A のクエリ → base64url
Q=$(printf '\x00\x01\x01\x00\x00\x01\x00\x00\x00\x00\x00\x00%s\x00\x01\x00\x01' \
      "$(hexqname example.com)" | base64 -w0 | tr '+/' '-_' | tr -d '=')
```

## デバッグ

### ヘッダまで見る curl(GET)
```bash
curl -s -D - -o /dev/null --max-time 5 \
     -H 'accept: application/dns-message' \
     "https://doh.example.com/dns-query?dns=$Q"
```
- `-D -` でヘッダ出力、`-o /dev/null` でボディは捨てる。
- **2回叩いて `cf-cache-status: MISS` → `HIT` になればキャッシュ動作確認。**

### 注意:GET でメッセージをボディに乗せない
GET に `--data-binary @-` でボディを渡すと **origin が invalid request を返す**。GET は必ず `?dns=` に base64url で乗せる。POST で送るなら `-X POST -H 'content-type: application/dns-message' --data-binary @-`。

### kdig
- `kdig @<host> -p 443 +https +tls-hostname=<host> <name> A` など。DoH 専用は `+https`(HTTPS 互換)。
- `+dohurl=` オプションは**非対応**なバージョンがあるので、無理に使わず curl で代替する。

### kdig の証明書検証(デフォルトは検証しない)
kdig の TLS 検証は **opt-in** であり、`+tls` / `+https` だけでは RFC 7858 §4.1 の「Opportunistic privacy profile」、つまり**証明書を検証しない**。

| 目的 | オプション |
|---|---|
| **検証を有効化**(既定の CA ストア) | `+tls-ca`(引数なしでシステム CA を使用) |
| 検証を有効化(CA を指定) | `+tls-ca=/path/to/ca.pem`(複数回指定可) |
| 検証を無効化 | `+no-tls-ca`(または付けない) |
| 検証対象ホスト名を指定 | `+tls-hostname=<STR>`(未指定なら対象サーバ名で厳密認証) |
| SNI のみ設定(検証と独立) | `+tls-sni=<STR>` |
| SPKI ピン留め(RFC 7858 §4.2) | `+tls-pin=<base64 of SHA-256 SPKI>`(複数回指定可) |

例:
```
# システム CA で検証しつつ DoH
kdig @8.8.8.8 +https +tls-ca +tls-hostname=dns.google example.com A

# ピン留めで検証
kdig -d @185.49.141.38 +tls-ca +tls-host=getdnsapi.net \
     +tls-pin=foxZRnIh9gZpWnl+zEiKa0EJ2rdCGroMWm02gaxSc9S= soa example.com.
```
出典: kdig(1) man page(Knot DNS 各バージョン共通)。

### 証明書そのものの診断は openssl で
kdig の `+tls-ca` は合否のみを返し、失敗理由(チェーン欠落・期限切れ・SAN 不一致)は語らない。
原因の診断は `openssl s_client` で証明書を直接見る。

```bash
# 概要(subject / issuer / 有効期限 / SAN)
openssl s_client -connect doh.example.com:443 -servername doh.example.com </dev/null 2>/dev/null \
  | openssl x509 -noout -subject -issuer -dates -ext subjectAltName

# チェーン検証結果(0 (ok) = システム CA で通過)
openssl s_client -connect doh.example.com:443 -servername doh.example.com </dev/null 2>&1 \
  | grep -E "^depth|verify|Verify return"

# 中間証明書を含む全チェーン
openssl s_client -connect doh.example.com:443 -servername doh.example.com -showcerts </dev/null

# 期限早期警告(24時間以内に切れると非ゼロ終了)
echo | openssl s_client -connect doh.example.com:443 -servername doh.example.com 2>/dev/null \
  | openssl x509 -checkend 86400 -noout
```

private origin を Worker 結線前に単独検証するには、IP 宛て + SNI 指定を使う(curl の `--resolve` 相当):
```bash
openssl s_client -connect 10.0.10.53:443 -servername doh.example.com </dev/null 2>&1 | grep "Verify return"
```
これで `Verify return code: 0 (ok)` を確認できれば、Workers fetch の 500 は証明書以外の要因に絞れる。

## 403 の切り分け

| 状況 | 原因 | 対処 |
|---|---|---|
| `403 Forbidden` + `cf-access-*` ヘッダ / `location: ...cloudflareaccess.com` | Access がブロック | サービストークンを発行して `cf-access-client-id/secret` ヘッダを付ける、または Access ポリシーで許可 |
| 一時的な中段応答「Just a moment」 | custom domain の SSL 発行中 | 数分待つ |
| カスタムドメインで安全ルールが介入 | Security rule(チャレンジ等) | 対象 path/IP を除外 |
| deploy 時 403 | token 権限不足 | `Zone > Workers Routes: Edit` + `Zone > DNS: Edit` を token に追加 |

## Workers で DoH を公開する際の要点

- **公開クライアントの TLS は workers.dev / custom domain の証明書で終端**される。origin を https にする必要は必ずしも無い。
- **Workers の `fetch` は TLS 検証を無効化できない**。self-signed な origin を `https://` で叩くと cert エラーで 500 になる。公開 CA 証明書が使える origin を hostname(SNI)で参照するのが正解。
- **Private IP に hostname で向けたい(= `curl --resolve` 相当)**:
  - VPC Network binding + Mesh:mesh node に **hostname route** を張り、**node 側の `/etc/hosts` に `IP hostname`** を書く。dashboard の hostname route フォームに IP 欄は無い(それは node 側で解決する)。
  - hostname routing は **node が MASQUE(2026.6.822.0 以上)必須**。WireGuard のプロファイルだと動かない。
- **キャッシュ**:`caches.default`(Cache API)は **custom domain でのみ機能**(workers.dev では効かない。`.pages.dev` は Pages Functions で効く)。Workers Cache(`ctx.cache`)は workers.dev でも動く。
- GET のクエリは `?dns=` ごとに別キャッシュキー。POST はボディを SHA-256 してキーに混ぜる。TTL は応答の DNS 最小 TTL をパースして `s-maxage` に反映するのが正しい。
