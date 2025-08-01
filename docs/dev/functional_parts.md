# 機能部品

ファンクショナル・パートとは、同じようなタイプの検証を含むルールを論理的にグループ化したものである。

これらの機能部接頭辞は、規範規則の命名に使用される。
and are reported in the results listed by the validation service.

## カタログ

以下は、IFCの機能部品のリストである。

| TAG | 機能部 | 説明 |
|-----|-----------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| ジェーシービー | プロジェクトの定義 | モデル内のオブジェクトの全体的なコンテキストとディレクトリを定義する機能。 特に、コンテキスト定義には、デフォルトの単位と形状表現の幾何学的表現コンテキストが含まれる。 |
| GRF | ジオリファレンス | 国家座標系や地球座標系などの参照座標系に対するモデルの地理的位置と方位を正確に定義する能力。 |
| BLT | 内蔵エレメント（キック可能） | 壁、床、屋根、階段、ドア、窓、柱、道路舗装、橋梁デッキ、鉄道軌道要素など、さまざまな建物やインフラ要素をモデル化する能力。 |
| 空対地ミサイル | アセンブリー（キック可） | 他の要素から構成される／他の要素によって構築される要素をモデル化する機能。 例えば、屋根は一連のプレハブトラス部品から組み立てられるかもしれない。 |
| スパ | スペース（キック不可） | 部屋、廊下、クリアランスゾーン、循環エリアなどの空間をモデル化する機能。 |
| ブイアールティー | 仮想エレメント（キック不可） | 空間要素をモデル化する機能で、架空の、プレースホルダ、または仮の領域（クリアランスなど）、体積、境界を提供するために使用される。 仮想要素は表示されず、量、材料、その他の尺度を持たない。 |
| ジェム | ジオメトリ表現 | パラメトリック、メッシュ、ボクセルベースの表現など、さまざまなジオメトリタイプを使用して構築要素や空間を表現する機能。 |
| オージェイ | オブジェクトの配置 | 建物モデル内の建築要素やシステムの位置、方向、規模を定義する能力。 |
| POS | ポジショニング・エレメント | 他の要素を相対的に配置するために使用される（仮想的な）オブジェクトを定義する機能。 グリッド、アライメント、リファレンスを含む。 |
| オージェーティー | オブジェクトのタイピング | モデル内のタイプや機能に基づいて要素（残念ながらシステムではない）を定義し、そのようなタイプの出現によって情報の再利用性を可能にする機能。 |
| グルタミン酸ナトリウム | グループ（キック不可） | 機能的または論理的な基準によって関連するオブジェクトをグループ化する機能。 HVAC（暖房、換気、空調）システムなどの建物システムのモデル化、同じメンテナンススケジュールを持つ資産のグループ化、防火システムの全端末のグループ化などに使用される。 |
| SPS | 地域別内訳 | 建築物やインフラプロジェクト内の空間構成を定義する機能。 敷地内の建物、建物内の階、階内の部屋など、空間コンテナ間の階層関係も含まれる。 |
| MAT | 材料 | 要素に割り当てられた素材を定義する機能。 |
| PSE | オブジェクトのプロパティ | 要素やシステムの性能特性などの特性を定義する能力。 |
| 数量 | オブジェクトの数量 | 寸法、体積、面積、重量など、要素の量を定義する能力。 |
| CLS | 分類の参照 | UNIFORMATやOmniclass分類システムなど、様々な分類システムに従って要素、材料、システムを分類する能力。 |
| ANN | 注釈 | ラベル、メモ、寸法などの注釈を要素やスペースに追加する機能。 |
| レイ | プレゼンテーション層 | レイヤー（CADレイヤーとも呼ばれる）を要素の集まりに割り当てる機能。 これは主にグループ化と可視性のコントロールに使用され、一般的にはジオメトリを表示または非表示にできるグループに整理するために使用される。 |
| シーティーエックス | プレゼンテーションの色と質感 | オブジェクトに色やテクスチャなどの外観情報を割り当てる機能。 |
| POR | ポート接続とネスティング | ポート（要素が他の要素に接続するための手段）と、2つのポート間の関係を定義する機能。 |
| STR | 構造項目と行動 | 構築された要素の解析理想化として、構造部材および構造接続を定義する機能。 また、基本荷重定義を使用して指定された構造物に関連する作用（力、変位など）および反作用（支持反力、内力、たわみなど）を定義する機能。 |
| CST | 原価計算 | 原価計算情報をオブジェクトに割り当てる機能。 |
| データリンク制御言語 | 活動のスケジューリング | オブジェクトにスケジュール情報を割り当てる機能。 |
| リブ | 図書館レファレンス | 製品ライブラリや外部データベースなどのライブラリエンティティをオブジェクトやオブジェクトタイプに関連付ける機能。 |
| DOC | ドキュメンテーション・リファレンス | ドキュメントへの参照をオブジェクトに関連付ける機能。 |
| CTR | 制約条件 | 最小・最大寸法やクリアランスなど、建築要素に対する制約をモデル化し、設計・施工中にこれらの制約を強制する機能。 |
| VER | バージョン管理／リビジョン管理 | 建物データの経時的な変更を追跡し、変更履歴を管理する機能。 |

*These 3 functional parts are further decomposed as indicated below.

### ジオメトリ表現のサブパーツ

