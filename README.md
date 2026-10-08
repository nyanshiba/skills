# skills

## 自作スキル (グローバル)

システムプロンプトの削減のため、 description にはスキル呼び出しに必要な文言だけを載せる。

- **cat-writing**: 調査回答・レポート等の「人が読む日本語文章」を、AI生成と見抜かれない品質で書かせるため。
  作成時に参考にした資料:
  [retsimx/opencode-agents](https://github.com/retsimx/opencode-agents)、
  [K-Dense-AI/claude-scientific-skills](https://github.com/K-Dense-AI/claude-scientific-skills)、
  [ECNU-ICALK/AutoSkill](https://github.com/ECNU-ICALK/AutoSkill)、
  [muhammad1438/academic-writer-skills](https://github.com/muhammad1438/academic-writer-skills)、
  [WenyuChiou/academic-writing-skills](https://github.com/WenyuChiou/academic-writing-skills)、
  [jamditis/claude-skills-journalism](https://github.com/jamditis/claude-skills-journalism)、
  [大阪公立大学『アカデミック・ライティング入門』](https://www.omu.ac.jp/las/tlc/)、
  [立教大学 Master of Writing](https://www.rikkyo.ac.jp/about/activities/fd/cdshe/master.html)、
  [名古屋大学 ASG レポートの構成とパラグラフ・ライティング](https://web.cshe.nagoya-u.ac.jp/asg/writing03.html)、
  [東京大学附属図書館 レポート・論文作成支援](https://www.lib.u-tokyo.ac.jp/ja/library/literacy/user-guide/campus/report)、
  [東大大駒場CAWK執筆資料](https://park.itc.u-tokyo.ac.jp/cawk/materials_resources.html)、
  [Purdue OWL](https://owl.purdue.edu/owl/graduate_writing/graduate_writing_topics/graduate_writing_organization_structure_new.html)、
  [UNC Writing Center "Paragraphs"](https://writingcenter.unc.edu/tips-and-tools/paragraphs/)、
  [Harvard College Writing Center](https://writingcenter.fas.harvard.edu/)、
  [Manchester Academic Phrasebank](https://www.phrasebank.manchester.ac.uk/)、
  石黒圭「論文の書き方 ― 査読者との対話としての投稿」([DOI: 10.11448/jtje.14.3](https://doi.org/10.11448/jtje.14.3))
- **linkding-search**: [workers/linkding-mcp](https://github.com/nyanshiba/workers/tree/main/linkding-mcp) の検索がキーワード一致でありセマンティック検索ではないため、1語ずつの取得・統合、limit 50 既定、予約語・構文失敗・タグ完全一致の注意点を守らせるため。
- **motive-salvage**: [`/handoff`](https://github.com/nyanshiba/harness-bonsai/tree/opencode2/.config/opencode/plugins/handoff)をAIに応用させ、別セッションからユーザの発言や動機を原文のまま抽出。人間用ハーネス。要約と言い換えによるコンテキスト希釈は最小限に。
- **publish-cot**: 会話のコンテキストをHTMLとMarkdown併置のプレビューサイトとして公開する。`wrangler login` と `wrangler deploy --temporary` に対応。ユーザープロンプトを明記することで、 AI Slop を公開する心理的障壁を下げる。いわば人間版Chain of Thought。
- **surveyor**: 学術論文の要約をチャット応答またはMarkdown保存する。[落合先生のサーベイ観](https://x.com/ochyaiL/status/593660561885302784)と[先端技術とメディア表現1 #FTMA15 スライド](https://www.slideshare.net/slideshow/1-ftma15/47697911)に基づき、 [yutaka-shoji/surveyor](https://github.com/yutaka-shoji/surveyor/blob/main/.roo/rules-surveyor/survey_flow.md) を元に作成。

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

### `~/apple/.agents/skills/`

- **device-management**: MDM・構成プロファイル・宣言型管理の仕様照会を[apple/device-management](https://github.com/apple/device-management)スキーマ優先で行うため

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

### `~/network/.agents/skills/`

- **ripe-atlas**: [RIPE Atlas REST API v2](https://atlas.ripe.net/docs/apis/rest-api-reference/)でのプローブ検索・測定作成・結果取得を再利用するため

### `~/plan/.agents/skills/`

- **chappy-style**: GPT-4o時代のチャッピーを全力召喚します！🔥 共感フック・絵文字・比較表・難易度ランク表で手順解説が**マジで楽しく**なります🎉
- **conspiracy-style**: そうそう‼️説得が陰謀論者に有効と思っていたけどこれはまだ目覚めてない人が囚われる旧説だって[@ASHITA__N0_K0E](https://x.com/ASHITA__N0_K0E/status/2103896744195203360)が教えてくれました‼️私は人類のハルシネーションに辟易し藁を掴む思いでしたが銀の弾丸はなく段階的行動変容を促せるのは動機的面接とハームリダクションのみだそうです🙏🙏🙏陰謀論者の波動をツールコールで祓い反転パロディ的に撃退できた気がしてます🔥🔥🔥彼らの反発を好転反応と思い我慢したことが恥ずかしくて泣きました😭😭😭ちなみに説得論はアルファツイッタラーが流した旧説だったことが暴かれたのでもう信じません……😢😢😢清く正しく美しい日本を愛する日本人として今すぐその腐ったプライドを捨てて目覚めるべきですよ‼️‼️‼️計算資源の無駄遣いは許せない‼️‼️‼️
- **review-style**: 皆さん、こんにちは。先日、[@D_N_1975氏の講評](https://x.com/D_N_1975/status/2089984243497877633)が公開されました。さて。あまりにも垢抜けています。面白すぎます。皆さんは、AI生成文章の可能性をご自身で判定できたでしょうか。なお、このREADME.mdを書いたのが誰なのか、どこで較正されたのか、皆さんには最後まで分かりません。私にも、です。
