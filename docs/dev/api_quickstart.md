# APIクイックスタート
Validation Service APIのプレビューは`https://dev.validate.buildingsmart.org/api`から入手可能である。

## ドキュメンテーション
自動生成されたドキュメントは、[Swagger](https://dev.validate.buildingsmart.org/api/swagger-ui)形式と[Redocly](https://dev.validate.buildingsmart.org/api/redoc)形式の両方で利用できます。





## 認証トークン
APIを呼び出す前に認証トークンが必要です。この認証トークンは、[validate@buildingsmart.org。](mailto:validate@buildingsmart.org)


このトークンは、ベアラートークンとして使用することも、ユーザー名／Eメールと組み合わせてパスワードとして使用することもできます。

## 使用例
1. トークン認証とベーシック認証の違いを示すサンプル

    ``` shell
     curl -X GET --location 'https://dev.validate.buildingsmart.org/api/validationrequest/' --header 'Authorization: Token <TOKEN>'
    ```

   -または

    ```shell
    curl -X GET --location 'https://dev.validate.buildingsmart.org/api/validationrequest/' --header 'Authorization: Basic <HASH>'
    ```

   ここで `<HASH>`はBase64エンコードされたEメールとトークンをパスワードとして、コロンで区切って指定します。

1. `/validationrequest` エンドポイントに POST リクエストを送信して、新しい検証リクエストを開始します（ファイル名とファイルの内容が必要です）：

   ```shell
      curl -X POST --location 'https://dev.validate.buildingsmart.org/api/validationrequest/' \

      --header 'Authorization: Token <TOKEN>' \

      --form 'file_name="valid_file.ifc"' \

      --form 'file=@"/.../buildingSMART/sample_files/valid_file.ifc"'
   ```

   これは、今後のGETリクエストに使用できるID（public_id）を含むJSONオブジェクトを返します。

1. `/validationrequest` エンドポイントへの GET リクエストにより、1 つの ValidationRequest の詳細を取得する。

   ```shell
      curl -X GET --location 'https://dev.validate.buildingsmart.org/api/validationrequest/r767775526' --header 'Authorization: Token <TOKEN>'
   ```

1. `/validationrequest` エンドポイントへの GET リクエストにより、すべての ValidationRequests の詳細を取得する。

   ```shell
   curl -X GET --location 'https://dev.validate.buildingsmart.org/api/validationrequest/' --header 'Authorization: Token <TOKEN>'
   ```

1. `/validationtask` エンドポイントへの GET リクエストによって、2 つの ValidationRequests の ValidationTasks をすべて取得する。

   ```shell
   curl -X GET --location 'https://dev.validate.buildingsmart.org/api/validationtask/?request_public_id=r75257132,r383446691' --header 'Authorization: Token <TOKEN>'
   ```

1. `/validationoutcome` エンドポイントへの GET リクエストにより、1 つの ValidationRequest の結果をすべて取得する。

   ```shell
   curl -X GET --location 'https://dev.validate.buildingsmart.org/api/validationoutcome/?request_public_id=r75257132' --header 'Authorization: Token <TOKEN>'
   ```
