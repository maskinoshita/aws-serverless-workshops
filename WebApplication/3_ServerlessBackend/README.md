# Module 3: Serverless Service Backend

このモジュールでは、AWS LambdaとAmazon DynamoDBを使用して、Webアプリケーションからのリクエストを処理するためのバックエンド処理を構築します。最初のモジュールにデプロイしたブラウザアプリケーションを使用すると、ユーザーはユニコーンを選択した場所に配送するようにリクエストできます。このリクエストの要求に応えるためには、ブラウザで実行されているJavaScriptがクラウド内で実行されているサービスを呼び出す必要があります。

ユーザーがユニコーンを要求するたびに呼び出されるLambda関数を実装します。この機能は、プールからユニコーンを選択し、その要求をDynamoDBテーブルに記録し、フロントエンドアプリケーションに派遣されるユニコーンに関する詳細を応答します。

![Serverless backend architecture](../images/serverless-backend-architecture.png)

この関数は、Amazon API Gatewayを使用してブラウザから呼び出されます。その機能は次のモジュールで実装します。このモジュールでは、関数を単独でテストするだけです。

次のモジュールに進む必要がある場合は、 [module 4 (RESTful APIs)](../4_RESTfulAPIs)の**launch the stack**を選択することでこのモジュールをスキップできます。

## 実装手順

以下の各セクションでは、実装の概要と詳細なステップバイステップの手順を説明します

AWS管理コンソールに精通している場合や、段階的な説明に従わずに自身でサービスを探索したい場合は、概要は実装を完了するのに十分な内容を提供しています。

最新バージョンのChrome、Firefox、SafariのWebブラウザを使用している場合は、セクションを展開するまで、ステップバイステップの手順は表示されません。

### 1. Amazon DynamoDB テーブルを作成する

Amazon DynamoDBコンソールを使用して、新しいDynamoDBテーブルを作成します。あなたのテーブル `Rides`とし、String型の` RideId`というパーティションキーを与えます。テーブル名とパーティションキーでは大文字と小文字が区別されます。正確なIDを使用してください。他のすべての設定にはデフォルトを使用してください。

テーブルを作成したら、次のステップで使用するARNをメモしてください。

<details>
<summary><strong>ステップバイステップ手順 (詳細を展開)</strong></summary><p>

1. AWS マネージメントコンソールで **サービス** から データベースの下にある **DynamoDB** を選択します。

1. **テーブルの作成**をクリックします。

1. **テーブル名** に`Rides`を入力します。小文字／大文字は区別されます。

1. **パーティションキー** に`RideId`を入力し、**文字列**をキータイプに設定します。小文字／大文字は区別されます。

1. **デフォルト設定の使用** を`チェック`し、**作成**　を押します。

    ![Create table screenshot](../images/ddb-create-table.png)

1. 新しいテーブルの概要セクションの一番下までスクロールし、**ARN**を確認します。次のセクションでこれを使用します。

</p></details>


### 2. Lambda 関数のIAMロールを作成する

#### Background

すべてのLambda関数にはIAMロールが関連付けられています。このロールは、この関数が他のAWSサービスと対話することを許可するサービスを定義します。このワークショップでは、Lambda関数に、Amazon CloudWatchログにログを書き込み、DynamoDBテーブルにアイテムを書き込むためのアクセス権を与えるIAMロールを作成する必要があります。

#### 詳細な手順

新しいロールを作成するには、IAMコンソールを使用します。それに`WildRydesLambda`と名前をつけ、ロールタイプとしてAWS Lambdaを選択します。 Amazon CloudWatchログに書き込み、項目をDynamoDBテーブルに入れるために、権限を与えるポリシーをアタッチする必要があります。

`AWSLambdaBasicExecutionRole`という管理ポリシーをこのロールにアタッチして、必要なCloudWatchログ権限を付与します。また、前のセクションで作成したテーブルの `ddb：PutItem`アクションを可能にする独自のインライン・ポリシーを作成してください。

<details>
<summary><strong>ステップバイステップ手順 (詳細を展開)
</strong></summary><p>

1. AWS マネージメントコンソールで **サービス** から セキュリティ、 アイデンティティ、 コンプライアンスの下にある **IAM** を選択します。

1. 左のナビゲーションバーから **ロール**を選択し、**ロールの作成**を選択します。

1.  **AWS サービス** グループを選択し、`このロールを使用するサービスを選択`で**Lambda**を選び、**次のステップ: アクセス権限**をクリックします。

    **Note:** AWSロールタイプを選択すると、ロールの信頼ポリシーが自動的に作成され、AWSサービスがあなたの代わりにこのロールを引き受けることができます。CLI、AWS CloudFormationまたは別のメカニズムを使用してこのロールを作成する場合は、信頼ポリシーを直接指定します。

1. **検索**テキストボックスに`AWSLambdaBasicExecutionRole`と入力し、そのロールの横にあるチェックボックスをオンにします。

1. **次のステップ: 確認** をクリックします。

1. **ロール名に**に`WildRydesLambda`。

1. **ロールの作成**　をクリックします。

1. ロールのページで`WildRydesLambda`を検索ボックスに入力し、作成したロールを選択します。

