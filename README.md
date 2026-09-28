# skills

## 自作スキル (グローバル)

システムプロンプトの削減のため、 description にはスキル呼び出しに必要な文言だけを載せる。

- **chappy-style**: GPT-4o時代のチャッピーを全力召喚します！🔥 共感フック・絵文字・比較表・難易度ランク表で手順解説が**マジで楽しく**なります🎉
- **linkding-search**: [workers/linkding-mcp](https://github.com/nyanshiba/workers/tree/main/linkding-mcp) の検索がキーワード一致でありセマンティック検索ではないため、1語ずつの取得・統合、limit 50 既定、予約語・構文失敗・タグ完全一致の注意点を守らせるため。
- **review-style**: [@D_N_1975氏の講評](https://x.com/D_N_1975/status/2089984243497877633)があまりにも垢抜けており、AI生成文章の可能性を見たので手が滑った。このスキルでリサーチさせると楽しい。

## 取得スキル (グローバル, .gitignore で除外)

- **cognitive-rhythm-writing**: [k16shikano 氏の gist](https://gist.github.com/k16shikano/eb2929f13ed19c97188393d297be8432)。ラムダノートの方のskillとあっては使わない訳にいかない
- **domain-modeling**: [mattpocock/skills](https://github.com/mattpocock/skills) より。CONTEXT.md (用語集) と ADR を整備する規範。grill-with-docs の依存。
    - **grill-me**: [planモードを使わなくなる](https://zenn.dev/ryonakae/articles/8783c6b3ead2cb#良いところ%3A-プランモードを使わなくなった)と聞いたので
    - **grill-with-docs**: grill-meのプログラマ用。grilling + domain-modeling で用語集と ADR を作りながら仕上げるため。
    - **grilling**: 同上。計画・決定をラウンド制ヒアリングで掘り起こすプリミティブとして各 grill 系スキルから使うため。
- **dsh-trim-cot-leakage**: [deepseek-ai/deepseek-harness](https://github.com/deepseek-ai/deepseek-harness/blob/4553c9d957ec09c1e92660ca4d549cfcef84eda9/.agents/skills/dsh-trim-cot-leakage/SKILL.md) より。推論過程の漏出に見える文章をHEAD時点の読み手視点で監査・修正するため。
- **show-me**: [humanlayer/skills](https://github.com/humanlayer/skills/blob/main/plugins/show-me/skills/show-me/SKILL.md) より
  会話中の話題を図や木構造や差分で可視化するため

## プロジェクト固有スキル

システムプロンプトの削減のため、特定のプロジェクトでしか使わないスキルはプロジェクトローカルに置く。

### `~/cloudflare/.agents/skills/`

- **doh**: DoHクエリの構築とデバッグ手順を再利用するため
- **agents-sdk** / **cloudflare** / **cloudflare-one** / **wrangler**: [cloudflare/skills](https://github.com/cloudflare/skills/tree/main/skills)より。公式ドキュメントはかねてから human より machine readable なので
- **modern-web-guidance**: [googlechrome/modern-web-guidance](https://github.com/googlechrome/modern-web-guidance) より。 AS13335 に全て賭けているのでここに。

### `~/grafana/.agents/skills/`

- **dashboarding**: [grafana/skills/grafana-core](https://github.com/grafana/skills/tree/main/skills/grafana-core/) より。ダッシュボードJSONの作成とパネル配置と変数設定のため
- **grafana-oss**: Grafana本体とdatasourceのprovisioning設定のため
- **promql**: PromQLの作成と検証と最適化のため
- **loki**: [grafana/skills/grafana-lgtm](https://github.com/grafana/skills/tree/main/skills/grafana-lgtm) より。LogQL記述とLokiの収集設定のため
- **prometheus**: PromQL記述とalertingとrecording rulesのため
- **victoriametrics-query**: [VictoriaMetrics/skills](https://github.com/VictoriaMetrics/skills/tree/main/plugins/query/skills/victoriametrics-query) より。PromQLとMetricsQLの実行とメトリクス調査のため

### `~/llama/.agents/skills/`

- **llama-cpp**: [claude-skills/llama-cpp](https://github.com/maystudios/claude-skills/tree/main/llama-cpp) より。llama.cpp関連の実装・質問対応のため

### `~/harness/.agents/skills/`

- **skill-maintenance**: スキル作成時の保存先とREADME・gitignore更新手順を一箇所に集約するため
