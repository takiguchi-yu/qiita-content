---
title: AWSアカウントを跨いで利用料金を取得する
tags:
  - AWS
  - lambda
  - AssumeRole
  - CostExplorer
  - 利用料金
private: false
updated_at: '2021-11-14T14:42:37+09:00'
id: 565abbcde1dcaaf75d12
organization_url_name: null
slide: false
ignorePublish: false
---
`Assume Role` を使ってAWSの別アカウントから利用料金を取得します。

仕組みは次のようなイメージになります。
① で `STS` の `AssumeRole` を叩くと、一時的な**アクセスキー、シークレット、セッショントークン**が払い出されます。あとはこれを使うだけです。

<img src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/59081/45c7057e-e2ae-42a8-e306-a5859102921a.png" width="700px">

# AWSアカウントBにロールを作成

## ポリシー作成

今回はコスト参照するだけなので、以下のようなポリシーを作ります。

```json:GetCostAndUsagePolicy
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "VisualEditor0",
            "Effect": "Allow",
            "Action": "ce:GetCostAndUsage",
            "Resource": "*"
        }
    ]
}
```

## ロール作成

「別のAWS アカウント」を選択して、AWSアカウントA のアカウントIDを指定します。

<img src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/59081/091fecdc-67a6-8c82-2126-ce94559bbd62.png" width="700px">

先ほど作成した GetCostAndUsagePolicy をアタッチします。ロール名は GetCostAndUsageRole とかにしてみます。

<img src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/59081/9d985e96-4e19-e2d4-2d9c-a3d59801370f.png" width="700px">

# アカウントA側でLambdaを実行

## Lambda を構築

以前の記事を参考に構築します。

https://qiita.com/takiguchi-yu/items/d2c87d49ce07af194865

## 今回のLambdaのソースコード

```python
import os
import datetime as dt
import pandas as pd
import matplotlib.pyplot as plt
import boto3

s3 = boto3.client('s3')

def lambda_handler(event, context):
    # 現在日付を取得
    now = dt.datetime.now()
    start = (now - dt.timedelta(days=8)).strftime('%Y-%m-%d')
    end = now.strftime('%Y-%m-%d')
    date_index = pd.date_range(start, periods=8, freq="D")
    date_ary = date_index.to_series().dt.strftime("%-m/%-d")
    
    acct_name = ''  # アカウント名
    data = []       # pandasで使うデータの２次元配列
    acct = []       # アカウントリスト
    # AWSアカウント分ループ
    for i, role_arn in enumerate(os.environ['AWS_ASSUME_ROLE_ARN'].split(',')):
        acct_name = role_arn.split(':')[4]
        acct.append(acct_name)
        
        # STS:AssumeRole
        sts_connection = boto3.client('sts')
        acct_b = sts_connection.assume_role(
            RoleArn=role_arn,
            RoleSessionName="cross_acct_lambda"
        )
        ACCESS_KEY = acct_b['Credentials']['AccessKeyId']
        SECRET_KEY = acct_b['Credentials']['SecretAccessKey']
        SESSION_TOKEN = acct_b['Credentials']['SessionToken']
        
        ce = boto3.client(
            'ce',
            aws_access_key_id=ACCESS_KEY,
            aws_secret_access_key=SECRET_KEY,
            aws_session_token=SESSION_TOKEN,
        )
        
        # AWS Cost Explorer 実行
        response = ce.get_cost_and_usage (
            TimePeriod = {
                'Start': start,
                'End': end,
            },
            Granularity='DAILY',
            Metrics = [
                'AmortizedCost'
            ],
        )
    
        # 日ごとにループ
        data_row = []
        for result in response['ResultsByTime']:
            amount = round(float(result['Total']['AmortizedCost']['Amount']), 1)
            date = dt.datetime.strptime(result['TimePeriod']['Start'], '%Y-%m-%d').strftime('%-m/%-d')
            data_row.append(amount)
            
        data.append(data_row)
        
    dataset = pd.DataFrame(data, 
                         columns=date_ary.values, 
                         index=acct)
        
    # 積み上げ棒グラフ内にデータラベルを表示する
    fig, ax = plt.subplots(figsize=(12, 8))
    for i in range(len(dataset)):
        ax.bar(dataset.columns, dataset.iloc[i], bottom=dataset.iloc[:i].sum())
        for j in range(len(dataset.columns)): 
            plt.text(x=j, 
                  y=dataset.iloc[:i, j].sum() + (dataset.iloc[i, j] / 2), 
                  s=dataset.iloc[i, j], 
                  ha='center', 
                  va='bottom'
                )
    ax.set(xlabel='', ylabel='Cost($)')
    ax.legend(dataset.index)
    plt.show()
    # 作成した積み上げ棒グラフを一時保存する
    fig.savefig('/tmp/cost2.png')    
    
    # S3アップロード
    s3_upload_file('/tmp/cost2.png', os.environ['S3_BUCKET'], 'cost2.png')
    
def s3_upload_file(file_name, bucket, object_name):
    """S3にmatplotlibで作成したファイルをアップロードする"""

    response = s3.upload_file(file_name, bucket, object_name, 
        ExtraArgs={
            'ACL': 'public-read',
            'ContentType': 'image/png',
        }
    )
```

## 環境変数

|キー|値|
|-|-|
| S3_BUCKET |S3バケット名|
| AWS_ASSUME_ROLE_ARN |arn:aws:iam::アカウントB:role/GetCostAndUsageRole,<br>arn:aws:iam::アカウントC:role/GetCostAndUsageRole<br>※カンマ区切りで複数指定|

## 実行結果

集計したAWSアカウントの合計を棒グラフにしています。

<img src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/59081/bf815d9c-e26b-70d3-91b0-e4d70b811848.jpeg" width="500">

# 参考

https://aws.amazon.com/jp/premiumsupport/knowledge-center/lambda-function-assume-iam-role/

https://dev.classmethod.jp/articles/try-to-access-the-resource-of-another-account-from-lambda/

