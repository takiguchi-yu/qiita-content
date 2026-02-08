---
title: Terraformで構築したRDS（Amazon Aurora PostgreSQL）のメジャーアップグレード
tags:
  - RDS
  - Terraform
private: false
updated_at: ''
id: null
organization_url_name: null
slide: false
ignorePublish: false
---

## やりたいこと

Terraformで構築したAmazon Aurora PostgreSQLのRDSクラスタをメジャーアップグレードしたい。  
今回は PostgreSQL を 13.x → 17.x にアップグレードする。  
アップグレードにともなうダウンタイムは許容する。

## エラー内容

`terraform apply` を実行すると、以下のようなエラーが発生してアップグレードに失敗した。

```sh
module.default.aws_db_parameter_group.postgres_13: Destroying... [id=foobar-instance-postgres-13]
module.default.aws_db_parameter_group.postgres_17: Creating...
module.default.aws_rds_cluster_parameter_group.postgres_17: Creating...
module.default.aws_db_parameter_group.postgres_17: Creation complete after 2s [id=foobar-instance-postgres-17]
module.default.aws_rds_cluster_parameter_group.postgres_17: Creation complete after 3s [id=foobar-cluster-postgres-17]
module.default.aws_rds_cluster.foobar: Modifying... [id=foobar]
module.default.aws_db_parameter_group.postgres_13: Still destroying... [id=foobar-instance-postgres-13, 10s elapsed]
module.default.aws_db_parameter_group.postgres_13: Still destroying... [id=foobar-instance-postgres-13, 20s elapsed]
module.default.aws_db_parameter_group.postgres_13: Still destroying... [id=foobar-instance-postgres-13, 30s elapsed]
module.default.aws_db_parameter_group.postgres_13: Still destroying... [id=foobar-instance-postgres-13, 40s elapsed]
module.default.aws_db_parameter_group.postgres_13: Still destroying... [id=foobar-instance-postgres-13, 50s elapsed]
module.default.aws_db_parameter_group.postgres_13: Still destroying... [id=foobar-instance-postgres-13, 1m0s elapsed]
module.default.aws_db_parameter_group.postgres_13: Still destroying... [id=foobar-instance-postgres-13, 1m10s elapsed]
module.default.aws_db_parameter_group.postgres_13: Still destroying... [id=foobar-instance-postgres-13, 1m20s elapsed]
module.default.aws_db_parameter_group.postgres_13: Still destroying... [id=foobar-instance-postgres-13, 1m30s elapsed]
module.default.aws_db_parameter_group.postgres_13: Still destroying... [id=foobar-instance-postgres-13, 1m40s elapsed]
module.default.aws_db_parameter_group.postgres_13: Still destroying... [id=foobar-instance-postgres-13, 1m50s elapsed]
module.default.aws_db_parameter_group.postgres_13: Still destroying... [id=foobar-instance-postgres-13, 2m0s elapsed]
module.default.aws_db_parameter_group.postgres_13: Still destroying... [id=foobar-instance-postgres-13, 2m10s elapsed]
module.default.aws_db_parameter_group.postgres_13: Still destroying... [id=foobar-instance-postgres-13, 2m20s elapsed]
module.default.aws_db_parameter_group.postgres_13: Still destroying... [id=foobar-instance-postgres-13, 2m30s elapsed]
module.default.aws_db_parameter_group.postgres_13: Still destroying... [id=foobar-instance-postgres-13, 2m40s elapsed]
module.default.aws_db_parameter_group.postgres_13: Still destroying... [id=foobar-instance-postgres-13, 2m50s elapsed]
╷
│ Error: deleting RDS DB Parameter Group (foobar-instance-postgres-13): operation error RDS: DeleteDBParameterGroup, https response error StatusCode: 400, RequestID: , InvalidDBParameterGroupState: One or more database instances are still members of this parameter group foobar-instance-postgres-13, so the group cannot be deleted
│
│
╵
╷
│ Error: updating RDS Cluster (foobar): InvalidParameterCombination: The current DB instance parameter group foobar-instance-postgres-13 is custom. You must explicitly specify a new DB instance parameter group, either default or custom, for the engine version upgrade.
│
│
│   with module.default.aws_rds_cluster.foobar,
│   on ../../modules/rds.tf line 5, in resource "aws_rds_cluster" "foobar":
│    5: resource "aws_rds_cluster" "foobar" {
│
╵
make: *** [apply] Error 1
```

## 原因

Terraform は `replace` の場合、 `create` の前に `destroy` が実行されるようで、そのため、RDSクラスタにまだ紐づいているパラメータグループを削除しようとしてエラーになっていた。

## 今回試した解決策

Terraform 単体ではアップグレードできなかったため、`AWS CLI` と組み合わせてアップグレードを実施した。

AWS CLI で以下を実施：

1. インスタンスをデフォルトのパラメータグループ（default.aurora-postgresql13）に一時変更
1. クラスターのエンジンバージョンを17.7にアップグレード
1. アップグレード完了後、インスタンスをカスタムパラメータグループ（foobar-instance-postgres-17）に戻す予定
1. 最後にTerraformで状態を同期

## まとめ

- カスタムパラメータグループを使用しているケースで、Terraform の仕様により直接アップグレードできなかった
- AWS CLIで手動介入が必要だった

## 参考ページ

- https://techblog.asia-quest.jp/202312/major-version-update-of-rds-amazon-aurora-mysql-built-with-terraform-was-unexpectedly-troublesome
