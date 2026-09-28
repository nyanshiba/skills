---
name: device-management
description: "Apple MDM・構成プロファイル・宣言型管理の仕様照会に使用する。ペイロード可否・必須キー・OS導入版・監視条件をapple/device-managementスキーマで裏取りする"
---

# MDMリファレンス

## 目的

Apple MDMと構成プロファイルと宣言型管理の仕様照会を正確に行う
一次情報のYAMLスキーマを優先して回答の根拠を示す
開発者文書と運用文書は解説の補助として扱う

## 使用条件

MDMコマンドやチェックインやエラーについて問われたときに読み込む
構成プロファイルやmobileconfigやペイロードについて問われたときに読み込む
宣言型管理のconfigurationやstatusやactivationやassetについて問われたときに読み込む
iOSやmacOSやtvOSやvisionOSやwatchOSの導入版や非推奨や削除の確認に読み込む
監視端末やチャネルや登録種別の可否確認に読み込む

## 参照元

公式スキーマはapple/device-managementのreleaseブランチを正とする
取得は `git clone --depth 1 --branch release https://github.com/apple/device-management.git` で行う
常用の配置先は `~/apple/device-management/` とする
開発者文書は https://developer.apple.com/documentation/devicemanagement で補う
運用解説はApple Platform Deploymentで補う
スキーマと文書が食い違う場合はスキーマを優先して食い違いを明記する
改善要望の送付先はFeedback AssistantのEnterprise and Education領域と案内する

## 対応表

種別ごとに次のディレクトリを起点に探す
MDMコマンドは `mdm/commands/` で `requesttype` を探す
MDMチェックインは `mdm/checkin/` で `requesttype` を探す
MDMエラーは `mdm/errors/` で内容を探す
構成プロファイルは `mdm/profiles/` で `payloadtype` を探す
トップレベル構造は `mdm/profiles/TopLevel.yaml` と `mdm/profiles/CommonPayloadKeys.yaml` を先に読む
宣言の共通構造は `declarative/declarations/declarationbase.yaml` を先に読む
configurationは `declarative/declarations/configurations/` で `declarationtype` を探す
activationは `declarative/declarations/activations/` で探す
assetは `declarative/declarations/assets/` で探す
managementは `declarative/declarations/management/` で探す
statusは `declarative/status/` で `statusitemtype` を探す
protocolは `declarative/protocol/` で `requesttype` を探す
その他形式は `other/` の `esso.yaml` と `machineinfo.yaml` と `manifesturl.yaml` と `passwordhash.yaml` と `skipkeys.yaml` を探す
実例は `examples/` 配下の同名ディレクトリで探す
書式の解釈は `docs/schema.md` に従う
版差分は `CHANGES.md` で確認する

## 手順

問われた用語から種別を判定して対応表のディレクトリに絞る
`grep -rl` で型名を引き当てて候補YAMLを1件に絞る
候補YAMLの `payload` と `payloadkeys` と `responsekeys` を読む
該当キー固有の `supportedOS` を読む
`examples/` の同名ディレクトリで実例を1件読んで裏取りする
必要に応じて `CHANGES.md` で新規や非推奨や削除を確認する
リポジトリ全体を先読みせず該当ファイルだけを読む

## supportedOSの読み方

キー固有の値は `payload` 全体の値を継承して上書きする
`introduced` と `deprecated` と `removed` と `n/a` を版として読む
`supervised` と `requiresdep` と `userapprovedmdm` を前提条件として読む
`devicechannel` と `userchannel` を配信経路として読む
`allowmanualinstall` を手動導入可否として読む
`userenrollment` の `mode` を `allowed` と `required` と `forbidden` と `ignored` で読む
`sharedipad` の `mode` を同様に読む
宣言型では `allowed-enrollments` と `allowed-scopes` を登録種別と適用範囲として読む
MDMコマンドでは `accessrights` を権限ビットの前提として読む
`presence` の `required` と `optional` を必須可否として読む
`rangelist` と `range` と `default` を値域として読む
`subkeys` を辞書と配列の入れ子構造として読む

## 回答形式

型名とファイルパスを先に示す
可否の結論を次に示す
必須キーと値域と既定値を列挙する
OS別の導入版と非推奨版と削除版を添える
監視条件とチャネル条件と登録種別条件を添える
根拠のYAMLパスと実例パスを示す
文書で補った場合はスキーマ優先の旨を添える
