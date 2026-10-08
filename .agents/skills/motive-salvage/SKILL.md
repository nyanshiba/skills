---
name: motive-salvage
description: 別セッションからユーザの発言・動機を抜き出すときに使用する。
---

# 動機サルベージ

## 目的

別セッションの履歴から機能のきっかけ (ユーザの動機) を原文のまま抜く
要約や言い換えと引用を混ぜないこと
うろ覚えは書かず本文の裏が取れたものだけ残すこと

## 手順1 一覧を取る

`handoff_list` で `limit=50` を取り対象を絞る
表題検索 (`search`) は機能名・ファイル名で行う
`directory` が一致するものに絞る

## 手順2 文脈を読まず発言だけ抜く

`handoff_read` は `mode=messages` のみ使う (`context` は読まない)
`limit=100 order=desc` 一発で取り `type=user` だけ残す
先頭限定はしない (きっかけは中盤にある)
`cursor` と `order` の併用は 400 になるため使わない

## 手順3 fork を辿る

子の先頭 user 文は親の履歴であり動機にしない
`parentID` を辿り分岐点以降だけ見る
子では `session.created` より後の user 文だけ対象にする
先頭文が同一の別セッションは fork ではなく再投として区別する

## 手順4 定型文を除外する

次に当たるものは人間の動機にしない
`You are a subagent spawned` で始まる委譲指示文
`Use the handoff tools` で始まる誘導文
`type=synthetic` の注意書き

## 手順5 引用と要約を分ける

残った user 文は原文のまま `sessionID` と `messageID` 付きで残す
言い換えは別工程にする
出典のないものは説明で補う場合のみ短く書く

## 制約

`type` で区別する (幻覚で埋めない)
ログとツール記録も裏取りに使える
引用は短く保つ (人間も希釈するため)
