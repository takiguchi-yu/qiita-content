---
title: AWS SAM CLI を使ってローカル環境でホットリロード開発する方法
tags:
  - sam
  - ローカル開発環境
private: false
updated_at: '2023-10-01T00:15:17+09:00'
id: da2d6ab6ad3da922ffd6
organization_url_name: null
slide: false
ignorePublish: false
---
AWS SAM CLI を使ってローカルだけで開発したいことがある。ただ、これをやろうとするとソースコードを変更する度に `sam build` を実行する必要があるため、面倒臭い。

## ローカル環境の構成

TypeScript 前提。

`nodemon` でソースコードを監視することで自動で sam build を動かす。これによりホットリロード開発を実現する。

![localenv.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/59081/bc874d1a-c1f5-fdf5-ce45-8966f6327421.png)

## 手順

以下の手順を実施する。

#### 1. ターミナルを２つ開く

#### 2. １つ目のターミナルで監視を起動

[nodemon](https://github.com/remy/nodemon) がローカル PC に入っていなければインストールする。

<details><summary>nodemonインストール方法はこちら...</summary>

```sh
npm install -g nodemon
```
</details>


以下のコマンドでソースコードを監視し変更があれば `sam build` を自動で実行する。

```sh
nodemon --ext ts --watch src --exec 'sam build'
```
- **--ext**: 監視する拡張子を指定
- **--watch**: 監視するディレクトリを指定
- **--exec**: 変更を検知したときに実行するコマンドを指定

#### 3. ２つ目のターミナルで API Gateway を起動

```sh
sam local start-api
```

これでホットリロード開発ができるようになる。
