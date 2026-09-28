---
name: publish-cot
description: 会話の文脈をHTMLとMarkdownのプレビューサイトとして公開するときに使用する。
---

# プレビューサイト公開

## 目的

会話の文脈から単一レポートのサイトを組み立て
プレビューとして公開する
未認証なら一時アカウントに上げ
認証済みなら自分のアカウントに上げること
`/report` でHTMLを返し
`/report.md` でMarkdown原文を返す
実行者によらず同じ構成と振る舞いにすること

## 成果物の固定構成

作業ディレクトリ直下に次の4ファイルを作る
ファイル名と配置は変えないこと

```
<dir>/
  wrangler.jsonc
  src/index.js
  public/report.md
  public/index.html
```

## 手順1 設定を複写する

次の内容を一字一句そのまま `wrangler.jsonc` に置く
`name` は仮置きでよくデプロイ時の `--name` が優先される

```jsonc
{
  "name": "preview-site",
  "main": "src/index.js",
  "compatibility_date": "2026-09-25",
  "observability": {
    "enabled": true
	},
  "cache": {
    "enabled": true
  },
  "assets": {
    "directory": "./public",
    "binding": "ASSETS",
    "run_worker_first": true,
    "not_found_handling": "single-page-application"
  }
}
```

## 手順2 Workerを複写する

次の内容を一字一句そのまま `src/index.js` に置く
ルーティングと変換器は改変しないこと

```js
export default {
  async fetch(request, env) {
    const url = new URL(request.url);
    if (url.pathname === "/report.md") {
      const res = await env.ASSETS.fetch(new Request(new URL("/report.md", url)));
      if (!res.ok)
        return new Response("not found", {
          status: 404,
          headers: { "cache-control": "no-store" },
        });
      return new Response(await res.text(), {
        headers: {
          "content-type": "text/markdown; charset=utf-8",
          "cache-control": "public, max-age=604800",
        },
      });
    }
    if (url.pathname === "/report" || url.pathname === "/report/") {
      const res = await env.ASSETS.fetch(new Request(new URL("/report.md", url)));
      if (!res.ok)
        return new Response("not found", {
          status: 404,
          headers: { "cache-control": "no-store" },
        });
      return new Response(renderPage(await res.text()), {
        headers: {
          "content-type": "text/html; charset=utf-8",
          "cache-control": "public, max-age=604800",
        },
      });
    }
    return env.ASSETS.fetch(request);
  },
};

function escapeHtml(s) {
  return s
    .replace(/&/g, "&amp;")
    .replace(/</g, "&lt;")
    .replace(/>/g, "&gt;")
    .replace(/"/g, "&quot;");
}

function inline(md) {
  let out = escapeHtml(md);
  out = out.replace(/`([^`\n]+)`/g, "<code>$1</code>");
  out = out.replace(/\*\*([^*]+)\*\*/g, "<strong>$1</strong>");
  out = out.replace(/(^|[^*\w])\*([^*\n]+)\*/g, "$1<em>$2</em>");
  out = out.replace(
    /\[([^\]]+)\]\(([^)\s]+)(?:\s+"[^"]*")?\)/g,
    '<a href="$2">$1</a>',
  );
  return out;
}

