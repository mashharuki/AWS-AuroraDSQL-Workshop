# AWS-AuroraDSQL-Workshop
Amzon Aurora DSQLのハンズオン用のリポジトリ

## リージョン
- us-east-1
- us-east-2

## ハンズオンの手順

1つ目のリージョンでCloudShellを立ち上げて以下を実行

```bash
cd ~
wget 'https://static.us-east-1.prod.workshops.aws/public/23fae69d-085a-46b0-8298-f15c5d3c7dc2/static/workshop.zip' -O workshop.zip
unzip workshop.zip
cd workshop
chmod u+x setup.sh
sh setup.sh
```

以下のようになればOK!

```bash
Code Bucket:       code-796032104877-us-east-1
App Template URL:  https://code-796032104877-us-east-1.s3.amazonaws.com/rewards-app-stack.yml
```

環境変数を設定する

```bash
source ~/workshop/init.rc
```

Javaがインストールされているかを確認する

```bash
java --version
```

mavenがインストールされているかを確認する

```bash
mvn --version
```

使用する2つのリージョンを指定してDSQLのクラスターを作成する。

それぞれのリージョンで以下の情報をメモすること

- Cluster ID
- クラスターのEndpoint
- クラスターの Amazon Resource Name (ARN)

今回だと以下のような値となる

- us-east-1
  - Cluster ID  
    7aabuioufvzv7efit7d3hz2r6a
  - クラスターのEndpoint  
    7aabuioufvzv7efit7d3hz2r6a.dsql.us-east-1.on.aws
  - クラスターの Amazon Resource Name (ARN)  
    arn:aws:dsql:us-east-1:796032104877:cluster/7aabuioufvzv7efit7d3hz2r6a
- us-east-2
  - Cluster ID  
    w4abuiouh2lg7cszsxlx7mc25u
  - クラスターのEndpoint  
    w4abuiouh2lg7cszsxlx7mc25u.dsql.us-west-2.on.aws
  - クラスターの Amazon Resource Name (ARN)  
    arn:aws:dsql:us-west-2:796032104877:cluster/w4abuiouh2lg7cszsxlx7mc25u

## ハンズオンページ
[Amazon Aurora DSQL Immersion Day](https://catalog.workshops.aws/aurora-dsql/ja-JP/01-getting-started)
