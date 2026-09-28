---
name: skill-maintenance
description: 新規スキルの作成・追加・編集をするときに使用する。
---

# スキルメンテナンス

## 目的

スキル作成作業の保存先と公開用ファイル更新を一箇所に集約する
AGENTS.mdに誘導文を残さず
descriptionによる自動想起で運用する

## 保存先

複数プロジェクトで使うスキルは次のパスに保存する
`~/.agents/skills/<id>/SKILL.md`
単一プロジェクト専用のスキルはそのプロジェクト直下に保存する
`~/<project>/.agents/skills/<id>/SKILL.md`
`<id>` はケバブケースで付ける
既存スキルと重複しないIDを選ぶ

## 作成手順

作業は次の順序で進める
1 配置先を決める (グローバルかプロジェクト固有か)
2 SKILL.md本体を配置先に作成する
3 グローバルREADME.mdに一行意図を追記する
4 取得スキルに該当する場合のみ配置先の.gitignoreを更新する
5 保存結果を読み直して検証する

## SKILL.mdの書式

先頭にnameとdescriptionのfrontmatterを置く
本文は会話経緯なしで理解できる形で書く
初めから計画したものとして全面的に書き直す
文はSemBrで改行し句点の代わりに改行で区切る
やるべき内容を肯定的に書き何をするかで指示する

## README.mdの更新

対象は `~/.agents/skills/README.md` とする
グローバル配置は自作・取得の節に追記する
プロジェクト配置は `~/<project>/.agents/skills/` の節に追記する
表記は絶対パスとし相対パスにしない
一行で作成意図が伝わる説明を書く
並びはアルファベット順に合わせる
取得スキルには出所URLを添える
出所はリポジトリやgistのURLとする

## .gitignoreの更新

対象は各配置先の `.agents/skills/.gitignore` とする
インターネットから取得したスキルのみ列挙する
自作スキルは列挙対象から外す
形式は `/<id>/` の一行形式とする
既存行の順序を崩さず追記位置を決める

## 自作と取得の判定

外部URLから取り込んだものは取得として扱う
自前で文章化したものは自作として扱う
取得はREADMEに出所URLを添えてgitignoreにも加える
自作はREADMEのみに加えてgitignoreには加えない

## READMEのskill認識抑止

ソース直下のMarkdownはskillとして検出される
README.mdも例外ではなくID `README` で登録される
descriptionがなくても一覧には載るため抑止が必要になる
抑止は `opencode.json` の権限で一括して行う
`{ "action": "skill", "resource": "README", "effect": "deny" }`
この一行で全ソースのREADMEが検出対象から外れる
新規プロジェクトにskills置き場を作っても追加設定は要らない

## 検証

SKILL.mdが所定パスに存在することを確認する
frontmatterのnameとdescriptionを確認する
READMEの追記位置と出所URL有無を確認する
gitignoreの要否判定と形式を確認する
最終回答もSemBrで書き会話前提の説明を残さない
