---
name: skill-maintenance
description: 新規スキルの作成・追加・編集で使用する／SKILL.md保存先の決定とREADME・gitignore更新が必要な場合に読み込む
---

# スキルメンテナンス

## 目的

スキル作成作業の保存先と公開用ファイル更新を一箇所に集約する
AGENTS.mdに誘導文を残さず
descriptionによる自動想起で運用する

## 保存先

スキル本体は常に次のパスに保存する
`~/.agents/skills/<id>/SKILL.md`
`<id>` はケバブケースで付ける
既存スキルと重複しないIDを選ぶ

## 作成手順

作業は次の順序で進める
1 SKILL.md本体を作成する
2 README.mdに一行意図を追記する
3 取得スキルに該当する場合のみ .gitignoreを更新する
4 保存結果を読み直して検証する

## SKILL.mdの書式

先頭にnameとdescriptionのfrontmatterを置く
本文は会話経緯なしで理解できる形で書く
初めから計画したものとして全面的に書き直す
文はSemBrで改行し句点の代わりに改行で区切る
やるべき内容を肯定的に書き何をするかで指示する

## README.mdの更新

対象は `~/.agents/skills/README.md` とする
自作スキルは `## 自作スキル` に追記する
取得スキルは `## 取得スキル (.gitignore で除外)` に追記する
一行で作成意図が伝わる説明を書く
並びはアルファベット順に合わせる
取得スキルには出所URLを添える
出所はリポジトリやgistのURLとする

## .gitignoreの更新

対象は `~/.agents/skills/.gitignore` とする
インターネットから取得したスキルのみ列挙する
自作スキルは列挙対象から外す
形式は `/<id>/` の一行形式とする
既存行の順序を崩さず追記位置を決める

## 自作と取得の判定

外部URLから取り込んだものは取得として扱う
自前で文章化したものは自作として扱う
取得はREADMEに出所URLを添えてgitignoreにも加える
自作はREADMEのみに加えてgitignoreには加えない

## 検証

SKILL.mdが所定パスに存在することを確認する
frontmatterのnameとdescriptionを確認する
READMEの追記位置と出所URL有無を確認する
gitignoreの要否判定と形式を確認する
最終回答もSemBrで書き会話前提の説明を残さない
