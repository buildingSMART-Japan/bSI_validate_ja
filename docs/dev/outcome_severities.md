# 結果の重大性

注：このドキュメントはバージョン0.6.8のコードベースに基づいて2024年11月に作成されました。

## 概要 - データベースモデルの列挙

厳しさとは、以下のような厳しさの値を列挙したものである。
[Validation Outcome](https://github.com/buildingSMART/ifc-validation-data-model/blob/main/models.py#L978).

| 文字列の値 | 整数列挙値 |
|----------------|---------------------------|
| 実行済み | 1 |
| 可決 | 2 |
| 警告 | 3 |
| エラー | 4 |
| NOT_APPLICABLE | 0 |

## Severity Usage

### スキーマチェック

スキーマ・チェックは常に適用されるため、可能な列挙値は以下のとおりである。`PASSED`そして`ERROR`.
[Passing outcomes for schema checks are stored to the database](https://github.com/buildingSMART/validate/blob/8f04bcc6d1f400240485a33b2c81e2f7d0edbeab/backend/apps/ifc_validation/tasks.py#L607).

### 構文チェック

構文チェックも常に適用されるため、可能な列挙値は以下の通りである。`PASSED`そして`ERROR`.
[Passing outcomes for syntax checks are stored to the database](https://github.com/buildingSMART/validate/blob/8f04bcc6d1f400240485a33b2c81e2f7d0edbeab/backend/apps/ifc_validation/tasks.py#L607).

### ガーキンルール（規範と業界のベストプラクティス）

これらのルールは一般的に同様の方法で処理される。
Unlike syntax and schema checks, a gherkin rule may or may not be applicable to a given model depending on the schema
and content of the model.
For example, alignment rules are only applicable to models with schema `IFC4X3_ADD2` as those entities were not part of
the
`IFC2X3` or `IFC4` schema versions.

したがって、どちらの検証チェックも、深刻度が以下の結果を返す可能性がある。`NOT_APPLICABLE`.

#### 規範的ルール

規範的ルールは、実施者の合意や非公式な命題から要件を強制する。
Therefore, the potential outcomes are:

- `NOT_APPLICABLE`
- `ERROR`
- `PASSED`

#### 業界のベストプラクティス

業界のベストプラクティスのチェックでは、要求されていない項目が強制される、
but rather represent the preferred or most idiomatic way of implementing the IFC standard.

したがって、考えられる結果は以下の通りだ：

- `NOT_APPLICABLE`
- `WARNING`
- `PASSED`

#### ガーキンルール処理

現在のところ、severity= で保存されたガーキンルールの個々のインスタンス結果はありません。`PASSED`.
The initial idea was to pass them to the DB but it was quickly flooded with outcomes of this severity
and is currently [commented out](https://github.com/buildingSMART/ifc-gherkin-rules/blob/b363041433f252fc1b9e043ee3aac0bd6fcfad3d/features/steps/validation_handling.py#L254-L268).

_(混乱を避けるため、次の段落は削除する可能性がある...）。_

ガーキンルール（*.featureファイル）を`behave`深刻度`PASSED`にのみ使用される。`Given`という声明を発表した。
A `PASSED` outcome is added to the temporary outcomes when the conditions of a `Given` statement are met.

全体の処理ループは以下の通り：

1. 厳しさ`NOT_APPLICABLE`はすべてのエンティティ・インスタンスに適用される
2. について`Given`ステートメントが実行される
3. もし*少なくとも1つ*の要件を満たすエンティティ・インスタンス。*すべて* `Given`という声明を発表した、
   the rule is considered to be "activated" and severity=`EXECUTED` is applied to these instance(s).
   This is an 'all or nothing' situation where all `Given` statements must be "activated" in order for the rule
   to be considered "activated".
4. これらのインスタンスは、次のようにテストされる。`Then`という声明を発表した。
5. 厳しさ`ERROR`の要件を満たさない各インスタンスに適用される。`Then`という声明を発表した。

を持つインスタンスが少なくとも1つあれば`ERROR`その結果、ルールの集約ステータスが以下のように返される：

- 厳しさ`ERROR`規範的ルール（実施者合意と非公式命題）に対して
- 厳しさ`WARNING`業界のベストプラクティスのために。

このステータスは、検証レポートの各ルールの「ブロック」に色を付けるために使用される。

## 成果の表示と報告

### 個別規則（規範および業界のベストプラクティス）

結果は常に、すべてのインスタンスの集約されたステータスに基づいてウェブUIで報告される。
activated by a given rule.

| ルールの「重要度」の集計値 | 表示色 | 重症度欄のラベル |
|-------------------------------------------|----------------|-------------------------------------|
| `NOT_APPLICABLE` | 灰色 | 該当なし |
| `実行済み` | グリーン | 該当する |
| `可決` | (使用せず） |  |
| `警告` | イエロー | 警告 |
| `エラー` | 赤 | エラー |

Display colour is determined by the
[statusToColor](https://github.com/buildingSMART/validate/blob/development/frontend/src/mappings.js#L1)
mapping function.

表示ラベルは 
[statusToLabel](https://github.com/buildingSMART/validate/blob/development/frontend/src/mappings.js#L10)
mapping function.

### 全体的な状況

各タイプのチェック（構文、スキーマ、規範的ルール、ベストプラクティス）に対する単一の全体的なステータス
is displayed on the Validation Service dashboard for each model.

それぞれの全体的な状況
[ValidationTask](https://github.com/buildingSMART/ifc-validation-data-model/blob/main/models.py#L778)
is captured in a different data structure of
[Model.Status](https://github.com/buildingSMART/ifc-validation-data-model/blob/main/models.py#L324)
with the following possible enumeration values:

| Model.Statusの値 | 表示色 | シンボル |
|-------------------------|----------------|------------------------------------------------------------------------------|
| `有効` | グリーン | チェックサークルアイコン |
| `無効` | 赤 | 警告アイコン |
| `NOT_VALIDATED` | 灰色 | 砂時計の底アイコン |
| `警告` | イエロー | エラーアイコン |
| `NOT_APPLICABLE` | 灰色 | BrowserNotSupportedIcon（技術的には可能だが、実際には発生しない） |

The options for overall status of each category of ValidationTask are as follows:

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
[taking the highest severity](https://github.com/buildingSMART/ifc-validation-data-model/blob/f32164ab762fc695690d380e12e87c815b641912/models.py#L948)
of outcomes for all the rules contained in that task.

