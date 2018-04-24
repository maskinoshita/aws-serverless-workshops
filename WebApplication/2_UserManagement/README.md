# Module 2: User Authentication and Registration with Amazon Cognito User Pools

このモジュールでは、ユーザーのアカウントを管理するAmazon Cognitoのユーザープールを作成します。顧客が新しいユーザーとして登録し、電子メールアドレスを確認し、サイトにサインインできるようにするページをデプロイします。

このモジュールをスキップしたい場合は、必要なリソースを自動的に構築するために、CloudFormationテンプレートの1つを選択した地域で起動できます。

Region| Launch
------|-----
Asia Pacific (Tokyo) | [![Launch Module 2 in ap-northeast-1](http://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/images/cloudformation-launch-stack-button.png)](https://console.aws.amazon.com/cloudformation/home?region=ap-northeast-1#/stacks/new?stackName=wildrydes-webapp-2&templateURL=https://s3.amazonaws.com/wildrydes-ap-northeast-1/WebApplication/2_UserManagement/user-management.yaml)
US East (N. Virginia) | [![Launch Module 2 in us-east-1](http://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/images/cloudformation-launch-stack-button.png)](https://console.aws.amazon.com/cloudformation/home?region=us-east-1#/stacks/new?stackName=wildrydes-webapp-2&templateURL=https://s3.amazonaws.com/wildrydes-us-east-1/WebApplication/2_UserManagement/user-management.yaml)

<details>
<summary><strong>CloudFormation Launch Instructions (expand for details)</strong></summary><p>

1. 好みのリージョンの **Launch Stack** をクリックします。

1. テンプレートの選択ページで **次へ** をクリックします。

1. **Website Bucket Name** に モジュール１で入力した値(`wildrydes-{{あなたの名前}}` のような)を入力し、**次へ** をクリックします。

    **Note:** 前のモジュールで使用したものと同じバケット名を指定する必要があります。存在しないか、書き込みアクセス権を持たないバケット名を指定すると、CloudFormationスタックは作成中に失敗します。

    ![Speficy Details Screenshot](../images/module2-cfn-specify-details.png)

1. オプションページではすべてデフォルトのままで **次へ** をクリックします。

1. 確認ページでは "AWS CloudFormation によって IAM リソースが作成される場合があることを承認"のボックスを **チェック** し、 **作成** をクリックします。
    ![Acknowledge IAM Screenshot](../images/cfn-ack-iam.png)

    このテンプレートは、カスタムリソースを使用してAmazon Cognitoのユーザープールとクライアントを作成し、このユーザープールに接続してWebサイトのバケットにアップロードするのに必要な詳細を含む設定ファイルを生成します。テンプレートは、これらのリソースを作成し、設定ファイルをバケットにアップロードするためのアクセス権を提供するロールを作成します。

1. `wildrydes-webapp-2` スタックが `CREATE_COMPLETE` ステータスに変わるまで待ちます。

1.   次のモジュールに進む準備が整ったことを確認するには、[実装検証](#実装検証) セクションに記載されている手順に従ってください。

</p></details>

## アーキテクチャ概要

ユーザーがあなたのウェブサイトにアクセスすると、まず新しいユーザーアカウントを登録します。このワークショップでは、登録するために電子メールアドレスとパスワードを要求するだけです。ただし、Amazon Cognitoでは、独自のアプリケーションで追加の属性を要求するように設定できます。

ユーザーが登録を送信すると、Amazon Cognitoは、提供されたアドレスに確認コード付きの確認メールを送信します。アカウントを確認するために、ユーザーはサイトに戻り、自分のメールアドレスと受信した確認コードを入力します。テスト用に偽の電子メールアドレスを使用する場合は、Amazon Cognitoコンソールを使用してユーザーアカウントを確認することもできます。

ユーザーが確認済みのアカウントを持っている（電子メールの確認プロセスを使用するか、コンソールから手動で確認する）と、ユーザーはサインインできます。ユーザーがサインインする場合、ユーザー名（または電子メール）とパスワードが入力します。その後、JavaScript関数はAmazon Cognitoと通信し、Secure Remote Passwordプロトコル（SRP）を使用して認証し、一連のJSON Webトークン（JWT）を受信します。 JWTにはユーザーの身元に関する主張が含まれており、次のモジュールでAmazon API Gatewayで構築したRESTful APIに対して認証するために使用されます。

![Authentication architecture](../images/authentication-architecture.png)

## 実装手順

以下の各セクションでは、実装の概要と詳細なステップバイステップの手順を説明します。

AWS管理コンソールに精通している場合や、段階的な説明に従わずに自身でサービスを探索したい場合は、概要は実装を完了するのに十分な内容を提供しています。

最新バージョンのChrome、Firefox、SafariのWebブラウザを使用している場合は、セクションを展開するまで、ステップバイステップの手順は表示されません。

### 1. Amazon Cognito User Pool を作成する

#### Background

Amazon Cognitoは、ユーザーを認証するための2つの異なるメカニズムを提供しています。**Cognito User Pools**によりアプリケーションにサインアップおよびサインイン機能を追加するか、**Cognito Identity Pool**により、Facebook、Twitter、Amazonなどのソーシャル認証プロバイダ、SAML認証ソリューション、または独自の認証システムを使用してユーザを認証できます。このモジュールでは、**Cognito User Pool**を登録、サインインページのバックエンドとして使用します。

#### 詳細な手順

Amazon Cognitoコンソールを使用して、デフォルト設定を使用して新しいユーザープールを作成します。プールが作成されたら、プールIDをメモします。後のセクションでこの値を使用します。

<details>
<summary><strong>ステップバイステップ手順 (詳細を展開)</strong></summary
<p>

1. AWS マネージメントコンソールで **サービス** から セキュリティ、 アイデンティティ、 コンプライアンスの下にある **Cognito** を選択します。

1. **ユーザープールの管理** を選択します。

1. **ユーザープールを作成する** を選択します。

1. `WildRydes-USERNAME`のようなあなたのユーザープールの名前を入力し、**デフォルトを確認する**を選択してください。

    ![Create a user pool screenshot](../images/create-a-user-pool.png)

1. 確認ページで **プールの作成** をクリックします。

1. 新しく作成されたユーザープールのプールの詳細ページで、**プール ID** (`us-east-1_ygbFpYlRC`のような) をメモしておきます。

</p></details>

### 2. アプリケーションクライアントをユーザープールに追加する

Amazon Cognitoコンソールで、ユーザープールを選択してから、**アプリクライアント**セクションを選択します。新しいアプリを追加して、[クライアントシークレットの生成]オプションがオフになっていることを確認します。クライアントシークレットは、JavaScript SDKではサポートされていません。もしアプリケーションをクライアントシークレット付きで作成してしまった場合は、削除して正しい設定で新しいものを作成してください。

<details>
<summary><strong>ステップバイステップ手順 (詳細を展開)</strong></summary><p>

1. ユーザープールのプールの詳細ページの左のナビゲーションから **アプリクライアント** を選択します。

1. **アプリクライアントの追加** を選択します。

1. `WildRydesWebApp-USERNAME` のようなアプリクライアント名を設定します。

1. クライアントシークレットの生成オプションのチェックを**外します**。クライアントシークレットは、ブラウザベースのアプリケーションではサポートされていません。

1. **アプリクライアントの作成** を選択します。

   <kbd>![Create app client screenshot](../images/add-app.png)</kbd>

1. 新しく作成されたアプリクライアントの**App client id**(`40f3an21v98dj996sflhia83jv`のような)をメモしておきます。

</p></details>

### 3. Webサイトのバケット内の`config.js`を更新する

 [/js/config.js](../1_StaticWebHosting/website/js/config.js) ファイルには、ユーザープールID、アプリケーションクライアントID、地域の設定が含まれています。前の手順で作成したユーザープールとアプリケーションの設定でこのファイルをCloud9で更新し、ファイルをバケットにアップロードし直してください(CLIで実行した`aws sync ...`を再度実行すればOK)。

<details>
<summary><strong>ステップバイステップ手順 (詳細を展開)</strong></summary><p>

1. Webサイトディレクトリから [config.js](../1_StaticWebHosting/website/js/config.js) を開きます。

1. `cognito`セクションを、作成したユーザープールとアプリケーションの正しい値で更新してください。

    作成したユーザープールを選択後、Amazon Cognitoコンソールのプール詳細ページから`userPoolId`を探すことができます。

     ![Pool ID](../images/pool-id.png)

    左のナビゲーションバー中の**アプリクライアント** を選択すると、`userPoolClientId`を探すことができます。

    ![Pool ID](../images/client-id.png)

    `region`の値は、あなたのユーザープールを作成したAWS Regionコードでなければなりません。例えば。バージニア州の場合は「us-east-1」、オレゴン州の場合は「us-west-2」です。使用するコードが不明な場合は、プールの詳細ページでプールのARN値を確認できます。リージョンコードはARNの `arn：aws：cognito-idp：`のすぐ後の部分です。

    更新されたconfig.jsファイルは次のようになります。ファイルの実際の値は異なることに注意してください。
    ```JavaScript
    window._config = {
        cognito: {
            userPoolId: 'us-west-2_uXboG5pAb', // e.g. us-east-2_uXboG5pAb
            userPoolClientId: '25ddkmj4v6hfsfvruhpfi7n4hv', // e.g. 25ddkmj4v6hfsfvruhpfi7n4hv
            region: 'us-west-2' // e.g. us-east-2
        },
        api: {
            invokeUrl: '' // e.g. https://rc7nyt4tql.execute-api.us-west-2.amazonaws.com/prod',
        }
    };
    ```

1. 変更した`config.js`を保存します。

1. 変更したファイルをS3にコピーします。変更したファイルだけ転送されます。

    aws s3 sync aws-serverless-workshops/WebApplication/1_StaticWebHosting/website s3://YOUR_BUCKET_NAME --region YOUR_BUCKET_REGION

</p></details>

<p>

**Note:** 登録の管理、検証、サインインのフローをブラウザ側のコードで記述するのではなく、モジュール中のアセットに動作する実装を提供しています。 [cognito-auth.js](../1_StaticWebHosting/website/js/cognito-auth.js)ファイルには、UIイベントを処理し、適切なAmazon Cognito Identity SDKメソッドを呼び出すコードが含まれています。 SDKの詳細については、[GitHubのプロジェクトページ]（https://github.com/aws/amazon-cognito-identity-js）を参照してください。

</p>

## 実装検証

1. ウェブサイトのドメインの下で `/register.html`にアクセスするか、サイトの** Giddy Up！**ボタンを選択してください。

1. 登録フォームに記入し、**Let's Ryde**を選択します。あなた自身の電子メールを使用するか、偽の電子メールを入力することができます。**少なくとも1つの大文字、数字、および特殊文字を含むパスワード**を選択してください。後で入力したパスワードを忘れないでください。ユーザーが作成されたことを確認するアラートが表示されます。

1. 次の2つの方法のいずれかを使用して、新しいユーザーを確認します。

  1. 管理しているメールアドレスを使用している場合は、ウェブサイトドメインの下にある`/verify.html`にアクセスし、メールで送信された確認コードを入力してアカウント確認プロセスを完了できます。確認メールはスパムフォルダに保存される可能性があります。 実際のアプリケーションでは [SMS と E メール確認メッセージとユーザー招待メッセージのカスタマイズ](https://docs.aws.amazon.com/ja_jp/cognito/latest/developerguide/cognito-user-pool-settings-message-customizations.html)を使用して、所有するドメインからメールを送信します。

  1. ダミーのメールアドレスを使用した場合は、Cognitoコンソールから手動で確認する必要があります。

    1. AWS マネージメントコンソールで **サービス** から セキュリティ、 アイデンティティ、 コンプライアンスの下にある **Cognito** を選択します
    1. **ユーザープールの管理** を選択します。
    1. `WildRydes-USERNAME` ユーザープールを選択し、左のナビゲーションバーから**ユーザーとグループ** を選択します。
    1. 登録ページから送信したメールアドレスに対応するユーザーが表示されます。ユーザー詳細ページを表示するユーザー名を選択します。
    1. **ユーザの確認** を押し、ユーザ作成プロセスを完了します。

1. `/verify.html`ページまたはCognitoコンソールを使用して新しいユーザーを確認した後、`/signin.html`にアクセスして、登録ステップで入力した電子メールアドレスとパスワードを使用してログインします。

1. 成功した場合は`/ride.html`にリダイレクトされます。APIが設定されていないという通知が表示されます。

    ![Successful login screenshot](../images/successful-login.png)

Webアプリケーションに正常にログインしたら、次のモジュールに進むことができます。 [Serverless Backend](../3_ServerlessBackend).

### Extra

* あなたが受け取った**auth_token**をコピーして、[online JWT Decoder](https://jwt.io/)に貼り付けてみてください。このトークンがアプリケーションにとってどんな意味を持つのかの手助けになります。

