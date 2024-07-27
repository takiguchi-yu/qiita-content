---
title: macOS を Ventura にアップデートしたら ssh でサーバに入れなくなった
tags:
  - macOS
  - ventura
private: false
updated_at: '2024-07-27T12:54:50+09:00'
id: f39f6abbeb4cda3ef4c2
organization_url_name: null
slide: false
ignorePublish: false
---

### 事象

macOS を Monterey 12 から Ventura 13 にしたら、 ssh でサーバに入れなくなった。

### 原因

MacOS Ventura に同梱されているバージョンの OpenSSH がデフォルトで RSA 署名を無効にしているため。

### 対処方法

`.ssh/config` に以下の設定を追加したら解消した。

```conf
Host *
  : (省略)
  HostkeyAlgorithms +ssh-rsa
  PubkeyAcceptedAlgorithms +ssh-rsa
```

人によっては以下の設定も必要だったりする。（Monterey でセキュリティ強化されたため。）

```
  KexAlgorithms +diffie-hellman-group-exchange-sha1,diffie-hellman-group14-sha1,diffie-hellman-group1-sha1
```

### 参考ページ

https://osxdaily.com/2022/12/22/fix-ssh-not-working-macos-rsa-issue/
