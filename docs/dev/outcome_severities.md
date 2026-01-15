# 結果の重大性
注：このドキュメントはバージョン0.6.8のコードベースに基づいて2024年11月に作成されました。

## 概要 - データベースモデルの列挙
Severitiesは、[Validation Outcomeの](https://github.com/buildingSMART/ifc-validation-data-model/blob/main/models.py#L978)深刻度を表す値の列挙である。


| 文字列の値 | 整数列挙値 |
|----------------|---------------------------|
| 実行済み | 1 |
| 可決 | 2 |
| 警告 | 3 |
| エラー | 4 |
| NOT_APPLICABLE | 0 |

## 重要度の使用
### スキーマチェック
スキーマ・チェックは常に適用されるため、可能な列挙値は`PASSED`と`ERROR` のみである。  
[スキーマ・チェックの合格結果はデータベースに保存される](https://github.com/buildingSMART/validate/blob/8f04bcc6d1f400240485a33b2c81e2f7d0edbeab/backend/apps/ifc_validation/tasks.py#L607)。

### 構文チェック
構文チェックも常に適用されるため、可能な列挙値は`PASSED`と`ERROR` のみである。  
[構文チェックの合格結果はデータベースに保存される](https://github.com/buildingSMART/validate/blob/8f04bcc6d1f400240485a33b2c81e2f7d0edbeab/backend/apps/ifc_validation/tasks.py#L607)。

### ガーキンルール（規範と業界のベストプラクティス）
これらのルールは一般的に同様の方法で処理されます。構文やスキーマのチェックとは異なり、ガーキンルールはモデルのスキーマや内容によって、あるモデルに適用できる場合とできない場合があります。例えば、整列ルールはスキーマが `IFC4X3_ADD2`モデルにのみ適用されます。 `IFC2X3`または `IFC4`スキーマのバージョンには含まれていないためです。






したがって、どちらのバリデーション・チェックも、深刻度が `NOT_APPLICABLE`返す可能性がある。

#### 規範的ルール
規範的ルールは、実施者の合意や非公式な命題から要件を強制する。従って、潜在的な成果は以下の通りである：


- `NOT_APPLICABLE`
- `ERROR`
- `PASSED`

#### 業界のベストプラクティス
インダストリー・ベスト・プラクティスのチェックは、必須項目ではなく、IFC標準を実装する上で好ましい、あるいは最も慣用的な方法を示す項目を実施するものです。


したがって、考えられる結果は以下の通りだ：

- `NOT_APPLICABLE`
- `WARNING`
- `PASSED`

#### ガーキンルール処理
現在、severity=`PASSED`保存されたgherkinルールの個々のインスタンス結果はありません。最初のアイデアはそれらをDBに渡すことでしたが、すぐにこの重大度の結果であふれましたので、現在は[コメントアウトされて](https://github.com/buildingSMART/ifc-gherkin-rules/blob/b363041433f252fc1b9e043ee3aac0bd6fcfad3d/features/steps/validation_handling.py#L254-L268)います。



*(混乱を避けるため、次の段落は削除する可能性がある...)*。

`behave` gherkin ルール (*.feature ファイル) を処理する場合、`Given`ステートメントに対してのみ Severity=`PASSED`が使用されます。  
`Given`ステートメントの条件が満たされると、`PASSED`結果が一時的な結果に追加される。

全体の処理ループは以下の通り：

1. Severity=`NOT_APPLICABLE`はすべてのエンティティインスタンスに適用されます。

1. `Given`ステートメントが実行される。

1. *ALL* `Given` ステートメントの要件を満たすエンティティ・インスタンスが少なくとも1つある場合、ルールは"アクティブ化さ"れたと見なされ、severity=`EXECUTED`がこれらのインスタンスに適用される。これは"all or nothing"状況であり、ルールが"有効化さ"れたと見なされるためには、`Given`ステートメントがすべて"有効化"されていなければならない。




1. それらのインスタンスは、`Then` のステートメントに対してテストされる。

1. Severity=`ERROR` は、`Then` ステートメントの要件に失敗した各インスタンスに適用される。

`ERROR`結果を持つインスタンスが少なくとも1つある場合、ルールの集約ステータスが以下のように返される：

- 厳しさ＝`ERROR` 規範的ルール（実施者合意と非公式提案）の場合
- 深刻度=`WARNING` 業界のベストプラクティス。

このステータスは、検証レポートの各ルールの"ブロックに"色を付けるために使用される。

## 成果の表示と報告
### 個別規則（規範および業界のベストプラクティス）
結果は常に、指定されたルールによってアクティブ化されたすべてのインスタンスの集約されたステータスに基づいて、ウェブUIで報告される。


| ルールの`Severity` の合計値 | 表示色 | `Severity`列に報告されたラベル |
|-------------------------------------------|----------------|-------------------------------------|
| `NOT_APPLICABLE` | 灰色 | 該当なし |
| `実行済み` | グリーン | 該当する |
| `可決` | (使用せず) |  |
| `警告` | イエロー | 警告 |
| `エラー` | 赤 | エラー |

表示色は[statusToColor](https://github.com/buildingSMART/validate/blob/development/frontend/src/mappings.js#L1)マッピング関数によって決定される。



表示ラベルは、[statusToLabel](https://github.com/buildingSMART/validate/blob/development/frontend/src/mappings.js#L10)マッピング関数によって決定される。



### 全体的な状況
各タイプのチェック（構文、スキーマ、規範ルール、ベストプラクティス）に対する単一の全体的なステータスは、各モデルのバリデーション・サービス・ダッシュボードに表示されます。


[各ValidationTaskの](https://github.com/buildingSMART/ifc-validation-data-model/blob/main/models.py#L778)全体的なステータスは、[Model.Statusの](https://github.com/buildingSMART/ifc-validation-data-model/blob/main/models.py#L324)別のデータ構造で取得され、以下の列挙値が考えられます：





| 価値`Model.Status` | 表示色 | シンボル |
|-------------------------|----------------|------------------------------------------------------------------------------|
| `有効` | グリーン | チェックサークルアイコン |
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

規範ルール ValidationTask の全体的なステータスは、そのタスクに含まれるすべてのルールの結果の中で[最も高い重大度を取る](https://github.com/buildingSMART/ifc-validation-data-model/blob/f32164ab762fc695690d380e12e87c815b641912/models.py#L948)ことによって決定される。



