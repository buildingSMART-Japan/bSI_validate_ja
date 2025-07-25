# 規範規則の詳細情報

バリデーション・サービスに新しいルールを追加するには、以下の手順に従ってください。

| n. | ステップ | 責任 |
|----|------------------------------------------------------------------------------------------|-----------------------------|
| 1 | bSI ifc-gherkin-rules リポジトリに新しいブランチを作成する。 | bSIバリデーションサービスチーム |
| 2 | このブランチでは、必要なルールの開発を開始する。 | ルール開発者 |
| 3 | プルリクエストを作成し、サンドボックス環境を使用してルールの動作をさらにテストする。 | ルール開発者 |
| 4 | プルリクエストにレビュアーを割り当てます。 | ルール開発者 |
| 5 | プルリクエストを確認する | bSIバリデーションサービスチーム |
| 6 | (オプション) レビューアからのフィードバックに従ってルールを修正する。 | ルール開発者 |
| 7 | プルリクエストを承認してマージする | bSIバリデーションサービスチーム |

## 1. Branch creation

ビルSMARTでは[すべてのルールを含むGitHubリポジトリ](https://github.com/buildingSMART/ifc-gherkin-rules)新しいルールの開発に使用するブランチを作成する。

- ブランチに新しいルールの名前を付ける。 例．`GEM900`ジオメトリー機能部の新しいルールについて
- レビューを容易にするために、ブランチごとに1つのルールを追加する（1ルール＝1`.feature`ファイル)

## 2. ルール開発

ルールが完成したとみなされる：

- ガーキン[**フィーチャーファイル**](21-write-feature-files-gherkin-rules-for-ifc)
- 対応するpython実装（別名、[**パイソンステップ**](22-write-python-steps))
- 一組の[**単体テストファイル**](23-write-unit-test-files)

以下は、これら3つのコンポーネントすべてについての説明である。

(21-write-feature-files-gherkin-rules-for-ifc)=。
### 2.1) IFC用のフィーチャーファイル（ガーキンルール）を書く

フィーチャーファイルとは、ルールの動作を記述した、ガーキン構文で記述されたファイルのことである。
In the branch just created, add a Gherkin feature file following these instructions.

**ファイル形式**:`.feature`

**所在地**https://github.com/buildingSMART/ifc-gherkin-rules/tree/main/features

#### 機能ファイルの命名規則

- ファイル名はrule code_rule title。
- ルールコードは3桁の大文字で構成される。[機能部品](./functional_parts.md)) + 3桁の数字
- ルールコードとルールタイトルは一意でなければならない。
- ルールのタイトルにはスペースを入れず、以下のようにする。`-`セパレータとして

<details><summary>wrong</summary>

```
SPS001 - Basic-spatial-structure-for-buildings.feature
SPS001_Basic spatial structure for buildings.feature
SPS001 - Basic spatial structure for buildings.feature
```
</details>
<details><summary>right</summary>

```
SPS001_Basic-spatial-structure-for-buildings.feature
```
</details>

#### 必須コンテンツ

