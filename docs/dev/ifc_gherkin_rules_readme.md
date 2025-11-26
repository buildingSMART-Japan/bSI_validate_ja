# IFC ガーキンのルール
## buildingSMART検証サービスの一部としての利用
このリポジトリは、検証サービス全体における3つのサブモジュールのうちの1つである。  
参照 [application_structure](#application-structure)を参照のこと。

## 変更を加える
このリポジトリで開発されたルールは、Gherkinとそのpython実装の動作の一般的なアイデアに従っています。

つまり、人間が読めるルールの定義とPythonの実装があるということだ。

このリポジトリの3つ目の要素は、期待される結果を備えた最小限のサンプルファイルである。これは、既存の機能を壊さないという確信を持って、拡張や修正を提案できることを意味する。

## コマンドラインの使用法
IfcOpenShell で実装されたステップを用いたIFC 建物モデルの自動検証のために、Gherkin で記述された非公式命題と実装者合意。

```shell
$ python -m ifc-gherkin-rules ifc-gherkin-rules\test\files\gem001\fail-gem001-cube-advanced-brep.ifc
Feature: Shell geometry propositions/IfcClosedShell.v1
    URL: /blob/8dbd61e/features/geometry.shells.feature

   Step: Every oriented edge shall be referenced exactly 1 times by the loops of the face

       * #29=IfcClosedShell((#104,#115,#131,#147,#163,#179))
         On instance #29=IfcClosedShell((#104,...,#179)) the edge (-0.183012701892219,
         1. 683012701892219, 1.0) -> (-0.683012701892219, -0.183012701892219, 1.0) was
         referenced 2 times

       * #29=IfcClosedShell((#104,#115,#131,#147,#163,#179))
         On instance #29=IfcClosedShell((#104,...,#179)) the edge (-0.5, -0.5, 0.0) ->
         (-0.5, 0.5, 0.0) was referenced 2 times

       * #29=IfcClosedShell((#104,#115,#131,#147,#163,#179))
         On instance #29=IfcClosedShell((#104,...,#179)) the edge (-0.5, 0.5, 0.0) ->
         (0.5, 0.5, 0.0) was referenced 2 times

       * #29=IfcClosedShell((#104,#115,#131,#147,#163,#179))
         On instance #29=IfcClosedShell((#104,...,#179)) the edge (-0.683012701892219,
         -0.183012701892219, 1.0) -> (0.183012701892219, -0.683012701892219, 1.0) was
         referenced 2 times

       * #29=IfcClosedShell((#104,#115,#131,#147,#163,#179))
         On instance #29=IfcClosedShell((#104,...,#179)) the edge (0.183012701892219,
         -0.683012701892219, 1.0) -> (0.683012701892219, 0.183012701892219, 1.0) was
         referenced 2 times

       * #29=IfcClosedShell((#104,#115,#131,#147,#163,#179))
         On instance #29=IfcClosedShell((#104,...,#179)) the edge (0.5, -0.5, 0.0) ->
         (-0.5, -0.5, 0.0) was referenced 2 times

       * #29=IfcClosedShell((#104,#115,#131,#147,#163,#179))
         On instance #29=IfcClosedShell((#104,...,#179)) the edge (0.5, 0.5, 0.0) ->
         (0.5, -0.5, 0.0) was referenced 2 times

       * #29=IfcClosedShell((#104,#115,#131,#147,#163,#179))
         On instance #29=IfcClosedShell((#104,...,#179)) the edge (0.683012701892219,
         1. 183012701892219, 1.0) -> (-0.183012701892219, 0.683012701892219, 1.0) was
         referenced 2 times
```
