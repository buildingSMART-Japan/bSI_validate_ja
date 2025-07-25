# ガーキンルールの実装を深く掘り下げる

## 装飾家

### `@gherkin_ifc`

の代わりに使用される。`behave`デフォルト`@step_implementation`装飾家
to provide additional capabilities related to context stacking and other concerns
related to tracking and evaluating instances in the IFC model.

### `@register_enum_type`

これは、列挙型をよりシンプルな方法で登録するための、小さな新しいデコレーターである。

## ステップハンドリング

### `execute_step()`

現在処理中のステップが`Given`または`Then`.

### `handle_given()`

を扱う。`Given`ステップ

### `handle_then()`

を扱う。`Then`ステップ

## コンテキスト・スタッキング

ステップが処理されると、それらは以下の型の永続オブジェクトに取り込まれます。`behave.runner.Context`.
This context object includes a hidden attribute `_stack` that is used to 'stack' information
and results for each step that is processed.

の内容をモニターすることは有益である。`instances`の各アイテムの 
`context._stack` list.

---

## 特集タグ

BehaveのFeatureタグは、検証サービスにおけるGherkinベースのテストルールの実行を分類し、制御するために使用されます。 これらのタグは、各テストルールの先頭に配置されます。`.feature`ファイルを使い、コマンドラインインターフェイスから参照できる。`--tags`オプションを通してアクセスする。`context.tags`をステップの実装ファイルに追加する。

### `@informal_propositions`そして`@implementer-agreement`

これらのタグは**IFC規則**になる。**パッシング**,**失敗**または**該当なし**の結果を検証サービスに反映させる。

> **注：**これらのタグは1つのタグに統合される予定である：`@normative-rule`.

これらのルールだけをコマンドラインで実行するには

> python3 -m behave --no-capture -v --tags=@informal-propositions --define input=/path/to/your.ifc

### `@industry-practice`

このタグは以下を示す。**ベストプラクティスルール**これらの結果**パッシング**,**警告**あるいは**該当なし**結果

これらをローカルで実行する：
> python3 -m behave --no-capture -v --tags=@industry-practice --define input=/path/to/your.ifc

### `@disabled`

このタグが付いたルールは以下の通り。**使用不能**であり、検証サービスによって実行されることはない。

明示的に**除外**ルールの場合は、同じ --tags 変数とハイフンを使用する：
> python3 -m behave --no-capture -v --tags=@informal-propositions --tags=-@disabled --define input=/path/to/your.ifc



### `@AAA000`

このタグは**機能部**(`AAA`)と**ルール番号**(`000`例えば

- ジオリファレンス機能部分の4つ目のルール（以下のことを考慮する。`@GRF000`とタグ付けされる。`@GRF003`.


### `@versionX`

このタグは`.feature`ファイルである。
The version number gets incremented whenever meaningful changes are made to the rule after its release.
A 'meaningul' change is one that could result in different outcomes for the same IFC model.

誤字脱字の修正、ステップ実装への制御文字の追加などのマイナーな変更
are not considered a 'meaningful' change as they do not affect the end results of the validation process.


### `@no-activation`

このタグは、合格した結果が機能部分をアクティブにしないことを保証します。 代わりに、それは適用されないとマークされます。

例えば、地理参照ルールである GRF003 は、すべての IfcFacility が IfcCoordinateReferenceSystem にリンクされているかどうかを検証する。 しかし、IfcFacility の存在は、必ずしもそのファイルが地理参照を含むことを意図していることを意味しない。 したがって、地理参照が必要でない場合、このルールは、ソフトウェア認証スコアカード上でその機能部分を緑色にマークする「合格」ステータスをトリガーすべきではない。

ルールに@no-activationのタグを付けると、データベースに記録された情報がフィルタリングされ、ルールは合格とマークされない。 

**このタグはエラーの結果に影響しない**- ルールが失敗しても、エラーは発生し、通常通り記録される。
