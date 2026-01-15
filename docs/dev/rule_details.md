# 規範規則の詳細情報
バリデーション・サービスに新しいルールを追加するには、以下の手順に従ってください。

| n. | ステップ | 責任 |
|----|------------------------------------------------------------------------------------------|-----------------------------|
| 1 | bSIifc-gherkin-rules リポジトリに新しいブランチを作成する。 | bSIバリデーションサービスチーム |
| 2 | このブランチでは、**以下の**手順に従って、必要なルールの開発を開始する。 | ルール開発者 |
| 3 | プルリクエストを作成し、サンドボックス環境を使用してルールの動作をさらにテストする。 | ルール開発者 |
| 4 | プルリクエストにレビュアーを割り当てます。 | ルール開発者 |
| 5 | プルリクエストを確認する | bSIバリデーションサービスチーム |
| 6 | (オプション) レビューアからのフィードバックに従ってルールを修正する。 | ルール開発者 |
| 7 | プルリクエストを承認してマージする | bSIバリデーションサービスチーム |

## 1.ブランチの創設
[すべてのルールを含む](https://github.com/buildingSMART/ifc-gherkin-rules)buildingSMART[GitHubリポジトリに](https://github.com/buildingSMART/ifc-gherkin-rules)、新しいルールを開発するために使用するブランチを作成します。

- ブランチに新しいルールの名前を付けます。例:`GEM900` ジオメトリ機能部の新規ルールの場合
- ブランチごとに1つのルールを追加し、レビューを容易にする（1ルール = 1`.feature` ファイル）

## 2.ルール開発
ルールが完成したとみなされる：

- [**ガーキンフィーチャーファイル**](21-write-feature-files-gherkin-rules-for-ifc)
- 対応するpythonの実装（別名、[**pythonのステップ）**](22-write-python-steps)
- [**単体テストファイル**](23-write-unit-test-files)一式

以下は、これら3つのコンポーネントすべてについての説明である。

(21-write-feature-files-gherkin-rules-for-ifc)=。
### 2.1)IFC用のフィーチャーファイル（ガーキンルール）を書く
フィーチャーファイルとは、ルールの動作を記述した、ガーキン構文で記述されたファイルのことである。  
作成したブランチに、以下の手順に従ってGherkin featureファイルを追加します。

**ファイル形式**：`.feature`

**場所**：https://github.com/buildingSMART/ifc-gherkin-rules/tree/main/features

#### 機能ファイルの命名規則
- ファイル名はrulecode_ruletitle。
- ルールコードは、3桁の大文字（[一部の機能部品](./functional_parts.md)リストより抜粋）＋3桁の数字で構成される。
- ルールコードとルールタイトルは一意でなければならない。
- ルールのタイトルにはスペースを入れず、区切り文字として`-` を使用する。

<details><summary>不正解</summary>

```
SPS001 - Basic-spatial-structure-for-buildings.feature
SPS001_Basic spatial structure for buildings.feature
SPS001 - Basic spatial structure for buildings.feature
```
</details>
<details><summary>右</summary>

```
SPS001_Basic-spatial-structure-for-buildings.feature
```
</details>

#### 必須コンテンツ
`.feature`ファイル：
- それらは、検証カテゴリーを分類するために、これらのタグを1つだけ含まなければならない：
    - `@critical`
    - `@implementer-agreement`
    - `@informal-proposition`
    - `@industry-practice` (警告；合否ではない）
- それらは、一部の機能部品に3文字のアルファタグを付けなければならない。[機能部品を](./functional_parts.md)参照。
- フィーチャーファイルのバージョンを 1 ベースの整数で示すタグが 1 つ含まれていなければなりません。
  - 例：`@version1` フィーチャーファイルの初期バージョン用
  - 例：`@version3` フィーチャーファイルの第3バージョン用
    - 誤字脱字の修正や説明文の書き直しなどの軽微な変更では、バージョンは上がりません。
    - **"Given"** 文や**"Then"** 文、あるいはステップの実装を変更する場合は、バージョン番号を1インクリメントする必要がある。
- [エラーコードを](error-codes)示す1つ以上のタグが含まれていなければならない。
  - すべてのシナリオで同じエラーが発生する場合は、**"Feature:"**行のすぐ上に次のタグを記述する。

    <details><summary>例</summary>

    ```
    @implementer-agreement
    @GRF
    @version1
    @E00050
    Feature: GRF001 - Identical....
    ```

    </details>

- いくつかのシナリオで異なるエラーコードが発生する場合は、このタグを各"**"** ライン


    <details><summary>例</summary>

    ```
    @implementer-agreement
    @ALS
    @version1
    Feature: ALS005 - Alignment shape representation

    Background: ...

    @E00020
    Scenario: Agreement on ... representation - Value

    @E00010
    Scenario: Agreement on ... representation - Type
    ```
 
    </details>
  
- 正確に1つのフィーチャーを含まなければならない
- ルールコード - ルールタイトル（ファイル名と同じ）。ルールタイトルには、`-` の代わりに空白を使用する必要があります。

<details><summary>不正解</summary>

```
Feature: ALB001_Alignment Layout

Given ...
Then ...
```
```
@ALB
Feature: ALB001_Alignment-Layout

Given ...
Then ...
```
```
@ALB
Feature: ALB001 - Alignment-Layout

Given ...
Then ...
```

</details>
<details><summary>右</summary>

```
@ALB
Feature: ALB001 - Alignment Layout

Given ...
Then ...
```
</details>

 - ルールには「このルールは以下を確認します...」で始まる説明を含める必要があります

<details><summary>例</summary>

```
@implementer-agreement
@ALB
Feature: ALB003 - Allowed entities nested in Alignment
The rule verifies that an Alignment has a nesting relationship with its components (i.e., Horizontal, Vertical, Cant layouts) or with Referents (e.g., mileage markers). And not with any other entity.

  Scenario: Agreement on nested elements of IfcAlignment
  Given ...
  Then ...
```
</details>

#### 必須
フィーチャファイルのルールが特定のIFCバージョンおよび/またはビュー定義にのみ適用される場合、フィーチャファイル（またはシナリオが複数ある場合はその各シナリオ）は、以下のステップの適用可能性を指定する Given ステップで開始する必要があります。

<details><summary>例</summary>

```
Given A model with Schema "IFC2X3"
Given A file with Model View Definition "CoordinationView"
```
```
Given A model with Schema "IFC2X3" or "IFC4"
Given A file with Model View Definition "CoordinationView" or "ReferenceView"
```
</details>

#### オプション
`.feature`ファイル：
- 1つ以上のシナリオを含むことができる
- シナリオタイトルに制約はない
- `@disabled` 、一時的に処理から外すことができる。

#### ステップとステップの間にスペースがない
<details><summary>不正解</summary>

```
Given A model with Schema "IFC4.3"

Then Each IfcAlignmentHorizontal must be nested only by 1 IfcAlignment
Then Each IfcAlignmentVertical must be nested only by 1 IfcAlignment
Then Each IfcAlignmentCant must be nested only by 1 IfcAlignment
```
</details>
<details><summary>右</summary>

```
Given A model with Schema "IFC4.3"
Then Each IfcAlignmentHorizontal must be nested only by 1 IfcAlignment
Then Each IfcAlignmentVertical must be nested only by 1 IfcAlignment
Then Each IfcAlignmentCant must be nested only by 1 IfcAlignment
```
</details>

#### 余分な空白に注意
<details><summary>不正解</summary>

```
Given A model with Schema "IFC4.3"
Then Each IfcAlignmentHorizontal must be nested only by 1 IfcAlignment
Then  Each IfcAlignmentVertical must be nested only by 1 IfcAlignment
Then  Each IfcAlignmentCant must be nested only by 1 IfcAlignment
```
</details>
<details><summary>右</summary>

```
Given A model with Schema "IFC4.3"
Then Each IfcAlignmentHorizontal must be nested only by 1 IfcAlignment
Then Each IfcAlignmentVertical must be nested only by 1 IfcAlignment
Then Each IfcAlignmentCant must be nested only by 1 IfcAlignment
```
</details>

#### ステップの最後に句読点を使用しないでください
<details><summary>不正解</summary>

```
Given A model with Schema "IFC4.3",
Then Each IfcAlignmentHorizontal must be nested only by 1 IfcAlignment;
Then Each IfcAlignmentVertical must be nested only by 1 IfcAlignment;
Then Each IfcAlignmentCant must be nested only by 1 IfcAlignment.
```
</details>
<details><summary>右</summary>

```
Given A model with Schema "IFC4.3"
Then Each IfcAlignmentHorizontal must be nested only by 1 IfcAlignment
Then Each IfcAlignmentVertical must be nested only by 1 IfcAlignment
Then Each IfcAlignmentCant must be nested only by 1 IfcAlignment
```
</details>

#### パラメータを入力するときは注意してください。大文字と小文字は区別されます！
<details><summary>不正解</summary>

```
Given A model with schema "IFC4.3",
```
</details>
<details><summary>右</summary>

```
Given A model with Schema "IFC4.3"
```
</details>

#### Must vs Shall
要件を課すには**shall ではなく** **must** を使うこと。[ALB001_Alignment-in-spatial-structure.feature](https://github.com/buildingSMART/ifc-gherkin-rules/blob/main/features/ALB002_Alignment-layout.feature) "Shall"は"曖昧である。また、法律分野では、要求や義務を表現する最も明確な方法として、"must"を強く好む傾向にある。



<details><summary>不正解</summary>

```
Given A model with Schema "IFC2X3"
Given A file with Model View Definition "CoordinationView"
Then There shall be exactly 1 IfcSite element(s)
```
</details>
<details><summary>右</summary>

```
Given A model with Schema "IFC2X3"
Given A file with Model View Definition "CoordinationView"
Then There must be exactly 1 IfcSite element(s)
```
</details>

#### IFC関係の動詞
ルールが特定のIFCリレーションシップの存在を必要とする場合、以下の表を参照し、適切な動詞を使用する。

| IFC relationship       | Verb for rules        | Examples                                                           |
|------------------------|-----------------------|--------------------------------------------------------------------|
| IfcRelAggregates       | aggregate, aggregates | Then IfcSite must aggregate IfcBuilding                            |
| IfcRelNests            | nest, nests           | Then Each IfcAlignmentVertical nests a list of IfcAlignmentSegment |
| ...                    |                       |


#### スキーマのバージョニングに関するリファレンス
特定のスキーマ・バージョンにのみ適用されるルールは、最初の`Given`ステートメントでスキーマ・バージョンを指定しなければならない。


例えば、アライメント・エンティティはIFC4.3で導入されたもので、それ以前のバージョンのスキーマでは無効である。


```
Given A model with Schema "IFC4.3"
Given An IfcAlignment
Then ...
```

該当する場合は、複数のスキーマ・バージョンを指定することができる。

```
Given A model with Schema "IFC2X3" or "IFC4"
Given An IfcElement
Then ...
```

##### 有効な（撤回または引退していない）スキーマ・バージョン
| バージョン | 正式名称 | スキーマID | 一般名 |
|---------|---------------|-------------|-------------|
| 4.3.2.0 | IFC4.3ADD2 | IFC4X3_ADD2 | IFC4.3 |
| 4.0.2.1 | IFC4ADD2 TC1 | IFC4 | IFC4 |
| 2.3.0.1 | IFC2x3TC1 | IFC2X3 | IFC2x3 |

(22-write-python-steps)=
### 2.2) pythonのステップを書く
pythonのステップは、特徴ファイルで使用されているGherkin文法の実装（python言語を使用）です。  
Gherkinルールで使用したブランチと同じブランチで、以下の指示に従ってpythonステップを変更または追加してください。

**ファイル形式**：`.py`

**場所**：https://github.com/buildingSMART/ifc-gherkin-rules/tree/main/features/steps

#### pythonファイルの命名規則
今のところ、すべてのpythonステップは[steps.pyに](https://github.com/buildingSMART/ifc-gherkin-rules/blob/main/features/steps/steps.py)含まれています。従って、**新しいpythonファイルを作成する必要はありません**。

:construction: :construction: :construction：*将来、このファイルが大きくなったとき、pythonのステップは、ある基準(例えば、機能的な部分)を使って、より多くのファイルに分割されるかもしれません。そうなった場合、次のようになります: あなたのステップをホストするのに最適な .py ファイルを探し、ステップの追加を開始します*。


#### ステップパラメーター
新しいステップを作成する際には、パラメトライゼーションと将来的なステップの最適化について考えてください。

#### ステップの再利用
新しいステップを作る前に、似たようなものがすでに存在しないかチェックする。  
既存のステップを再利用するようにする。

#### "whenや" And"キーワードは使わない
when"キーワードは使用してはならない。  
And"キーワードは使用してはならない。  
その代わりに、Given"や"Then"を適宜繰り返す。

許可されるキーワードは以下の通り： `Given`、`Then`。

#### 既存のIfcOpenShellAPIの使用
`ifcopenshell.api`名前含まれる既存の機能は使用しないでください。








(23-write-unit-test-files)=
### 2.3) ユニットテストファイルの作成
ユニット・テスト・ファイルは、ルールを開発し、その動作をテストするために作成されるアトミックなIFCファイルです。  
Gherkinルールとpythonステップで使用した同じブランチで、以下の指示に従ってユニットテストファイルを作成します。**重要**：開発したすべてのルールは、ユニットテストファイルのセットを持たなければなりません。

**ファイル形式**`.ifc`

**場所**[:ifc-gherkin-rules/tree/main/test/files](https://github.com/buildingSMART/ifc-gherkin-rules/tree/main/test/files)

- test/filesフォルダに、ルールコード（例：ALB001）を使用してサブフォルダを作成します。
- このサブフォルダに、そのルールのユニットテスト・ファイルを追加する。

#### ユニットテスト・ファイルの命名規則
ユニットテストのファイルは、この命名規則に従わなければならない：

`Expected result``rule code``rule scenario``short_informative_description`.ifc。

あるいは、ルールにシナリオがない場合： `Expected result``rule code``short_informative_description`.ifc


<details><summary>例</summary>

```shell
pass-alb001-short_informative_description.ifc
fail-alb001-scenario01-short_informative_description.ifc
fail-alb001-short_informative_description.ifc
```

</details>


#### ユニットテストサブフォルダーの内容
ユニットテストのサブフォルダーには、以下を含める必要がある：

- すべてのユニットテストファイル (.ifc)
- READMEファイル（.md）を作成し、ファイルとその期待される動作を列挙する。以下の[テンプレート・テーブルを](#table-template-for-unit-test-files)使用する
- を使用する場合は、ユニットテストファイルを生成するために作成されたスクリプト (.py) を使用します。

#### 必要な単体テストの数
- 開発された各ルールには、ユニットテスト・ファイルのセットが必要です。
- 完全に準拠したユニットテスト・ファイルが少なくとも1つなければならない。
- フェイルファイルは、ルールのすべてのシナリオをカバーしなければならない。

(ユニットテスト・ファイル用テーブルテンプレート)=。
#### ユニットテスト・ファイル用テーブル・テンプレート
単体テストの期待結果を記述した表の例

| ファイル名 | 期待される結果 | エラーログ | 説明 |
|-------------------------------------------------------|-----------------|----------------------------------------------------------------------------------|----------------------------------------------------------------------------------|
| pass-alb002-アライメント・レイアウト | 成功 | n.a. |  |
| fail-alb002-scenario01-nested_attributes_IfcAlignment | 失敗 | インスタンスIfcAlignmentは、2つのインスタンスIfcAlignmentHorizontal... をネストしている。 | エラーは説明的なものですか、それともpytestのエラーそのものですか？もしその通りなら、複数行... |
| fail-alb002-シナリオ02-2_アラインメント | 失敗 | 以下の2つのインスタンスに遭遇した：IfcAlignment#23、IfcAlignment#906 | IfcAlignmentHorizontal、IfcAlignmentVertical、IfcAlignmentCantの場合 |
| フェイルアルバム002-シナリオ03-レイアウト | 失敗 | インスタンス#906=IfcAlignmentは#907=IfcWallをネストしている。 | シナリオ2のエラーを含む |
| fail-alb002-scenario04-alignment_segments | 失敗 | インスタンス#28=IfcAlignmentHorizontalは#906=IfcWallに割り当てられている。 | @todo IfcAlignmentVertical,IfcAlignmentCant.空のリスト／タイポと同様に？ |



## 4.プルリクエストにレビュアーを割り当てる
...
## 5.プルリクエストを確認する
...
## 6.(オプション) レビューアからのフィードバックに従ってルールを修正する
...
## 7.プルリクエストを承認してマージする
...

## 付録
(エラーコード)
### エラーコード
エラーコードは検証サービスの結果を分類し、分類するために使用され、[ifc-validation-data-model/main/models.py#L937に](https://github.com/buildingSMART/ifc-validation-data-model/blob/main/models.py#L937)実装されています。


| エラーコード | 説明 |
|------------|----------------------------------------|
| P00010 | 合格 |
| N00010 | 該当なし |
|  |  |
| E00001 | 構文エラー |
| E00002 | スキーマエラー |
| E00010 | タイプエラー |
| E00020 | エラー値 |
| E00030 | ジオメトリー・エラー |
| E00040 | カーディナリティ・エラー |
| E00050 | 重複エラー |
| E00060 | プレースメント・エラー |
| E00070 | 単位エラー |
| E00080 | 数量エラー |
| E00090 | 列挙値エラー |
| E00100 | 人間関係のエラー |
| E00110 | ネーミング・エラー |
| E00120 | リファレンスエラー |
| E00130 | リソースエラー |
| E00140 | 非推奨エラー |
| E00150 | 形状表現エラー |
| E00160 | インスタンス構造エラー |
|  |  |
| W00010 | アライメントはビジネス・ロジックのみを含む |
| W00020 | アライメントはジオメトリのみを含む |
| W00030 | 警告 |
|  |  |
| X00040 | 実行済み |

#### 備考
`Not Applicable`スキーマ・バージョンのために適用されないルールを指す。 `Executed`、スキーマ・バージョンのために適用されるルールであるが、モデルには特定のルールの一部として検証されたエンティティが含まれていないことを指す。



検証サービスのユーザーインターフェイスでは、どちらの結果も"該当なし"と報告されている。
