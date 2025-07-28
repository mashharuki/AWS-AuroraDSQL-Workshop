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

## ハンズオンページ
[Amazon Aurora DSQL Immersion Day](https://catalog.workshops.aws/aurora-dsql/ja-JP/01-getting-started)
