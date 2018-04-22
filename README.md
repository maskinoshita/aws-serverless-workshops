# Wild Rydes Serverless Workshops

※ このリポジトリは、AWSが提供するWild Rydes Serverless Workshopを独自に改変したものになります。
オリジナルは５つのワークショップが用意されていますが、このリポジトリはWeb Applicationに限定して和訳、改変したものになります。オリジナルについては下記を参照ください。[aws-serverless-workhops](https://github.com/aws-samples/aws-serverless-workshops)

このリポジトリには、AWSラムダ、Amazon APIゲートウェイ、Amazon DynamoDB、AWSステップ関数、Amazon Kinesisなどのサービスを使用して、さまざまなサーバーレスアプリケーションを構築するためのワークショップやその他のコンテンツが含まれています。

# Workshops

- [**Web Application**](WebApplication) - このワークショップでは、動的でサーバレスなWebアプリケーションを構築する方法を説明します。 Amazon S3で静的なWebリソースをホストする方法、Amazon Cognitoを使用してユーザーと認証を管理する方法、Amazon API Gateway、AWS Lambda、Amazon DynamoDBを使用してバックエンド処理用のRESTful APIを構築する方法を学びます。

- [**Data Processing**](https://github.com/aws-samples/aws-serverless-workshops/tree/master/DataProcessing) - This workshop demonstrates how to collect, store, and process data with a serverless application. In this workshop you'll learn how to automatically process files on Amazon S3 using AWS Lambda, how to build real-time streaming applications using Amazon Kinesis Streams and Amazon Kinesis Analytics, how to archive data streams using Amazon Kinesis Firehose and Amazon S3, and how to run ad-hoc queries on those files using Amazon Athena.

- [**DevOps**](https://github.com/aws-samples/aws-serverless-workshops/tree/master/DevOps) - This workshop shows you how to use the [Serverless Application Model (SAM)](https://github.com/awslabs/serverless-application-model) to build a serverless application using Amazon API Gateway, AWS Lambda, and Amazon DynamoDB. You'll learn how to use SAM from your workstation to release updates to your application, how to build a CI/CD pipeline for your serverless application using AWS CodePipeline and AWS CodeBuild, and how to enhance your pipeline to manage multiple environments for your application.

- [**Image Processing**](https://github.com/aws-samples/aws-serverless-workshops/tree/master/ImageProcessing) - This module shows you how to build a serverless image processing application using workflow orchestration in the backend. You'll learn the basics of using AWS Step Functions to orchestrate multiple AWS Lambda functions while leveraging the deep learning-based facial recognition features of Amazon Rekogntion.

- [**Multi Region**](https://github.com/aws-samples/aws-serverless-workshops/tree/master/MultiRegion) - This workshop shows you how to build a serverless ticketing system that is replicated across two regions and provides automatic failover in the event of a disaster. You will learn the basics of deploying AWS Lambda functions, exposing them via API Gateway, and configuring replication using Route53 and DynamoDB streams.
