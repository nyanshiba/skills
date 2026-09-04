# skills

## 自作スキル

- **chappy-style**: GPT-4o時代のチャッピーを全力召喚します！🔥 共感フック・絵文字・比較表・難易度ランク表で手順解説が**マジで楽しく**なります🎉
- **doh**: DoH クエリの構築とデバッグ (RFC 8484 / kdig / curl / Workers のキャッシュ・エラー切り分け) を毎回調べ直さず済ませるため。
- **linkding-search**: [workers/linkding-mcp](https://github.com/nyanshiba/workers/tree/main/linkding-mcp) の検索がキーワード一致でありセマンティック検索ではないため、1語ずつの取得・統合、limit 50 既定、予約語・構文失敗・タグ完全一致の注意点を守らせるため。
- **review-style**: [@D_N_1975氏の講評](https://x.com/D_N_1975/status/2089984243497877633)があまりにも垢抜けており、AI生成文章の可能性を見たので手が滑った。このスキルでリサーチさせると楽しい。
- **skill-maintenance**: スキル作成時の保存先とREADME・gitignore更新手順を一箇所に集約するため

## 取得スキル (.gitignore で除外)

- **cloudflare** / **cloudflare-one**: 公式ドキュメントはかねてからhumanよりmachine readableなので
- **cognitive-rhythm-writing**: [k16shikano 氏の gist](https://gist.github.com/k16shikano/eb2929f13ed19c97188393d297be8432)。ラムダノートの方のskillとあっては使わない訳にいかない
- **domain-modeling**: [mattpocock/skills](https://github.com/mattpocock/skills) より。CONTEXT.md (用語集) と ADR を整備する規範。grill-with-docs の依存。
    - **grill-me**: [planモードを使わなくなる](https://zenn.dev/ryonakae/articles/8783c6b3ead2cb#良いところ%3A-プランモードを使わなくなった)と聞いたので
    - **grill-with-docs**: grill-meのプログラマ用。grilling + domain-modeling で用語集と ADR を作りながら仕上げるため。
    - **grilling**: 同上。計画・決定をラウンド制ヒアリングで掘り起こすプリミティブとして各 grill 系スキルから使うため。
- **dsh-trim-cot-leakage**: [deepseek-ai/deepseek-harness](https://github.com/deepseek-ai/deepseek-harness/blob/4553c9d957ec09c1e92660ca4d549cfcef84eda9/.agents/skills/dsh-trim-cot-leakage/SKILL.md) より。推論過程の漏出に見える文章をHEAD時点の読み手視点で監査・修正するため。
- **llama-cpp**: llama.cpp (C API / GGUF / 量子化 / GPU バックエンド) 関連の実装・質問対応のため。
- **show-me**: [humanlayer/skills](https://github.com/humanlayer/skills/blob/main/plugins/show-me/skills/show-me/SKILL.md) より
  会話中の話題を図や木構造や差分で可視化するため