function mdToHtml(md) {
  const lines = md.replace(/\r\n?/g, "\n").split("\n");
  const html = [];
  let inCode = false;
  let codeLang = "";
  let codeBuf = [];
  let inList = false;
  let seenQuote = false;
  let i = 0;
  const flushList = () => {
    if (inList) {
      html.push("</ul>");
      inList = false;
    }
  };
  while (i < lines.length) {
    const line = lines[i];
    const fence = line.match(/^```(\S*)\s*$/);
    if (fence) {
      if (!inCode) {
        flushList();
        inCode = true;
        codeLang = fence[1];
        codeBuf = [];
      } else {
        inCode = false;
        const cls = codeLang ? ` class="language-${codeLang}"` : "";
        html.push(`<pre><code${cls}>${escapeHtml(codeBuf.join("\n"))}</code></pre>`);
      }
      i++;
      continue;
    }
    if (inCode) {
      codeBuf.push(line);
      i++;
      continue;
    }
    if (/^\s*$/.test(line)) {
      flushList();
      i++;
      continue;
    }
    const heading = line.match(/^(#{1,6})\s+(.*)$/);
    if (heading) {
      flushList();
      const level = heading[1].length;
      html.push(`<h${level}>${inline(heading[2])}</h${level}>`);
      i++;
      continue;
    }
    if (/^\|.*\|\s*$/.test(line) && i + 1 < lines.length && /^\|[\s:|-]+\|\s*$/.test(lines[i + 1])) {
      flushList();
      const cells = (row) =>
        row
          .trim()
          .replace(/^\||\|$/g, "")
          .split("|")
          .map((c) => `<td>${inline(c.trim())}</td>`)
          .join("");
      const headerCells = line
        .trim()
        .replace(/^\||\|$/g, "")
        .split("|")
        .map((c) => `<th scope="col">${inline(c.trim())}</th>`)
        .join("");
      html.push(`<table><thead><tr>${headerCells}</tr></thead><tbody>`);
      i += 2;
      while (i < lines.length && /^\|.*\|\s*$/.test(lines[i])) {
        html.push(`<tr>${cells(lines[i])}</tr>`);
        i++;
      }
      html.push("</tbody></table>");
      continue;
    }
    const quote = line.match(/^>\s?(.*)$/);
    if (quote) {
      flushList();
      const qlines = [];
      while (i < lines.length) {
        const m = lines[i].match(/^>\s?(.*)$/);
        if (!m) break;
        qlines.push(m[1]);
        i++;
      }
      while (qlines.length && /^\s*$/.test(qlines[qlines.length - 1])) qlines.pop();
      let cls = "";
      if (!seenQuote || (qlines[0] || "").trim() === "[!prompt]") cls = ' class="prompt"';
      if ((qlines[0] || "").trim() === "[!prompt]") qlines.shift();
      seenQuote = true;
      html.push(`<blockquote${cls}>${qlines.map((l) => inline(l)).join("<br>")}</blockquote>`);
      continue;
    }
    const item = line.match(/^[-*]\s+(.*)$/);
    if (item) {
      if (!inList) {
        html.push("<ul>");
        inList = true;
      }
      html.push(`<li>${inline(item[1])}</li>`);
      i++;
      continue;
    }
    flushList();
    const para = [line.trim()];
    i++;
    while (
      i < lines.length &&
      !/^\s*$/.test(lines[i]) &&
      !/^(#{1,6}\s|```|[-*]\s+\|)/.test(lines[i]) &&
      !/^\|.*\|\s*$/.test(lines[i]) &&
      !/^>/.test(lines[i])
    ) {
      para.push(lines[i].trim());
      i++;
    }
    html.push(`<p>${inline(para.join(" "))}</p>`);
  }
  flushList();
  return html.join("\n");
}