| TAG | 機能部 | 説明 |
|-----|------------------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| AXG | 軸幾何学 | これは、軸のセットを使ってジオメトリを表現するもので、各軸は3D空間内の位置と方向ベクトルで構成されます。 これは、壁、梁、柱、その他の要素など、特定のパターンや方向に従うジオメトリを表現するのに便利です。 IFCでは、軸の位置と方向をそれぞれ定義するIfcCartesianPointクラスとIfcDirectionクラスを使用して、軸ジオメトリを表現できます。 |
| 筋萎縮性側索硬化症 | アライメント形状 | 道路、鉄道、パイプラインなどの直線オブジェクトを表現するのに便利である。 |
| PBG | 点ベースのジオメトリ | ポイント・ベースのジオメトリは、大規模な環境を表現したり、複雑なジオメトリを簡略化して表現したりするのに役立ちます。 ポイント・ベースのジオメトリは、3Dポイント・クラウドや散在したポイント・データなど、さまざまなポイント・タイプを使用してIFCで表現できます。 |
| TAS | テッセレーション（つまりメッシュ） | テッセレーションは、有機的な形状や不規則な形状を表現したり、より複雑な形状を低解像度で表現するのに役立ちます。 IFCでは、三角形、四角形、多面体メッシュなど、さまざまなメッシュタイプを使ってテッセレーションを表現できます。 |
| ブートレコード | バウンダリー・プレゼンテーション（BREP） | BREPジオメトリは、曲面や特殊なトポロジーを持つ複雑な形状を表現するために使用できます。 BREPジオメトリは、ポリゴンメッシュの場合はIfcFacetedBrepを、より複雑なソリッドジオメトリの場合はIfcManifoldSolidBrepを使用してIFCで表現できます。 これらのエンティティは、IfcCartesianPoint、IfcPolyLoop、IfcSurfaceなどのジオメトリ表現のコレクションを使用してさらに記述できます。 |
| CSG | コンストラクティブ・ソリッド・ジオメトリ（CSG） | CSGは、予測可能で再現性のある複雑な形状を作成するのに便利です。 CSGは、IFCではIfcBooleanResultエンティティを使って表現することができます。 |
| スイス | スイープ（エクストルージョン、ロフト、ブレンドなど） | これは、2D形状（多くの場合「プロファイル」と呼ばれる）を3Dパスに沿って移動させ、3D形状を生成するものです。 結果として得られる形状は通常、滑らかで連続的であり、その断面はパスの長さに沿って徐々に変化します。 スイープ形状は、パイプ、ケーブル、モールディングのような建築の細部など、さまざまなオブジェクトを表現するのに便利です。 IFCでは、スイープ形状はIfcExtrudedAreaSolid、IfcRevolvedAreaSolid、IfcSweptAreaSolidエンティティを使って表現できます。 |
| TFM | トランスフォーメーション | 変換ジオメトリは、回転、平行移動、スケーリングなどの空間変換を使用してジオメトリを表現します。 これにより、ゼロから新しいジオメトリを作成することなく、既存のジオメトリを変更することができます。 変換ジオメトリは、デカルト変換、配置変換、複合変換など、さまざまな種類の変換を表現するために使用できます。 これらの変換は、点、曲線、サーフェス、ソリッドなど、さまざまなジオメトリに適用して、新しいジオメトリや変更されたジオメトリを作成できます。 IFCでは、変換ジオメトリはIfcCCartesianTransformationOperatorエンティティおよびIfcObjectPlacementエンティティを使用して表現されます。 |
| BBX | バウンディングボックス | これは、直交ボックスを定義することを含み、そのボックスは、定義されたオブジェクト座標系の軸に平行で、ジオメトリオブジェクトを含み、後者の空間的範囲を定義する。 |
| 警視庁 | マップされたジオメトリ | これは、マッピングの目的のために、表現とその表現内の表現項目を特定することを含む。 表現項目は、マッピングの原点を定義する。 表現マップは、マッピングされた項目によるマッピングのソースとして使用される。 |
| RCO | 関係構成要素 | 接続ジオメトリ、スペース境界、干渉ジオメトリなど。 これはジオメトリのカテゴリーそのものではないが、IFCにおけるジオメトリの特定の使用方法である。 |
| CPD | 切り取られた表現 | これは、3Dモデルのビジュアライゼーションで、内部構造やコンポーネントを見やすくするために、モデルの一部を "クリッピング "または切り取ることを意味する。 これは、多くの場合、クリッピングプレーンや断面を使用して行われる。 |

### Object placement sub-parts

| TAG | 機能部 | 説明 |
|-----|------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| LOP | 現地プレースメント | ある製品の配置を、他の製品の配置との関係で定義する能力、またはプロジェクトの幾何学的表現コンテクスト内での絶対的な配置を定義する能力。 |
| リップ | リニア配置 | カーブに対する製品の配置を定義する能力。 |
| GDP | グリッド配置 | デザイングリッドに対する製品の配置を定義する機能。 |

### Positioning elements sub-parts

| TAG | 機能部 | 説明 |
|----------|-----------------|--------------------------------------------------------------------------------------------------------------------|
| GRD | グリッド | オブジェクト配置の基準となるデザイングリッドを定義する機能。 |
| ALA、ALB | アライメント | アライメント曲線とその構成要素を定義する機能。 これらはオブジェクト配置のリファレンスとして使用できます。 |
| 高速フーリエ変換 | 参照 | オブジェクトの配置に参照として使用する参照先を定義する機能。 |

