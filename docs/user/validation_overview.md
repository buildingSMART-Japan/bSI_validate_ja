# 検証プロセスを理解する

IFCファイルが与えられると、バリデーション・サービスは適合性の判断を提供します。
against the IFC standard - including schema and specification

## STEP構文

検証プロセスの最初のステップでは、アップロードされたファイルを見て、以下のことを確認します。
it is a valid STEP Physical File (SPF) in accordance with [ISO 10303-21](https://www.iso.org/standard/63141.html).

## IFCスキーマ

スキーマ検証は2つの部分で構成される：

1. スキーム・バージョン
2. コンプライアンス・スキーム

### スキーム・バージョン

このチェックでは、スキーマ識別子が以下のいずれかであることを確認する：

- `IFC2X3`
- `IFC4`
- `IFC4X3_ADD2`

### コンプライアンス・スキーム

スキーマコンプライアンスは、EXPRESSスキーマで定義されている以下の点をチェックする：

 - エンティティの属性が正しく入力され、属性の数が正しく、集約の場合は正しいタイプとカーディナリティが入力されている。
 - 逆属性は正しく入力され、正しいカーディナリティを持つ。
 - エンティティ・スコープ`WHERE`規則
 - グローバルルール

このチェックは、指定されたスキーマ・バージョンに含まれていないエンティティ・タイプや、抽象エンティティのインスタンス化にもフラグを立てる。

例えば、こうだ：`IfcAlignment`エンティティはスキーマのバージョンでのみ有効です。`IFC4X3_ADD2`,
so it is not valid as part of a file with schema version `IFC2X3`.

## 規範チェック

規範的なチェックには2つのカテゴリーがある：

1. インプリメンター契約
2. 非公式な提案

### インプリメンター契約

これらは、ソフトウェア実装者の間で公式な合意として批准されている規範的なチェックである。

### 非公式な提案

これらは実施協定として批准されていない規範的なチェックである、
but are still considered mandatory for a file to be considered valid.

## 追加的、規範的でないチェック

### 業界慣行

このステップでは、IFCファイルを一般的な慣行や常識的なデフォルトと照らし合わせてチェックします。
None of these checks render the IFC file invalid.
Therefore, any issues identified result in warnings rather than errors.

### buildingSMARTデータディクショナリ（bSDD）への準拠

```{note}
bSDD Checks are temporarily disabled as of v0.6.6.
```

