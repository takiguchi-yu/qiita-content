---
title: CodeBuild で Secrets Manager 環境変数を使用する際の TLS 証明書検証エラー
tags:
  - AWS
  - CodeBuild
private: false
updated_at: '2026-01-22T23:36:27+09:00'
id: 6c170a47fe576b44cd70
organization_url_name: null
slide: false
ignorePublish: false
---

## 事象

CodeBuild で Secrets Manager の環境変数を使用しようとした際、`DOWNLOAD_SOURCE` フェーズで以下のエラーが発生

```
Secrets Manager Error: RequestError: send request failed
caused by: Post "https://secretsmanager.ap-northeast-1.amazonaws.com/":
tls: failed to verify certificate: x509: certificate signed by unknown authority
```

![wedding-e2e-testing-bat_1c80788d-eb2f-481b-ad7c-06e917b45510___CodeBuild___ap-northeast-1.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/59081/155b88a2-7525-4984-ac23-ba6acfb59957.png)

## 原因

### 1. Docker イメージの CA 証明書不足

**根本原因**: 使用していた `node:24.13.0-trixie-slim` イメージには、**完全な CA 証明書バンドルが含まれていなかった**ため、VPC エンドポイント経由での TLS 証明書検証に失敗していた。

#### CA 証明書とは

- **CA (Certificate Authority) 証明書**: HTTPS 通信でサーバーの正当性を検証するための「信頼できる証明書発行機関のリスト」
- CodeBuild が Secrets Manager に HTTPS 接続する際、サーバー証明書を検証するために必要

#### slim 版と標準版の違い

| イメージ          | CA 証明書数        | サイズ | VPC エンドポイント対応 |
| ----------------- | ------------------ | ------ | ---------------------- |
| `node:*-slim`     | 50-100個（最小限） | ~200MB | ❌ 一部の CA が欠落    |
| `node:*-bookworm` | 200-300個（完全）  | ~1GB   | ✅ すべての CA を含む  |

### 2. VPC エンドポイントの証明書検証

VPC 内で CodeBuild を実行する場合:

1. **Private DNS が有効**: `secretsmanager.ap-northeast-1.amazonaws.com` が VPC エンドポイントのプライベート IP に解決される
2. **VPC エンドポイントの SSL 証明書**: AWS が発行した正規の証明書が使用される
3. **証明書検証**: その証明書を発行した CA（例: Amazon Root CA）がクライアント側の信頼リストに必要

slim 版では、**Amazon Trust Services などの特定の CA 証明書が不足**していたため、検証に失敗していました。

### 3. DOWNLOAD_SOURCE フェーズのタイミング

- `DOWNLOAD_SOURCE` フェーズは、**Docker コンテナが起動する前**に CodeBuild エージェント（ホスト OS）が実行
- この段階で Secrets Manager 環境変数を解決するため、使用する Docker イメージの CA 証明書が重要
- buildspec で CA 証明書をインストールしても、このフェーズには間に合わない

## 解決策

### Docker イメージを slim 版から標準版に変更

`node:*-slim` から `node:*-bookworm` に変更。

### VPC エンドポイントに Secrets Manager を追加

インバウンドルール

| タイプ | プロトコル | ポート範囲 | ソース                     | 説明                                                                |
| ------ | ---------- | ---------- | -------------------------- | ------------------------------------------------------------------- |
| HTTPS  | TCP        | 443        | xxx.xx.xxx.x/xx (VPC CIDR) | VPC 内のすべてのリソースから Secrets Manager API へのアクセスを許可 |

アウトバウンドルール

| タイプ     | プロトコル | ポート範囲 | 送信先 | 説明                                         |
| ---------- | ---------- | ---------- | ------ | -------------------------------------------- |
| すべて削除 | -          | -          | -      | VPC エンドポイント自体からの外向き通信は不要 |

## 試したけど効果がなかった解決策

### 1. buildspec での CA 証明書インストール

```yaml
install:
  commands:
    - apt-get update && apt-get install -y ca-certificates
    - update-ca-certificates
```

これは `DOWNLOAD_SOURCE` フェーズより後に実行されるため、Secrets Manager 環境変数の解決には間に合わない。

## まとめ

CodeBuild で VPC 内から Secrets Manager にアクセスする際の TLS 証明書検証エラーは、**Docker イメージの CA 証明書が不足していることが原因**だった。

**解決策**: slim 版のイメージを避け、標準版の `node:*-bookworm` または AWS 管理の `aws/codebuild/standard:*` イメージを使用することで、VPC エンドポイント経由でも正常に動作する。