`.feature`ファイルである：
- は、検証カテゴリーを分類するために、これらのタグを1つだけ含まなければならない：
    - `@critical`
    - `@implementer-agreement`
    - `@informal-proposition`
    - `@industry-practice`(警告；合否ではない）
- は、機能部品に3文字のアルファタグを付けなければならない。 を参照のこと。[機能部品](./functional_parts.md)
- には、フィーチャーファイルのバージョンを 1 ベースの整数で示すタグが 1 つ含まれていなければなりません。
  - 例`@version1`機能ファイルの初期バージョン用
  - 例`@version3`フィーチャーファイルの3番目のバージョン
    - 誤字脱字の修正や説明文の書き直しなどの軽微な変更では、バージョンは上がりません。
    - を変更した。**"与えられた"**または**「それから**ステートメント、またはステップの実装では、バージョン番号を1インクリメントする必要がある。
- を示す1つ以上のタグが含まれていなければならない。[エラーコード](error-codes)上がる
  - すべてのシナリオで同じエラーが発生する場合は、このタグを**特集**ライン

    <details><summary>example</summary>

    ```
    @implementer-agreement
    @GRF
    @version1
    @E00050
    Feature: GRF001 - Identical....
    ```

    </details>

    - いくつかのシナリオが異なるエラーコードを発生させる場合、このタグをそれぞれの**"シナリオ "のすぐ上に配置する。
      ** line

    <details><summary>example</summary>

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
- フィーチャーの命名規則は、ルールコード - ルールタイトル（ファイル名と同じ）です。 ルールタイトルには、以下の代わりに空白を使用する必要があります。`-` 

<details><summary>wrong</summary>

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
<details><summary>right</summary>

```
@ALB
Feature: ALB001 - Alignment Layout

Given ...
Then ...
```
</details>

 - を含まなければならない。**規則の説明**で始まる。 

<details><summary>example</summary>

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
フィーチャファイルのルールが特定の IFC バージョンおよび/またはビュー定義にのみ適用される場合、フィーチャファイル（またはシナリオが複数ある場合はその各シナリオ）は、以下のステップの適用可能性を指定する Given ステップで開始する必要があります。

<details><summary>examples</summary>

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
`.feature`ファイルである：
- 1つ以上のシナリオを含むことができる
- シナリオタイトルに制約はない
- を含むことができる。`@disabled`タグで一時的に処理から外す

#### ステップとステップの間にスペースがない

<details><summary>wrong</summary>

```
Given A model with Schema "IFC4.3"

Then Each IfcAlignmentHorizontal must be nested only by 1 IfcAlignment
Then Each IfcAlignmentVertical must be nested only by 1 IfcAlignment
Then Each IfcAlignmentCant must be nested only by 1 IfcAlignment
```
</details>
<details><summary>right</summary>

```
Given A model with Schema "IFC4.3"
Then Each IfcAlignmentHorizontal must be nested only by 1 IfcAlignment
Then Each IfcAlignmentVertical must be nested only by 1 IfcAlignment
Then Each IfcAlignmentCant must be nested only by 1 IfcAlignment
```
</details>

#### 余分な空白に注意

<details><summary>wrong</summary>

```
Given A model with Schema "IFC4.3"
Then Each IfcAlignmentHorizontal must be nested only by 1 IfcAlignment
Then  Each IfcAlignmentVertical must be nested only by 1 IfcAlignment
Then  Each IfcAlignmentCant must be nested only by 1 IfcAlignment
```
</details>
<details><summary>right</summary>

```
Given A model with Schema "IFC4.3"
Then Each IfcAlignmentHorizontal must be nested only by 1 IfcAlignment
Then Each IfcAlignmentVertical must be nested only by 1 IfcAlignment
Then Each IfcAlignmentCant must be nested only by 1 IfcAlignment
```
</details>

#### ステップの最後に句読点を使用しないでください。

<details><summary>wrong</summary>

```
Given A model with Schema "IFC4.3",
Then Each IfcAlignmentHorizontal must be nested only by 1 IfcAlignment;
Then Each IfcAlignmentVertical must be nested only by 1 IfcAlignment;
Then Each IfcAlignmentCant must be nested only by 1 IfcAlignment.
```
</details>
<details><summary>right</summary>

```
Given A model with Schema "IFC4.3"
Then Each IfcAlignmentHorizontal must be nested only by 1 IfcAlignment
Then Each IfcAlignmentVertical must be nested only by 1 IfcAlignment
Then Each IfcAlignmentCant must be nested only by 1 IfcAlignment
```
</details>

#### 大文字と小文字は区別される！

<details><summary>wrong</summary>

```
Given A model with schema "IFC4.3",
```
</details>
<details><summary>right</summary>

```
Given A model with Schema "IFC4.3"
```
</details>

#### Must vs Shall
用途**マスト**ではない。**は**要件を課すこと。
[ALB001_Alignment-in-spatial-structure.feature](https://github.com/buildingSMART/ifc-gherkin-rules/blob/main/features/ALB002_Alignment-layout.feature)
"Shall" is ambiguous, also in the legal field the community is moving to a strong preference for “must” as the clearest way to express a requirement or obligation.

<details><summary>wrong</summary>

```
Given A model with Schema "IFC2X3"
Given A file with Model View Definition "CoordinationView"
Then There shall be exactly 1 IfcSite element(s)
```
</details>
<details><summary>right</summary>

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



#### Reference for schema versioning

特定のスキーマ・バージョンにのみ適用される規則は、次のように指定しなければならない。
the schema version with the initial `Given` statement.

例えば、アライメント・エンティティはIFC4.3で導入されたもので、無効である。
in earlier schema versions.

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
| 4.3.2.0 | IFC4.3 ADD2 | IFC4X3_ADD2 | IFC4.3 |
| 4.0.2.1 | IFC4 ADD2 TC1 | IFC4 | IFC4 |
| 2.3.0.1 | IFC2x3 TC1 | IFC2X3 | IFC2x3 |

(22-write-python-steps)=
### 2.2) pythonのステップを書く 

pythonのステップは、特徴ファイルで使用されているGherkin文法の実装（python言語を使用）です。
In the same branch used for the Gherkin rules, change or add python steps following these instructions.

**ファイル形式**:`.py`

**所在地**https://github.com/buildingSMART/ifc-gherkin-rules/tree/main/features/steps

#### pythonファイルの命名規則

今のところ、すべてのpythonステップは[steps.py](https://github.com/buildingSMART/ifc-gherkin-rules/blob/main/features/steps/steps.py)したがって**新しいpythonファイルは作らず、既存のファイルを拡張してください。**

建設: :建設: :建設：
*In the future, when this file grows, python steps may be splitted in more files - using a certain criteria (e.g., functional parts). When this will be the case, the instruction will be: locate the best .py file to host your steps and start adding your steps*

#### ステップパラメーター

新しいステップを作成する際には、パラメトライゼーションと将来的なステップの最適化について考えてください。

#### ステップの再利用

新しいステップを作る前に、似たようなものが既に存在しないかチェックする。
Try to reuse existing steps.

#### when "や "And "のキーワードは使わないこと。

when」キーワードは使用してはならない。
The "And" keyword must not be used.
Instead, repeat the "Given" or "Then" as appropriate.

使用可能なキーワードは以下の通り：`Given`そして`Then`.

#### 既存のIfcOpenShell APIの使用

に含まれる既存の機能を使用しないようにしてください。`ifcopenshell.api`の名前空間を使用します。








(23-write-unit-test-files)=
### 2.3) ユニットテストファイルの作成 

ユニット・テスト・ファイルは、ルールを開発し、その動作をテストするために作成されるアトミックなIFCファイルです。
In the same branch used for the Gherkin rules, and python steps, create unit test files following these instructions. **IMPORTANT**: every rule developed must have a set of unit test files.

**ファイル形式**:`.ifc`

**所在地**:[ifc-gherkin-rules/tree/main/test/files](https://github.com/buildingSMART/ifc-gherkin-rules/tree/main/test/files)

- test/filesフォルダに、ルールコード（例：ALB001）を使用してサブフォルダを作成します。
- このサブフォルダに、そのルールのユニットテスト・ファイルを追加する。

#### ユニットテスト・ファイルの命名規則

ユニットテストのファイルは、この命名規則に従わなければならない：

`Expected result`-`rule code`-`rule scenario`-`short_informative_description`.ifc

あるいは、ルールにシナリオがない場合：
`Expected result`-`rule code`-`short_informative_description`.ifc

<details><summary>Examples</summary>

```shell
pass-alb001-short_informative_description.ifc
fail-alb001-scenario01-short_informative_description.ifc
fail-alb001-short_informative_description.ifc
```

</details>


#### ユニットテストサブフォルダーの内容

ユニットテストのサブフォルダーには、以下を含める必要がある：

- すべてのユニットテストファイル (.ifc)
- READMEファイル(.md)には、ファイルとその期待される動作が記載されています。[テンプレートテーブル](#table-template-for-unit-test-files)以下
- を使用する場合、ユニットテストファイルを生成するために作成されたスクリプト（.py）です。 

#### 必要な単体テストの数

- 開発された各ルールには、ユニットテスト・ファイルのセットが必要です。
- 完全に準拠したユニットテスト・ファイルが少なくとも1つなければならない。
- フェイルファイルは、ルールのすべてのシナリオをカバーしなければならない。

(ユニットテスト・ファイル用テーブルテンプレート）=。
#### ユニットテスト・ファイル用テーブル・テンプレート

単体テストの期待結果を記述した表の例

| ファイル名 | 期待される結果 | エラーログ | 説明 |
|-------------------------------------------------------|-----------------|----------------------------------------------------------------------------------|----------------------------------------------------------------------------------|
| pass-alb002-アライメント・レイアウト | 成功 | n.a. |  |
| fail-alb002-scenario01-nested_attributes_IfcAlignment | 失敗 | インスタンスIfcAlignmentは、2つのインスタンスIfcAlignmentHorizontal ... をネストしている。 | エラーは説明的なものですか、それともpytestのエラーと同じですか？ もし同じなら、複数の行が... |
| fail-alb002-シナリオ02-2_アラインメント | 失敗 | 次の2つのインスタンスが発生した：IfcAlignment #23、IfcAlignment #906 | IfcAlignmentHorizontal、IfcAlignmentVertical、IfcAlignmentCantの場合 |
| フェイルアルバム002-シナリオ03-レイアウト | 失敗 | インスタンス#906=IfcAlignmentは#907=IfcWallをネストしている。 | シナリオ2のエラーを含む |
| fail-alb002-scenario04-alignment_segments | 失敗 | インスタンス#28=IfcAlignmentHorizontalは#906=IfcWallに割り当てられている。 | リストやタイポの空白と同様に、@todo IfcAlignmentVertical, IfcAlignmentCant? |

## 4. Assign a reviewer to the pull request
...
## 5. プルリクエストを確認する
...
## 6. (オプション) レビューアからのフィードバックに従ってルールを修正する。
...
## 7. プルリクエストを承認してマージする
...

## 付録

(エラーコード
### エラーコード

エラーコードは、検証サービスの結果を分類し、分類するために使用される。
implemented in [ifc-validation-data-model/main/models.py#L937](https://github.com/buildingSMART/ifc-validation-data-model/blob/main/models.py#L937).

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

#### Notes

`Not Applicable`は、スキーマのバージョンによって適用されないルールを指す。
`Executed` refers to a rule that does apply because of schema version,
but the model does not contain any entities validated as part of a particular rule.

どちらの結果も、検証サービスのユーザーインターフェイスでは「該当なし」と報告される。