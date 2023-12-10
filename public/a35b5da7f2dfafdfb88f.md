---
title: AWS Chatbot が Amazon EventBridge に対応したので Amazon RDS イベントを通知させてみる
tags:
  - AWS
  - RDS
  - chatbot
  - EventBridge
private: false
updated_at: '2023-12-10T19:42:27+09:00'
id: a35b5da7f2dfafdfb88f
organization_url_name: null
slide: false
ignorePublish: false
---

# はじめに

『[こちらの記事](https://qiita.com/takiguchi-yu/items/2b2af40f7ef498a7f4e3)』の延長で、 Amazon RDS のイベント(起動や停止など)を `EventBridge` + `Chatbot` 経由 で Slack 通知するようにしました。

![chatboteventbridge.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/59081/725c7c0a-f624-bf00-2e59-c2780ef9d334.png)

今までは、Chatbot が EventBridge に対応してなかったみたいですね。

https://aws.amazon.com/jp/about-aws/whats-new/2021/04/aws-chatbot-now-expands-coverage-of-aws-services-monitored-through-amazon-eventbridge/

# やること

- SNS トピックの作成
- Chatbot の作成
- EventBridge ルールの作成

# SNS トピックの作成

まずは `SNS トピック` を作成します。

<img src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/59081/1dc65cd1-e007-2df0-1c78-0826b1c29013.jpeg" width="600px">

# Chatbot の作成

続いて `Chatbot` に先ほど作成した `SNS トピック` を設定します。

<img src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/59081/f179ce7a-5436-b5ba-af28-db327b4b6a4e.jpeg" width="600px">

`Chatbot` を設定すると自動で `SNS サブスクリプション` が作成されます。

<img src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/59081/2d13555a-aecf-a916-7983-bed1180c5674.jpeg" width="600px">

# EventBridge ルールの作成

今回は `RDS` のイベントを通知したいので、RDS をぽちぽちと選択します。

<img src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/59081/e3cec81f-aeaa-113b-e4bf-4e956af2451f.jpeg" width="600px">

# 通知デモ

試しに RDS のインスタンスを落としてみます。うまく通知がきましたね。

<img src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/59081/5309ef72-a519-e216-ebda-5e86377022d5.jpeg" width="600px">

# さいごに

これからは `AWS Chatbot`を使えば、アプリケーションのオートスケーリングや CI/CD ワークフローの監視など、運用アクティビティをより効果的に管理できそうですね！
