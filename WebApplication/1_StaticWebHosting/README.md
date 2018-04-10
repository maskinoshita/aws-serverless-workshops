# Module 1: Static Web Hosting with Amazon S3

このモジュールでは Web アプリケーションの静的リソースをホストするために Amazon Simple Storage Service (S3) を設定します. 以後のモジュールでは、JavaScriptを使用してこれらのページに動的機能を追加し、AWS LambdaとAmazon API Gatewayで構築されたRESTful APIを呼び出します。

既にAmazon S3での作業に慣れている場合、またはLambdaとAPI Gatewayの作業を省略したい場合は、必要なリソースを自動的に構築するために、AWS CloudFormationテンプレートの1つを起動できます。

Region| Launch
------|-----
Asia Pacific (Tokyo) | [![Launch Module 1 in ap-northeast-1](http://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/images/cloudformation-launch-stack-button.png)](https://console.aws.amazon.com/cloudformation/home?region=ap-northeast-1#/stacks/new?stackName=wildrydes-webapp-1&templateURL=https://s3.amazonaws.com/wildrydes-ap-northeast-1/WebApplication/1_StaticWebHosting/webapp-static-hosting.yaml)
US East (N. Virginia) | [![Launch Module 1 in us-east-1](http://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/images/cloudformation-launch-stack-button.png)](https://console.aws.amazon.com/cloudformation/home?region=us-east-1#/stacks/new?stackName=wildrydes-webapp-1&templateURL=https://s3.amazonaws.com/wildrydes-us-east-1/WebApplication/1_StaticWebHosting/webapp-static-hosting.yaml)

<details>
<summary><strong>CloudFormation 起動手順 (詳細を展開)</strong></summary><p>

1. 好みのリージョンの **Launch Stack** をクリックします。

1. テンプレートの選択ページで **次へ** をクリックします。

1. **Website Bucket Name** に `wildrydes-{{あなたの名前}}` のような世界で唯一の値を入力し、**次へ** をクリックします。
    ![Speficy Details Screenshot](../images/module1-cfn-specify-details.png)

1. オプションページではすべてデフォルトのままで **次へ** をクリックします。

1. 確認ページでは "AWS CloudFormation によって IAM リソースが作成される場合があることを承認"のボックスを **チェック** し、 **作成** をクリックします。

    ![Acknowledge IAM Screenshot](../images/cfn-ack-iam.png)

    このテンプレートでは、カスタムリソースを使用して静的なWebサイト資産を中央のS3バケットから独自の専用バケットにコピーします。カスタムリソースがアカウントの新しいバケットに書き込むためには、これらのアクセス許可を引き受けられるIAMロールを作成する必要があります。

1. `wildrydes-webapp-1` スタックが `CREATE_COMPLETE` ステータスに変わるまで待ちます。

1. `wildrydes-webapp-1` を選択し, **出力** タブをクリックして WebsiteURL のリンクをクリックします。

1. Wild Rydes のホームページが正しく動作することを確認し、次のモジュールへ移動してください。 [User Management](../2_UserManagement).

</p></details>


## アーキテクチャ概要

このモジュールのアーキテクチャは非常に簡単です。 HTML、CSS、JavaScript、画像、その他のファイルを含む静的WebコンテンツはすべてAmazon S3に保存されます。エンドユーザーは、Amazon S3に公開されている公開WebサイトのURLを使用してサイトにアクセスします。サイトを利用可能にするために、Webサーバーを実行したり、他のサービスを使用する必要はありません。

![Static website architecture](../images/static-website-architecture.png)

このモジュールでは、Amazon S3 WebサイトのエンドポイントURLを使用します。地域によっては、`http：// {your-bucket-name} .s3-website- {region} .amazonaws.com`または `bucket-name.s3-website.region.amazonaws.com`という形式があります。多くの実際のアプリケーションでは、カスタムドメインを使用してサイトをホストしたいと思うでしょう。 独自のドメインを使用することに興味がある場合は Amazon S3 ドキュメント内の [独自ドメインを使用して静的ウェブサイトをセットアップする](https://docs.aws.amazon.com/ja_jp/AmazonS3/latest/dev/website-hosting-custom-domain-walkthrough.html) をご参照ください。

## 実装手順

以下の各セクションでは、実装の概要と詳細なステップバイステップの手順を説明します

AWS管理コンソールに精通している場合や、段階的な説明に従わずに自身でサービスを探索したい場合は、概要は実装を完了するのに十分な内容を提供しています。

最新バージョンのChrome、Firefox、SafariのWebブラウザを使用している場合は、セクションを展開するまで、ステップバイステップの手順は表示されません。

### リージョンの選択

このワークショップでは次のサービスをサポートする任意のAWSリージョンにデプロイできます。

- Amazon Cognito
- AWS Lambda
- Amazon API Gateway
- Amazon S3
- Amazon DynamoDB

AWSドキュメントの[製品およびサービス一覧](https://aws.amazon.com/about-aws/global-infrastructure/regional-product-services/)を参照して、サービスをサポートしているリージョンを確認することができます。

リージョンを選択したら、そこにこのワークショップのすべてのリソースをデプロイする必要があります。開始する前にAWS コンソールの右上隅にあるドロップダウンからリージョンを選択してください。

![Region selection screenshot](../images/region-selection.png)

### 1. S3 バケットを作成する

Amazon S3 は Web サーバーを構成または管理することなく、静的 Web サイトをホストするために使用できます。この手順では、Web アプリケーション用のすべての静的アセット（HTML、CSS、JavaScript、画像ファイルなど）をホストするために使用する新しい S3 バケットを作成します。

#### 詳細な手順

コンソールまたはAWS CLIを使用してAmazon S3バケットを作成します。バケットの名前はすべての地域、顧客で全世界で唯一でなければならないことに注意してください。`wildrydes-firstname-lastname`のような名前を使用することをお勧めします。バケット名がすでに存在するというエラーが表示された場合は、未使用の名前が見つかるまで数字または文字を追加してみてください。

<details>
<summary><strong>ステップバイステップ手順 (詳細を展開)</strong></summary><p>

1. AWS マネージメントコンソールで **サービス** から ストレージの下にある **S3** を選択します。

1. **+バケットを作成する** を選択します。

1. `wildrydes-firstname-lastname`のような全世界で唯一の名前をバケット名に指定します。

1. このワークショップで利用するリージョンを選択します。

1. "既存のバケットから設定をコピー" を **選択せず**、ダイアログの左下にある **作成** を押します。

    ![Create bucket screenshot](../images/create-bucket.png)

</p></details>

### 2. コンテンツのアップロード

このモジュールの静的アセットを S3 バケットにアップロードします。この手順には、AWS マネージメントコンソール (Google Chrome ブラウザが必要) 、AWS CLI、または提供されている Cloud Formation テンプレートを使用できます。すでにローカルマシンにAWS CLIをインストールして設定している場合は、その方法を使用することをお勧めします。それ以外の場合は、最新バージョンのGoogle Chromeを利用してコンソールを使用します。

<details>
<summary><strong>CLI ステップバイステップ手順(詳細を展開)</strong></summary><p>

すでにCLIをインストール設定している場合は、それを使用して`s3://wildrydes-us-east-1/WebApplication/1_StaticWebHosting/website`から必要な Web アセットをバケットにコピーできます。

前のセクションで使用したバケット名で `YOUR_BUCKET_NAME`を置き換え、バケットを作成したリージョンコード（例えばap-northeast-1）で` YOUR_BUKET_REGION`を置き換えて次のコマンドを実行します。

    aws s3 sync s3://wildrydes-us-east-1/WebApplication/1_StaticWebHosting/website s3://YOUR_BUCKET_NAME --region YOUR_BUCKET_REGION

コマンドが成功した場合は、バケットにコピーされたオブジェクトのリストが表示されます。
</p></details>

<details>
<summary><strong>CloudFormation ステップバイステップ手順 (詳細を展開)</strong></summary><p>

上記のいずれの方法も使用できない場合は、必要なアセットをS3バケットにコピーするために、提供された CloudFormation テンプレートを起動することができます。

Region| Launch
------|-----
Asia Pacific (Tokyo) | [![Launch Module 1 in ap-northeast-1](http://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/images/cloudformation-launch-stack-button.png)](https://console.aws.amazon.com/cloudformation/home?region=ap-northeast-1#/stacks/new?stackName=wildrydes-copy-objects&templateURL=https://s3.amazonaws.com/wildrydes-ap-northeast-1/WebApplication/1_StaticWebHosting/webapp-copy-objects.yaml)
US East (N. Virginia) | [![Launch Module 1 in us-east-1](http://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/images/cloudformation-launch-stack-button.png)](https://console.aws.amazon.com/cloudformation/home?region=us-east-1#/stacks/new?stackName=wildrydes-copy-objects&templateURL=https://s3.amazonaws.com/wildrydes-us-east-1/WebApplication/1_StaticWebHosting/webapp-copy-objects.yaml)

1. Webサイト用のバケットを作成したリージョンの **Launch Stack** をクリックします。

1. テンプレートの選択ページで **次へ** をクリックします。

1. **Website Bucket Name** に 自分の S3 バケット名 (例えば、`wildrydes-yourname`) を入力し、**次へ** をクリックします。

1. オプションページではすべてデフォルトのままで **次へ** をクリックします。

1. 確認ページでは "AWS CloudFormation によって IAM リソースが作成される場合があることを承認"のボックスを **チェック** し、 **作成** をクリックします。

    ![Acknowledge IAM Screenshot](../images/cfn-ack-iam.png)

    このテンプレートでは、カスタムリソースを使用して静的なWebサイト資産を中央のS3バケットから独自の専用バケットにコピーします。カスタムリソースがアカウントの新しいバケットに書き込むためには、これらのアクセス許可を引き受けられるIAMロールを作成する必要があります。

1. `wildrydes-copy-objects` スタックが `CREATE_COMPLETE` ステータスに変わるのを待ちます.

</p></details>

### 3. パブリック読込を許可するバケットポリシーを追加する

バケットポリシーを使用して、S3 バケット内のコンテンツにアクセスできるユーザを定義することができます。バケットポリシーは、バケット内のオブジェクトに対してアクションを実行できるプリンシパル（ユーザ名等）を指定するJSONドキュメントです。

#### 詳細な手順

新しいAmazon S3バケットにバケットポリシーを追加して、匿名ユーザーにあなたのサイトを表示させる必要があります。デフォルトでは、AWSアカウントへのアクセス権を持つ認証済みのユーザーのみがバケットにアクセスできます。

匿名ユーザーへの読み取り専用アクセスを許可する [この例](https://docs.aws.amazon.com/ja_jp/AmazonS3/latest/dev/example-bucket-policies.html#example-bucket-policies-use-case-2) を参照してください。この例のポリシーでは、インターネット上の誰でもコンテンツを表示できます。バケットポリシーを更新する最も簡単な方法は、コンソールを使用することです。バケットを選択し、**アクセス権限** タブを選択して、**バケットポリシー** を選択します。

(訳注: **アクセス権限**タブの**アクセスコントロールリスト**タブのパブリックアクセスからも指定できます)

<details>
<summary><strong>ステップバイステップ手順 (詳細を展開)</strong></summary><p>

1. S3 コンソールで、 セクション1で自身が作成したバケットを選択します。

1. **アクセス権限** タブから **Bucket Policy** を選択します。.

1. 次のポリシードキュメントをバケットポリシーエディタに入力して、`YOUR_BUCKET_NAME` をセクション1で作成したバケットの名前に置き換えます。

    ```json
    {
        "Version": "2012-10-17",
        "Statement": [
            {
                "Effect": "Allow",
                "Principal": "*",
                "Action": "s3:GetObject",
                "Resource": "arn:aws:s3:::YOUR_BUCKET_NAME/*"
            }
        ]
    }
    ```

    ![Update bucket policy screenshot](../images/update-bucket-policy.png)

1. **保存** を選択します。

</p></details>

### 4. Web サイトホスティングの有効化

デフォルトでは、S3バケットのオブジェクトは、`http://<Regional-S3-prefix>.amazonaws.com/<bucket-name>/<object-key>`の構造を持つURLとして利用できます。アセットをルートURL (例えば /index.html) として配信するには、バケットの Web サイトホスティングを有効にする必要があります。これにより、バケットのAWSリージョン固有のWeb サイトエンドポイントでオブジェクトを利用できるようになります:
`<bucket-name>.s3-website-<AWS-region>.amazonaws.com`

あなたのウェブサイトにカスタムドメインを使用することもできます。たとえば、http：//www.wildrydes.com がS3でホストされています。このワークショップでは、カスタムドメインの設定については説明しませんが、詳細な手順についてはこちらをご覧ください。 [documentation](https://docs.aws.amazon.com/ja_jp/AmazonS3/latest/dev/website-hosting-custom-domain-walkthrough.html).

#### 詳細な手順

コンソールを使用して、静的 web サイトのホスティングを有効にします。この操作は、バケットを選択した後で [プロパティ] タブで行うことができます。**インデックスドキュメント** として **'index .html'** を設定し、**エラードキュメント** を **空白** のままにします。 詳細は [ウェブサイトホスティング用のバケットの設定](https://docs.aws.amazon.com/ja_jp/AmazonS3/latest/dev/HowDoIWebsiteConfiguration.html) をご参照ください。

<details>
<summary><strong>ステップバイステップ手順 (詳細を展開)</strong></summary><p>

1. S3 コンソールのバケット詳細ページから **プロパティ** タブを選択します。

1. **Static website hosting** カードを選択します。

1. **このバケットを使用してウェブサイトをホストする** を選択し、 **インデックスドキュメント**に`index.html`を残りのフィールドは空白にしておきます。

1. ダイアログの上部にある **エンドポイント** URL を **保存** をクリックする前にメモしておきます。ワークショップではこのURLをWebアプリケーションを表示するために使用します。以後はこのURLをあなたの Web サイトのベース URLを呼びます。

1. **保存** をクリックして変更を保存します。

    ![Enable website hosting screenshot](../images/enable-website-hosting.png)

</p></details>


## 実装の確認

実装手順が完了したら、S3バケットのWebサイトのエンドポイントURLを参照して、静的なWebサイトにアクセスできるようにする必要があります。

あなたの Web サイトのベース URL（セクション4のURLです）に、Web ブラウザで訪れてください。 Wild Rydes のホームページが表示されます。ベースURLを検索する必要がある場合は、S3コンソールにアクセスしてバケットを選択し、**[プロパティ]**タブの**Static website hosting**カードをクリックします。

ページが正しく表示されたら (下のスクリーンショット), 次のモジュールに移動してください。 [User Management](../2_UserManagement).

![Wild Rydes homepage screenshot](../images/wildrydes-homepage.png)
