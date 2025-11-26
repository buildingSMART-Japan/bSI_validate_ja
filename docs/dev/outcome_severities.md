# 結果の重大性
注：このドキュメントはバージョン0.6.8のコードベースに基づいて2024年11月に作成されました。

## 概要 - データベースモデルの列挙
厳しさとは、以下のような厳しさの値を列挙したものである。  
[検証結果](https://github.com/buildingSMART/ifc-validation-data-model/blob/main/models.py#L978)。

| <nobr>文字列の</nobr>値 | 整数<nobr>列挙</nobr>値       |
|----------------|---------------------------|
| 実行済み　　　 | 1　　　　　　 |
| 可決 | 2 |
| 警告 | 3 |
| エラー | 4 |
| NOT_APPLICABLE | 0 |

## 重要度の使用
### スキーマチェック
スキーマ・チェックは常に適用されるため、可能な列挙値は`PASSED` と`ERROR` のみである。  
[スキーマ・チェックの合格結果はデータベースに保存される](https://github.com/buildingSMART/validate/blob/8f04bcc6d1f400240485a33b2c81e2f7d0edbeab/backend/apps/ifc_validation/tasks.py#L607)。

### 構文チェック
構文チェックも常に適用されるため、可能な列挙値は`PASSED` と`ERROR` のみである。  
[構文チェックの合格結果はデータベースに保存される](https://github.com/buildingSMART/validate/blob/8f04bcc6d1f400240485a33b2c81e2f7d0edbeab/backend/apps/ifc_validation/tasks.py#L607)。

### ガーキンルール（規範と業界のベストプラクティス）
これらのルールは一般的に同様の方法で処理される。  
構文やスキーマのチェックとは異なり、ガーキンルールはスキーマによって、与えられたモデルに適用できるかどうかが決まります。  
そしてモデルの内容。  
例えば、整列ルールはスキーマを持つモデルにのみ適用される。 `IFC4X3_ADD2`の一部のエンティティではないためです。  
その  
`IFC2X3`または `IFC4`スキーマのバージョン

したがって、どちらの検証チェックも、重大度. `NOT_APPLICABLE`.

#### 規範的ルール
規範的ルールは、実施者の合意や非公式な命題から要件を強制する。  
したがって、考えられる結果は以下の通りだ：

- `NOT_APPLICABLE`
- `ERROR`
- `PASSED`

#### 業界のベストプラクティス
業界のベストプラクティスのチェックでは、要求されていない項目が強制される、  
むしろ、IFC 標準を実装する上で好ましい、あるいは最も慣用的な方法を表している。

したがって、考えられる結果は以下の通りだ：

- `NOT_APPLICABLE`
- `WARNING`
- `PASSED`

#### ガーキンルール処理
現在のところ、severity=`PASSED` で保存されたガーキンルールによる個別のインスタンス結果はありません。  
最初はDBに渡すつもりだったが、すぐにこの厳しさの結果で溢れかえった。  
現在は[コメントアウトされて](https://github.com/buildingSMART/ifc-gherkin-rules/blob/b363041433f252fc1b9e043ee3aac0bd6fcfad3d/features/steps/validation_handling.py#L254-L268)いる。

*(混乱を避けるため、次の段落は削除する可能性がある...)*。

gherkin ルール（*.feature ファイル）を`behave` で処理する場合、Severity=`PASSED` は`Given` ステートメントに対してのみ使用されます。  
`Given` ステートメントの条件が満たされると、`PASSED` の結果が一時的な結果に追加される。

全体の処理ループは以下の通り：

1. Severity=`NOT_APPLICABLE`はすべてのエンティティインスタンスに適用されます。

1. `Given` 。

1. *ALL* `Given` ステートメントの要件を満たすエンティティ・インスタンスが*少なくとも1つ*ある場合、  
   ルールが"有効化さ"れたとみなされ、これらのインスタンスに severity=`EXECUTED` が適用されます。  
   これは all or nothing の状況であり、ルールが"有効に"なるためには、すべての`Given` 。  
   が"活性化したと"みなされる。
1. それらのインスタンスは、`Then` のステートメントに対してテストされる。

1. Severity=`ERROR` は、`Then` ステートメントの要件に失敗した各インスタンスに適用される。

`ERROR` の結果を持つインスタンスが少なくとも1つある場合、ルールの集約ステータスが以下のように返される：

- 厳しさ＝`ERROR` 規範的ルール（実施者合意と非公式提案）の場合
- 深刻度=`WARNING` 業界のベストプラクティス。

このステータスは、検証レポートの各ルールの"ブロックに"色を付けるために使用される。

## 成果の表示と報告
### 個別規則（規範および業界のベストプラクティス）
結果は常に、すべてのインスタンスの集約されたステータスに基づいてウェブUIで報告される。  
与えられたルールによって活性化される。

| ルールの`Severity` の<nobr>合計</nobr>値 | <nobr>表示</nobr>色 | `Severity` <nobr>。</nobr> |
|-------------------------------------------|----------------|-------------------------------------|
| `NOT_APPLICABLE` | 灰色　　　　　 | 該当なし　　　 |
| `実行済み` | グリーン | 該当する |
| `可決` | (使用せず) |  |
| `警告` | イエロー | 警告 |
| `エラー` | レッド | エラー |

ディスプレイの色は  
[ステータストゥカラー](https://github.com/buildingSMART/validate/blob/development/frontend/src/mappings.js#L1)  
マッピング機能。

表示ラベルは   
[statusToLabel](https://github.com/buildingSMART/validate/blob/development/frontend/src/mappings.js#L10)  
マッピング機能。

### 全体的な状況
各タイプのチェック（構文、スキーマ、規範的ルール、ベストプラクティス）に対する単一の全体的なステータス  
は各モデルのバリデーションサービスダッシュボードに表示されます。

それぞれの全体的な状況  
[バリデーションタスク](https://github.com/buildingSMART/ifc-validation-data-model/blob/main/models.py#L778)  
の別のデータ構造に取り込まれる。  
[モデル.ステータス](https://github.com/buildingSMART/ifc-validation-data-model/blob/main/models.py#L324)  
を以下の列挙値で指定する：

| 価値 `Model.<nobr>Status</nobr>` | <nobr>表示</nobr>色 | <nobr>シンボル</nobr> |
|-------------------------|----------------|------------------------------------------------------------------------------|
| `有効`　　　　　 | グリーン　　　 | チェックサークルアイコン |
| `無効` | 赤 | 警告アイコン |
| `NOT_VALIDATED` | 灰色 | 砂時計の底アイコン |
| `警告` | イエロー | エラーアイコン |
| `NOT_APPLICABLE` | 灰色 | BrowserNotSupportedIcon（技術的には可能だが、実際には発生しない） |

ValidationTaskの各カテゴリの全体的なステータスのオプションは以下の通り：

- 構文とスキーマ
  - `VALID`
  - `INVALID`

- 規範的ルール
  - `VALID`
  - `INVALID`
  - `NOT_APPLICABLE`

- ベストプラクティス
  - `VALID`
  - `WARNING`
  - `NOT_APPLICABLE`

規範規則 ValidationTask の全体的な状態は、以下によって決定される。  
[最高度の厳しさ](https://github.com/buildingSMART/ifc-validation-data-model/blob/f32164ab762fc695690d380e12e87c815b641912/models.py#L948)  
そのタスクに含まれるすべてのルールの結果の。

