---
title: sam コマンドが Docker を見つけられない
tags:
  - sam
private: false
updated_at: '2025-10-13T11:49:18+09:00'
id: a7f7aefd6d7fa7a8f5d1
organization_url_name: null
slide: false
ignorePublish: false
---
## 前提

- Docker は起動済み
- MacPC

## 事象

`sam local start-api` を実行したら `Error: Running AWS SAM projects locally requires Docker. Have you got it installed and running` が出た。

```sh
$ sam local start-api
Error: Running AWS SAM projects locally requires Docker. Have you got it installed and running
```

## 原因

Docker は起動しているにも関わらず上記のエラーが出るのは、 SAM CLI が Docker を認識できていないため。

`docker context ls` コマンドで現在の Docker コンテキストを確認できる。コマンドの実行結果から現在は orbstack が選択されていることが分かる。

```sh
$ docker context ls
NAME         DESCRIPTION                               DOCKER ENDPOINT                                        ERROR
default      Current DOCKER_HOST based configuration   unix:///var/run/docker.sock
orbstack *   OrbStack                                  unix:///Users/takiguchi-yu/.orbstack/run/docker.sock
```


## 対処

`SAM CLI` ではどうやら default の `DOCKER ENDPOINT` を参照するようなので 

- 方法①： 環境変数 `DOCKER_HOST` のパスに orbstack のエンドポイントを設定するか
- 方法②： `/var/run/docker.sock` ファイルを作成し `/Users/takiguchi-yu/.orbstack/run/docker.sock` にシンボリックリンクを作成する

の 2 パターンの回避策がある。


以下はシンボリックリンクで対処する方法である。

```sh
sudo ln -s ~/.orbstack/run/docker.sock /var/run/docker.sock
```
