# Serverless Web Application Workshop

このワークショップでは、ユーザーが[Wild Rydes](http://www.wildrydes.com/)のユニコーンライドをリクエストできるシンプルなWebアプリケーションをデプロイします。いわゆる配車(馬？)アプリケーションです。

このアプリケーションは、ユーザがピックアップ場所を指示するためのHTMLベースのユーザインタフェースをもち、RESTfulウェブサービスでリクエスト処理と近くのユニコーンの配送を行います。またアプリケーションは、乗車を要求する前に、ユーザーがサービスに登録してログインするための機能を提供します。

このアプリケーションのアーキテクチャは [AWS Lambda](https://aws.amazon.com/lambda/), [Amazon API Gateway](https://aws.amazon.com/api-gateway/), [Amazon S3](https://aws.amazon.com/s3/), [Amazon DynamoDB](https://aws.amazon.com/dynamodb/), and [Amazon Cognito](https://aws.amazon.com/cognito/) を使用します。

S3は、HTML、CSS、JavaScript、およびユーザーのブラウザに読み込まれたイメージファイルを含む静的なWebリソースをホストします。ブラウザで実行されるJavaScriptは、LambdaとAPI Gatewayを使用して構築されたパブリックバックエンドAPIからデータを送受信します。 Amazon Cognitoは、バックエンドAPIを保護するためのユーザー管理機能と認証機能を提供します。最後に、DynamoDBは、APIのLambda関数によってデータを格納できる永続化レイヤーを提供します。

完全なアーキテクチャの図については、下の図を参照してください。

![Wild Rydes Web Application Architecture](images/wildrydes-complete-architecture.png)

さっそく開始するには、[Static Web hosting](1_StaticWebHosting)モジュールページを参照してワークショップを開始してください。

## 前提条件

### AWS アカウント

このワークショップを完了するには、AWS IAM、S3、DynamoDB、Lambda、API Gateway、およびCognitoリソースを作成するためのアクセス権を持つAWSアカウントが必要です。このワークショップのコードと手順では、一度に1人のAWSアカウントしか使用していないと想定しています。別の生徒とアカウントを共有しようとすると、特定のリソースの名前の競合が発生します。競合によって作成に失敗したリソースに一意のサフィックスを追加することで、これらの問題を回避することができますが、指示にはこの作業を行うために必要な変更の詳細は記載されていません。

このワークショップで使用するすべてのリソースは、アカウントが12か月未満の場合はAWSフリー層の対象となります。詳細については、[AWS Free Tierページ]（https://aws.amazon.com/free/）をご覧ください。

### ブラウザ

このワークショップを完了するには、Chromeの最新バージョンを使用することをおすすめします。

### テキストエディタ

構成ファイルの更新を行うには、テキストエディタが必要です。

## モジュール

このワークショップは複数のモジュールに分かれています。次の手順に進む前に各モジュールを完了する必要がありますが、モジュール1と2にはAWS CloudFormationテンプレートが用意されていますので、必要なリソースを手作業で作成しなくても起動できます。

0. [First Setup](0_FirstSetup)

1. [Static Web hosting](1_StaticWebHosting)
2. [User Management](2_UserManagement)
3. [Serverless Backend](3_ServerlessBackend)
4. [RESTful APIs](4_RESTfulAPIs)

ワークショップを完了したら、[クリーンアップガイド](9_CleanUp)に従って作成されたすべてのリソースを削除できます。