1. アクセス権限のタブで, 右下にある**インラインポリシーの追加** を選択します。
    ![Inline policies screenshot](../images/inline-policies.png)

1. **サービスの選択** をクリックします。

1.  **サービスの検索**に `DynamoDB` と入力し、表示された中から**DynamoDB** を選びます。
    ![Select policy service](../images/select-policy-service.png)

1. **アクションの選択** をクリックします。

1. **フィルタアクション**に `PutItem` と入力し、表示された中から **PutItem** を選びます。

1. **リソース** セクションをクリックします。

1. **指定** オプションの選択をし, **テーブル** セクション中の`ARNの追加`リンクをクリックします。

1. 前のセクションで作成したテーブルのARNを**DynamoDB_table の ARN の指定**に貼り付け、**追加**を選択します。

1. **Review Policy** を選択します。

1. 名前に`DynamoDBWriteAccess`を入力し、**Create policy** を選択します。
    ![Review Policy](../images/review-policy.png)

</p></details>

### 3. リクエストを処理するためのLambda関数を作成する

#### Background

AWS Lambdaは、HTTPリクエストなどのイベントに応答してコードを実行します。このステップでは、ユニコーンを配送するためのWebアプリケーションからのAPIリクエストを処理するコア機能を構築します。次のモジュールでは、Amazon API Gatewayを使用して、ユーザーのブラウザから呼び出せるHTTPエンドポイントを公開するRESTful APIを作成します。このステップで作成したラムダ関数をそのAPIに接続して、Webアプリケーション用の完全な機能を持つバックエンドを作成します。

#### 詳細な手順

AWS Lambdaコンソールを使用して、APIリクエストを処理する「RequestUnicorn」という新しいラムダ関数を作成します。関数コードには、提供されている[requestUnicorn.js](requestUnicorn.js)の実装例を使用してください。そのファイルからコピーしてAWS Lambdaコンソールのエディタに貼り付けるだけです。

前のセクションで作成した `WildRydesLambda` IAMロールを使用するように関数を設定してください。

<details>
<summary><strong>ステップバイステップ手順 (詳細を展開)
</strong></summary><p>

1. AWS マネージメントコンソールで **サービス** から コンピューティングの下にある **Lambda** を選択します。

1. **関数の作成**をクリックします。

1. **一から作成** カードを選択します。

1. **名前** フィールドに`RequestUnicorn`を入力します。

1. **ランタイム**に**Node.js 6.10**を選択します。

1. **ロール** ドロップダウンに`既存のロールを選択` が選択されていることを確認します。

1. **既存のロール** ドロップダウンから`WildRydesLambda`を選択します。
    ![Create Lambda function screenshot](../images/create-lambda-function.png)

1. **関数の作成**をクリックします。

1. **関数コード** セクションまでスクロールし、既存の**index.js** の内容を [requestUnicorn.js](requestUnicorn.js)で上書きします。
    ![Create Lambda function screenshot](../images/create-lambda-function-code.png)

1. ページの右上端にある**"保存"**をクリックします。

</p></details>

## 実装確認

このモジュールでは、AWS Lambdaコンソールを使用して構築した関数をテストします。次のモジュールでは、APIゲートウェイを備えたREST APIを追加して、最初のモジュールでデプロイしたブラウザベースのアプリケーションから関数を呼び出すことができます。

1. 関数のメイン編集画面から、`テストイベントの選択...` ドロップダウンから**テストイベントの設定** を選択します。
    ![Configure test event](../images/configure-test-event.png)

1. **新しいテストイベントの作成**を選択します。

1. **イベント名** に `TestRequestEvent` を入力します。

1. 下記のテストイベントをエディターにコピーアンドペーストします:

    ```JSON
    {
        "path": "/ride",
        "httpMethod": "POST",
        "headers": {
            "Accept": "*/*",
            "Authorization": "eyJraWQiOiJLTzRVMWZs",
            "content-type": "application/json; charset=UTF-8"
        },
        "queryStringParameters": null,
        "pathParameters": null,
        "requestContext": {
            "authorizer": {
                "claims": {
                    "cognito:username": "the_username"
                }
            }
        },
        "body": "{\"PickupLocation\":{\"Latitude\":47.6174755835663,\"Longitude\":-122.28837066650185}}"
    }
    ```

    ![Configure test event](../images/configure-test-event-2.png)

1. **作成** をクリックします。

1. 関数の編集画面で、ドロップダウンに`TestRequestEvent`が選択された状態で**テスト**をクリックします。

1. ページのトップにスクロールし、**実行結果** セクションの**詳細**を展開します。

1. 実行が成功し、関数の結果が次のようになっていることを確認します。
```JSON
{
    "statusCode": 201,
    "body": "{\"RideId\":\"SvLnijIAtg6inAFUBRT+Fg==\",\"Unicorn\":{\"Name\":\"Rocinante\",\"Color\":\"Yellow\",\"Gender\":\"Female\"},\"Eta\":\"30 seconds\"}",
    "headers": {
        "Access-Control-Allow-Origin": "*"
    }
}
```

Lambdaコンソールを使って新しい関数を正常にテストしたら、次のモジュール[RESTful APIs](../ 4_RESTfulAPIs)に進むことができます。
