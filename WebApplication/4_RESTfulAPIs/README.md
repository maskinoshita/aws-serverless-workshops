# Module 4: AWS Lambda と Amazon API Gateway を使用したRESTful API

このモジュールでは、APIゲートウェイを使用して、前のモジュールで構築したLambda関数をRESTful APIとして公開します。このAPIはインターネットでパブリックアクセス可能です。このAPIは前のモジュールで作成したAmazon Cognitoユーザープールを使用して保護されます。この設定を行うことで、公開されたAPIにAJAX呼び出しを行うクライアントサイドJavaScriptを追加することで、静的にホストされているWebサイトを動的Webアプリケーションに変えることができます。

![Dynamic web app architecture](../images/restful-api-architecture.png)

上の図は、このモジュールで構築するAPIゲートウェイコンポーネントを、以前構築した既存のコンポーネントと統合する方法を示しています。グレー表示された項目は、前の手順で既に実装した項目です。

最初のモジュールにデプロイした静的Webサイトには、既にこのモジュールでビルドするAPIと対話するように設定されたページがあります。 `/ride.html`のページには、ユニコーンライドをリクエストするための簡単なマップベースのインターフェースがあります。 `/signin.html`ページを使用して認証した後、ユーザーはマップ上のポイントをクリックし、右上の「Unicorn Request」ボタンを選択して乗車をリクエストすることで、ピックアップの場所を選択できます。