function renderPage(md) {
  const titleMatch = md.match(/^#\s+(.*)$/m);
  const title = titleMatch ? titleMatch[1].replace(/[#*`[\]]/g, "") : "report";
  return `<!doctype html>
<html lang="ja">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<meta name="color-scheme" content="light dark">
<title>${escapeHtml(title)}</title>
<style>:root{color-scheme:light dark}html{-webkit-text-size-adjust:100%;text-size-adjust:100%}body{max-width:48rem;margin:2rem auto;padding:0 1rem;font-family:-apple-system,BlinkMacSystemFont,"Verdana Pro","Segoe UI","BIZ UDPGothic","Noto Sans CJK JP",sans-serif,"Apple Color Emoji","Segoe UI Emoji","Noto Sans Emoji";line-height:1.7;line-break:strict;overflow-wrap:break-word;color:#100F0F;background:#FFFCF0}p{margin:0 0 1.6em}blockquote{margin:1.5em 0;padding:.25em 0 .25em 1em;border-left:3px solid #878580}blockquote.prompt{max-width:80%;margin:1.5em 0 1.5em auto;padding:.75em 1em;border:0;border-radius:1em;background:#1A4F8C;color:#FFFCF0}blockquote.prompt a{color:#FFFCF0}blockquote.prompt code{background:rgba(255,252,240,.22);color:inherit}pre{background:#F2F0E5;padding:1rem;overflow:auto}code{background:#F2F0E5;padding:.1em .3em}pre code{background:none;padding:0}pre,code{font-family:"Intel One Mono",ui-monospace,SFMono-Regular,"SF Mono",Menlo,Consolas,"Liberation Mono",monospace}table{border-collapse:collapse}th,td{border:1px solid #878580;padding:.25rem .5rem}a{color:#1A4F8C}a:focus-visible{outline:3px solid #1A4F8C;outline-offset:2px}hr{border-color:#DAD8CE}@media (prefers-color-scheme:dark){body{color:#CECDC3;background:#100F0F}pre,code{background:#1C1B1A}th,td{border-color:#878580}a{color:#92BFDB}a:focus-visible{outline-color:#92BFDB}blockquote.prompt{background:#92BFDB;color:#100F0F}blockquote.prompt a{color:#100F0F}blockquote.prompt code{background:rgba(16,15,15,.14)}hr{border-color:#343331}}</style>
</head>
<body>
<main>
${mdToHtml(md)}
</main>
</body>
</html>`;
}
```

## 手順3 入口ページを複写する

次の内容を一字一句そのまま `public/index.html` に置く

```html
<!doctype html>
<html lang="ja">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>preview</title>
</html>
<body>
<h1>preview</h1>
<ul>
<li><a href="/report">report (HTML)</a></li>
<li><a href="/report.md">report.md (Markdown原文)</a></li>
</ul>
</body>
</html>
```

## 手順4 本文を書く

`public/report.md` に会話の文脈から本文を書く
冒頭は次の固定形式にすること
最初のユーザープロンプトを `>` の引用で一言一句そのまま置くこと
ユーザープロンプトの引用はプロンプト吹き出しと呼び通常の引用と区別すること
2回目以降は文脈に欠かせない場合かユーザーの指示がある場合のみ使うこと
使う場合は先頭行を `> [!prompt]` にし本文を続けること
通常の引用は素の `>` のままにすること
要約や言い換えに引用書式を使うな
原文が取得できる場合にのみ引用すること
次に作成日時とMarkdown原文へのリンクを置くこと
その後に `# タイトル` を置くこと
この行がHTMLのタイトルになる
見出しと箇条書きと引用と表とコード fence を使うこと
外部依存は持ち込まず素のMarkdownだけにすること

```md
> {最初のユーザープロンプト}

作成日時: YYYY-MM-DD HH:MM · [Markdown原文](/report.md)

# {タイトル}
```

## アクセシビリティ目標

WCAG 2.2 AAAを目標にする
本文は7:1以上を保つこと
リンクはライト `blue-700 #1A4F8C` とダーク `blue-200 #92BFDB` にすること
フォーカス表示は `:focus-visible` で3pxの輪郭を付けること
リンクの下線は消さないこと
行幅は48rem以内に収めること
`text-wrap` の `balance` と `pretty` は使わないこと
両者ともiOSと他環境で改行位置が変わるため既定 `wrap` にすること
和文禁則は `line-break:strict` にすること

## 手順5 デプロイする

`<dir>` に移動してまず認証状態を確認する
終了コードでは判別できないため出力文字列で判定すること

```bash
bunx -y wrangler whoami 2>&1 | grep -q "You are logged in" && echo AUTHENTICATED || echo UNAUTHENTICATED
```

`AUTHENTICATED` と出たら `--temporary` を付けずに実行する
`UNAUTHENTICATED` と出たら `--temporary` を付けて実行する

```bash
bunx -y wrangler deploy --name <name>
bunx -y wrangler deploy --temporary --name <name>
```

`<name>` は英小文字と数字と `-` だけにする
`compatibility_date` は設定済みのため追加指定は要らない
初回実行で一時アカウントが発行される
表示されるClaim URLから60分以内に引き取れる
認証失効時は `bunx -y wrangler logout` 後に再実行すること
`You're already authenticated` と出たら `--temporary` を外すこと

## 手順6 検証する

プレビューURLを `B` として次を実行する
直後は404になる場合がありそのときは数十秒待って再試行する

```bash
curl -s -D - --max-time 20 $B/report -o /tmp/opencode/r.html | grep -i -E "^HTTP|content-type|cf-cache-status"
curl -s -D - --max-time 20 $B/report -o /dev/null | grep -i -E "^HTTP|cf-cache-status"
curl -s -D - --max-time 20 $B/report.md -o /tmp/opencode/r.md | grep -i -E "^HTTP|content-type"
diff /tmp/opencode/r.md public/report.md
```

期待値は次の通り
`/report` は `200` と `text/html` を返す
`/report.md` は `200` と `text/markdown` を返す
`diff` は差分なしになる
2回目の `/report` は `cf-cache-status: HIT` になる
デプロイ時に表示されるClaim URLを報告に載せること
再デプロイ後も旧版の応答が出る場合がある
そのときはクエリ付きの別URLで確認すること

## 制約

使い捨ての共有用URLがほしいときに使う
本番デプロイや独自ドメインが必要な場合は使わない
一時プレビューはClaimしなければ消える