このモジュールでは、APIのクラウドコンポーネントを構築するために必要なステップに焦点を当てますが、このAPIを呼び出すブラウザコードの仕組みに興味がある場合は、[ride.js](../1_StaticWebHosting/website/js/ride.js)ファイルを開きます。この場合、アプリケーションはjQueryの[ajax()](https://api.jquery.com/jQuery.ajax/)メソッドを使用してリモートリクエストを行います。

モジュールをスキップしたい場合は、必要なリソースを自動的に構築するために、これらのAWS CloudFormationテンプレートの1つを選択した地域で起動できます。

Region| Launch
------|-----
Asia Pacific (Tokyo) | [![Launch Module 4 in ap-northeast-1](http://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/images/cloudformation-launch-stack-button.png)](https://console.aws.amazon.com/cloudformation/home?region=ap-northeast-1#/stacks/new?stackName=wildrydes-webapp-4&templateURL=https://s3.amazonaws.com/wildrydes-ap-northeast-1/WebApplication/4_RESTfulAPIs/backend-api.yaml)
US East (N. Virginia) | [![Launch Module 4 in us-east-1](http://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/images/cloudformation-launch-stack-button.png)](https://console.aws.amazon.com/cloudformation/home?region=us-east-1#/stacks/new?stackName=wildrydes-webapp-4&templateURL=https://s3.amazonaws.com/wildrydes-us-east-1/WebApplication/4_RESTfulAPIs/backend-api.yaml)
<details>
<summary><strong>CloudFormation Launch Instructions (expand for details)</strong></summary><p>

1. Click the **Launch Stack** link above for the region of your choice.

1. Click **Next** on the Select Template page.

1. Provide the name of your website bucket from module 1 for the  **Website Bucket Name** (e.g. `wildrydes-yourname`) and choose **Next**.

    **Note:** You must specify the same bucket name you used in the previous module. If you provide a bucket name that does not exist or that you do not have write access to, the CloudFormation stack will fail during creation.

1. Provide the ARN for the User Pool we created in module 2. You can find the User Pool ARN in the [Amazon Cognito console](https://console.aws.amazon.com/cognito/users/).

1. On the Options page, leave all the defaults and click **Next**.

1. On the Review page, check the box to acknowledge that CloudFormation will create IAM resources and click **Create**.
    ![Acknowledge IAM Screenshot](../images/cfn-ack-iam.png)

    This template uses a custom resource to update the `/js/config.js` file with the new API endpoint URL

1. Wait for the `wildrydes-webapp-4` stack to reach a status of `CREATE_COMPLETE`.

1. Verify the Wild Rydes home page is loading properly and try to request a ride.

</p></details>

## 実装手順

以下の各セクションでは、実装の概要と詳細なステップバイステップの手順を説明します

AWS管理コンソールに精通している場合や、段階的な説明に従わずに自身でサービスを探索したい場合は、概要は実装を完了するのに十分な内容を提供しています。

最新バージョンのChrome、Firefox、SafariのWebブラウザを使用している場合は、セクションを展開するまで、ステップバイステップの手順は表示されません。

### 1. 新しいREST APIを作成する

Amazon API Gatewayコンソールを使用して新しいAPIを作成します。

<details>
<summary><strong>ステップバイステップ手順 (詳細を展開)</strong></summary><p>

1. AWS マネージメントコンソールで **サービス** から ネットワーキング ＆ コンテンツ配信の下にある **API Gateway** を選択します。

1. **新しいAPI の作成**を選択します。

1. **新しい API** を選択し、**API 名** に `WildRydes` を選択します。

1. **エンドポイントタイプ** ドロップダウンで `エッジ最適化` を選択します。
    ***Note***: **エッジ最適化** はパブリックサービスがインターネットからアクセスされる場合に最適です。通常、**地域** のエンドポイントは、主に同じAWSリージョン内からアクセスされるAPIに使用されます。

1. **API の作成** を選択します。

    ![Create API screenshot](../images/create-api.png)

</p></details>


### 2. Cognito ユーザープールオーソライザーの作成

#### Background

Amazon API Gatewayは、Cognito User Poolsから返されたJWTトークンを使用してAPI呼び出しを認証できます。このステップでは、[モジュール2](../2_UserManagement)で作成したユーザープールを使用するようにAPIのオーソライザを設定します。

#### 詳細な手順

Amazon API Gatewayコンソールで、API用の新しいCognitoユーザープールオーソライザーを作成します。前のモジュールで作成したユーザープールの詳細で構成します。現在のWebサイトの`/signin.html`ページからログインした後に表示されるauthトークンをコピーして貼り付けることで、コンソールで設定をテストできます。

<details>
<summary><strong>ステップバイステップ手順 (詳細を展開)</strong></summary><p>

1. 新しく作成したAPIページで、 **オーソライザー**を選択します。

1. **新しいオーソライザーの作成** を選択します。

1. オーソライザーの**名前** に `WildRydes`を選択します。.

1. **タイプ** に **Cognito** を選択します。

1. **Cognito ユーザープール**のRegionドロップダウンメニューで、モジュール2でCognito ユーザープールを作成した地域を選択します（デフォルトでは、現在の地域が選択されています）。

1. **Cognito ユーザープール**の入力欄に `WildRydes` (あなたが作成したユーザープール名) 入力します。

1. **トークンのソース** に `Authorization` を入力します。

1. **作成** を選択します。

    ![Create user pool authorizer screenshot](../images/create-user-pool-authorizer.png)

#### オーソライザー設定を確認する

1. 新しいブラウザタブで あなたのWeb サイトドメイン下の `/ride.html` にアクセスします。

1. ログインページにリダイレクトされた場合は、最後のモジュールで作成したユーザーでサインインします。 `/ride.html`にリダイレクトされます。

1. `/ ride.html`の通知からauthトークンをコピーします。

1. オーソライザーを作成したブラウザのタブに戻ります。

1. オーソライザーのカードの下部にある**テスト**をクリックします。

1. authトークンをダイアログの**Authorization**フィールドに貼り付けます。

    ![Test Authorizer screenshot](../images/apigateway-test-authorizer.png)

1. **テスト**ボタンをクリックし、応答コードが200で、ユーザーのクレームが表示されていることを確認します。

</p></details>

### 3. 新しいリソースとメソッドを作成する

API内に`/ride`という新しいリソースを作成します。次に、そのリソースのPOSTメソッドを作成し、このモジュールの最初のステップで作成したRequestUnicorn関数に基づくLambdaプロキシ統合を使用するように設定します。

<details>
<summary><strong>ステップバイステップ手順 (詳細を展開)</strong></summary><p>

1. 左側のナビゲーションで、WildRydes APIの下にある**リソース**をクリックします

1. **アクション** ドロップダウンから **リソースの作成** を選択します。

1. **リソース名** に `ride` を入力します.

1. **リソースパス** に `ride` が入力されていることを確認します。

1. **API Gateway CORSを有効にする** を**チェック**します。

1. **リソースの作成**をクリックします。

    ![Create resource screenshot](../images/create-resource.png)

1. 新しく作成した `/ride`リソースを選択して、**アクション** ドロップダウンから**メソッドの作成** を選択します。

1. 新しく表示されたドロップダウンで`POST`を選択し、**チェックマーク**をクリックします。

    ![Create method screenshot](../images/create-method.png)

1. **統合タイプ** に **Lambda 関数** を選択します。

1. **Lambda プロキシ統合の使用** をチェックします。

1. **Lambda リージョン** にLambda 関数の配備リージョンを選択します。

1. **Lambda 関数** に前のモジュールで作成した関数の名前、 `RequestUnicorn`を入力します。

1. **保存**を選択します。関数が存在しないというエラーが表示された場合は、選択したリージョンが前のモジュールで使用したものと一致することを確認してください。

    ![API method integration screenshot](../images/api-integration-setup.png)

1. Amazon API Gatewayに関数を呼び出す権限を与えるように促されたら、**OK**を選択してください。

1. **メソッドリクエスト** カードを選択してください。

1. **認証** の隣の**鉛筆アイコン**をクリックします。

1. ドロップダウンリストから WildRydes Cognito ユーザープールオーソライザーを選択し、`チェックマーク`アイコンをクリックします。

    ![API authorizer configuration screenshot](../images/api-authorizer.png)

</p></details>

### 4. API のデプロイ

Amazon API Gatewayコンソールから、「アクション」→「APIのデプロイ」を選択します。新しいステージを作成するように求められます。このステージ名にはprodを使うことができます。

<details>
<summary><strong>ステップバイステップ手順 (詳細を展開)</strong></summary><p>

1. **アクション** ドロップダウンから**APIのデプロイ** を選択します。

1. **デプロイされるステージ** ドロップダウンリストから **[新しいステージ]** を選択します。

1. **ステージ名** に `prod`を入力します。

1. **デプロイ** を選択します。

1. **URLの呼び出し**に記載されているURLをメモします。これは次のセクションで使用します。

</p></details>

### 5. Update the Website Config

Webサイトデプロイメントの/js/config.jsファイルを更新して、作成したステージの呼び出しURLを書き換えます。起動URLは、Amazon API Gatewayコンソールのステージエディタページの上部から直接コピーし、サイト/js/config.jsファイルの_config.api.invokeUrlキーに貼り付ける必要があります。設定ファイルを更新する際には、以前のモジュールで行ったCognitoユーザープールの更新が含まれていることを確認してください。

<details>
<summary><strong>ステップバイステップ手順 (詳細を展開)</strong></summary><p>

モジュール2を手作業で完成した場合は、ローカルに保存した `config.js`ファイルを編集することができます。 AWS CloudFormationテンプレートを使用した場合は、S3バケットから `config.js`ファイルをダウンロードする必要があります。これを行うには、あなたのウェブサイトのベースURLの `/js/config.js`にアクセスし、**ファイル**を選択し、ブラウザから**ページを保存**を選択してください。

1. テキストエディタでconfig.jsファイルを開きます

1. config.jsファイルの**api**キーの下にある**invokeUrl**設定を更新してください。前のセクションで作成したURLを**Invoke URL**に設定します。

    完全な `config.js`ファイルの例を以下に示します。ファイル内の実際の値は異なることに注意してください。

    ```JavaScript
    window._config = {
        cognito: {
            userPoolId: 'us-west-2_uXboG5pAb', // e.g. us-east-2_uXboG5pAb
            userPoolClientId: '25ddkmj4v6hfsfvruhpfi7n4hv', // e.g. 25ddkmj4v6hfsfvruhpfi7n4hv
            region: 'us-west-2' // e.g. us-east-2
        },
        api: {
            invokeUrl: 'https://rc7nyt4tql.execute-api.us-west-2.amazonaws.com/prod' // e.g. https://rc7nyt4tql.execute-api.us-west-2.amazonaws.com/prod,
        }
    };
    ```

1. ファイルを保存します。

1. 変更したファイルをS3にコピーします。変更したファイルだけ転送されます。

    aws s3 sync aws-serverless-workshops/WebApplication/1_StaticWebHosting/website s3://YOUR_BUCKET_NAME --region YOUR_BUCKET_REGION

</p></details>

## 実装の検証

**Note:** S3バケットのconfig.jsファイルの更新と更新されたコンテンツがブラウザに表示されるまでの間に遅延が発生する可能性があります。また、次の手順を実行する前に、ブラウザのキャッシュをクリアする必要があります。

1. Webサイトドメイン下の`/ ride.html`をアクセスします。

1. サインインページにリダイレクトされた場合は、前のモジュールで作成したユーザーでサインインします。

1. 地図がロードされたら、地図上の任意の場所をクリックしてピックアップの場所を設定します。

1. **Request Unicorn**を選択してください。右側のサイドバーに、ユニコーンが途中にあるという通知が表示され、ユニコーンアイコンがピックアップの場所に飛んでくるのが見えるはずです。

おめでとう、あなたはWild Rydes Web Application Workshopを完了しました！サーバーレスのユースケースをさらにカバーする[その他のワークショップ](../../ README.md＃ワークショップ)をご覧ください。

作成したリソースを削除する方法については、このワークショップの[クリーンアップガイド](../9_CleanUp)を参照してください。
