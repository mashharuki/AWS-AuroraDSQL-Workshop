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

us-east-1のAurora DSQLに接続する

```bash
echo "export CLUSTER_ENDPOINT=7aabuioufvzv7efit7d3hz2r6a.dsql.us-east-1.on.aws" >> ~/workshop/init.rc
source ~/workshop/init.rc
PGSSLMODE=require
```

以下のコマンドで認証トークンを作成する

有効時間は15分

```bash
export PGPASSWORD=$(aws dsql generate-db-connect-admin-auth-token --hostname $CLUSTER_ENDPOINT --expires-in 14400)
```

以下で接続

```bash
psql --quiet --username admin --dbname postgres --host $CLUSTER_ENDPOINT
```

もう片方のリージョンでも同様に接続する

```bash
echo "export CLUSTER_ENDPOINT=w4abuiouh2lg7cszsxlx7mc25u.dsql.us-west-2.on.aws" >> ~/workshop/init.rc
source ~/workshop/init.rc
PGSSLMODE=require
```

以下のコマンドで認証トークンを作成する

有効時間は15分

```bash
export PGPASSWORD=$(aws dsql generate-db-connect-admin-auth-token --hostname $CLUSTER_ENDPOINT --expires-in 14400)
```

以下で接続

```bash
psql --quiet --username admin --dbname postgres --host $CLUSTER_ENDPOINT
```

以下のコマンドでデータベースを作成する

```bash
\include ~/workshop/rewards-backend/src/main/sql/ddl.sql
```

以下のselect文で確認する

```sql
select * from sys.jobs;
```

両方のリージョンで同じ結果が返ってくるはず

```bash
postgres=> select * from sys.jobs;
           job_id           |  status   | details |  job_type   | class_id | object_id |                 object_name                 |       start_time       |      update_time       
----------------------------+-----------+---------+-------------+----------+-----------+---------------------------------------------+------------------------+------------------------
 p563k7emmbftjjoiiqdunomltq | completed |         | INDEX_BUILD |     1259 |     26391 | xpoints.customers_username_idx              | 2025-07-28 10:41:44+00 | 2025-07-28 10:41:53+00
 zei5tszlprd7dhrlxistbao7lq | completed |         | INDEX_BUILD |     1259 |     26403 | xpoints.catalog_images_item_id_idx          | 2025-07-28 10:41:44+00 | 2025-07-28 10:41:58+00
 psyvbyxepzdkbay7jfnpdncryi | completed |         | INDEX_BUILD |     1259 |     26409 | xpoints.shopping_cart_items_customer_id_idx | 2025-07-28 10:41:45+00 | 2025-07-28 10:41:54+00
 f5t6rcwqabdfvmimxstommpfgq | completed |         | INDEX_BUILD |     1259 |     26422 | xpoints.transactions_customer_id_idx        | 2025-07-28 10:41:45+00 | 2025-07-28 10:41:53+00
 g4yhzdiq4ffl7ahcyfqy2ztgoe | completed |         | INDEX_BUILD |     1259 |     26428 | xpoints.order_items_tx_id_idx               | 2025-07-28 10:41:46+00 | 2025-07-28 10:41:59+00
(5 rows)
```

PostgreSQLデータベースと同様に、psqlコマンドを使用してスキーマを探索し、データベース内のオブジェクトを説明できます。例えば、xpointsスキーマ内のすべてのテーブルを一覧表示するには：

```bash
\dt xpoints.*
```

```bash
               List of relations
 Schema  |        Name         | Type  | Owner 
---------+---------------------+-------+-------
 xpoints | catalog_images      | table | admin
 xpoints | catalog_items       | table | admin
 xpoints | customers           | table | admin
 xpoints | image_urls          | table | admin
 xpoints | images              | table | admin
 xpoints | order_items         | table | admin
 xpoints | points_balances     | table | admin
 xpoints | shopping_cart_items | table | admin
 xpoints | transactions        | table | admin
(9 rows)

```

xpoints.customersテーブルを説明するには、次のコマンドを実行します：

```bash
\d xpoints.customers
```

```bash
                            Table "xpoints.customers"
   Column    |          Type          | Collation | Nullable |      Default      
-------------+------------------------+-----------+----------+-------------------
 id          | uuid                   |           | not null | gen_random_uuid()
 username    | character varying(50)  |           |          | 
 first_name  | character varying(50)  |           |          | 
 last_name   | character varying(50)  |           |          | 
 maiden_name | character varying(50)  |           |          | 
 gender      | character varying(10)  |           |          | 
 email       | character varying(50)  |           |          | 
 phone_num   | character varying(20)  |           |          | 
 age         | integer                |           |          | 
 address     | character varying(100) |           |          | 
 city        | character varying(50)  |           |          | 
 state       | character varying(25)  |           |          | 
 state_code  | character varying(20)  |           |          | 
 postal_code | character varying(10)  |           |          | 
Indexes:
    "customers_pkey" PRIMARY KEY, btree_index (id) INCLUDE (username, first_name, last_name, maiden_name, gender, email, phone_num, age, address, city, state, state_code, postal_code)
    "customers_username_idx" btree_index (username)
```

クエリを実行する

```bash
select * from xpoints.customers;
```

```bash
 id | username | first_name | last_name | maiden_name | gender | email | phone_num | age | address | city | state | state_code | postal_code 
----+----------+------------+-----------+-------------+--------+-------+-----------+-----+---------+------+-------+------------+-------------
(0 rows)
```

サンプルデータをロードします。

```bash
\include ~/workshop/rewards-backend/src/main/sql/load_rewards_data.sql
```

さっきまでは空だったが、もう一度クエリを実行すると結果が返ってくる

```bash
                  id                  |               username               | first_name  |  last_name  | maiden_name | gender |                email                 | phone_num | age |    address    |   city   |     state      | s
tate_code | postal_code 
--------------------------------------+--------------------------------------+-------------+-------------+-------------+--------+--------------------------------------+-----------+-----+---------------+----------+----------------+--
----------+-------------
 0000a2c0-71c2-473a-856b-ffb89526d5e5 | carlos_salazar_121@example.mil       | Carlos      | Salazar     | García      | Other  | carlos_salazar_121@example.mil       | 555-0145  |  47 | 5425 O Street | Any Town | South Carolina | S
C         | 42591
 0005c022-5872-4798-8d44-6fb8d7ef1a56 | kwaku_mensah_108@example.net         | Kwaku       | Mensah      | Jayashankar | Male   | kwaku_mensah_108@example.net         | 555-0116  |  24 | 8851 Q Street | Nowhere  | Virginia       | V
A         | 94245
 001220df-486c-470b-a8e6-126de7aa0010 | efua_owusu_152@example.org           | Efua        | Owusu       | Whitlock    | Female | efua_owusu_152@example.org           | 555-0104  |  32 | 9282 J Street | Anytown  | California     | C
A         | 10603
 00152be0-72d3-4dfc-be63-aaa0596991b9 | nikki_wolf_132@example.edu           | Nikki       | Wolf        | Manu        | Female | nikki_wolf_132@example.edu           | 555-0111  |  54 | 2996 Z Street | Anywhere | Minnesota      | M
N         | 34789
 003b4d36-7628-45ad-8ea6-10804a545006 | nikhil_jayashankar_17@example.edu    | Nikhil      | Jayashankar | Hong        | Male   | nikhil_jayashankar_17@example.edu    | 555-0114  |  65 | 6211 W Street | Anytown  | Maryland       | M
D         | 75373
 0047d727-1384-4688-b875-b4f70025cbf4 | xiulan_wang_20@example.ca            | Xiulan      | Wang        | Li          | Female | xiulan_wang_20@example.ca            | 555-0162  |  26 | 1392 L Street | Any Town | Iowa           | I
A         | 48369
 008a49ef-ea41-4a4e-b694-73150b2395a7 | nikki_wolf_62@example.com            | Nikki       | Wolf        | Candella    | Female | nikki_wolf_62@example.com            | 555-0197  |  54 | 4744 V Street | Any Town | Maine          | M
E         | 66000
 0092d885-5208-4598-b7f4-08606c33fe5b | diego_ramirez_39@example.gov         | Diego       | Ramirez     | Stiles      | Other  | diego_ramirez_39@example.gov         | 555-0196  |  20 | 7829 M Street | Anywhere | Arkansas       | A
R         | 71093
 009a7306-7567-4ada-8ec8-9bf9c4ef7ecb | terry_whitlock_65@example.co.uk      | Terry       | Whitlock    | Ki          | Male   | terry_whitlock_65@example.co.uk      | 555-0191  |  50 | 7338 A Street | Any Town | Hawaii         | H
I         | 74457
 00a84b94-c434-407a-9019-2887d5d4af14 | shirley_rodriguez_75@example.ca      | Shirley     | Rodriguez   | Doe         | Female | shirley_rodriguez_75@example.ca      | 555-0111  |  49 | 7519 K Street | Anytown  | Vermont        | V
T         | 81910
 00c9d2a4-5fca-4f8c-aec2-918d04a8bae0 | kwesi_manu_8@example.nz              | Kwesi       | Manu        | Salazar     | Female | kwesi_manu_8@example.nz              | 555-0199  |  39 | 9655 B Street | Any Town | New Mexico     | N
M         | 65996
 00da3e10-19f9-4708-9e4f-a2ba127a2d0f | sofia_martinez_31@example.gov        | Sofia       | Martinez    | Whitlock    | Other  | sofia_martinez_31@example.gov        | 555-0113  |  55 | 3907 C Street | Anytown  | Kansas         | K
S         | 91700
 00e3632e-56f8-470c-b7fb-5c562b245605 | mary_major_132@example.au            | Mary        | Major       | Rivera      | Other  | mary_major_132@example.au            | 555-0128  |  43 | 4055 R Street | Nowhere  | California     | C
A         | 86594
 00eb67c7-781b-480e-9811-36415afeb2f3 | jane_doe_137@example.co.uk           | Jane        | Doe         | Doe         | Female | jane_doe_137@example.co.uk           | 555-0138  |  25 | 6624 G Street | Anytown  | Alaska         | A
K         | 41776
 00ed472a-cfca-4256-8622-d6dc3c69a908 | gil-dong_hong_41@example.com         | Gil-dong    | Hong        | Oliveria    | Female | gil-dong_hong_41@example.com         | 555-0192  |  60 | 2079 J Street | Anytown  | Minnesota      | M
N         | 63681
 00f0e67d-ca12-41ba-8199-492030d61efb | kwaku_mensah_66@example.org          | Kwaku       | Mensah      | Li          | Other  | kwaku_mensah_66@example.org          | 555-0113  |  35 | 1325 O Street | Any Town | New Jersey     | N
J         | 81603
 00f8c4d3-97e9-4e2f-bc9e-29a32b1a280c | kwesi_manu_110@example.com           | Kwesi       | Manu        | Candella    | Other  | kwesi_manu_110@example.com           | 555-0103  |  33 | 6069 Y Street | Anytown  | Ohio           | O
H         | 77578
 010ad45c-2c6c-4016-b9d4-e68410afd887 | nikki_wolf_45@example.co.uk          | Nikki       | Wolf        | Ki          | Female | nikki_wolf_45@example.co.uk          | 555-0104  |  64 | 5265 S Street | Anywhere | Idaho          | I
D         | 67320
 011b67db-0394-4aab-903b-ade1f8a6d4d9 | jie_liu_106@example.ca               | Jie         | Liu         | Martínez    | Other  | jie_liu_106@example.ca               | 555-0199  |  54 | 2162 H Street | Any Town | Colorado       | C
O         | 85813
 011fca7c-a258-48ab-89e8-281c241fbb28 | kwesi_manu_2@example.net             | Kwesi       | Manu        | Jayashankar | Male   | kwesi_manu_2@example.net             | 555-0162  |  26 | 8182 U Street | Anytown  | Utah           | U
T         | 17478
 01277d11-f49e-49a6-9692-49134a7c0b23 | shirley_rodriguez_2@example.org      | Shirley     | Rodriguez   | Hong        | Female | shirley_rodriguez_2@example.org      | 555-0182  |  41 | 8534 D Street | Any Town | New Hampshire  | N
H         | 27879
 013d5e90-d0e0-4b45-aa94-9809d2147f36 | john_stiles_91@example.au            | John        | Stiles      | Ki          | Female | john_stiles_91@example.au            | 555-0130  |  54 | 8606 H Street | Nowhere  | Texas          | T
X         | 44169
```

以下のクエリを実行する

```bash
select * from xpoints.customers limit 2;
```

実行結果

```bash
                  id                  |            username            | first_name | last_name | maiden_name | gender |             email              | phone_num | age |    address    |   city   |     state      | state_code | pos
tal_code 
--------------------------------------+--------------------------------+------------+-----------+-------------+--------+--------------------------------+-----------+-----+---------------+----------+----------------+------------+----
---------
 0000a2c0-71c2-473a-856b-ffb89526d5e5 | carlos_salazar_121@example.mil | Carlos     | Salazar   | García      | Other  | carlos_salazar_121@example.mil | 555-0145  |  47 | 5425 O Street | Any Town | South Carolina | SC         | 425
91
 0005c022-5872-4798-8d44-6fb8d7ef1a56 | kwaku_mensah_108@example.net   | Kwaku      | Mensah    | Jayashankar | Male   | kwaku_mensah_108@example.net   | 555-0116  |  24 | 8851 Q Street | Nowhere  | Virginia       | VA         | 942
45
(2 rows)

```

もう片方のリージョンでも実行すると同じデータが確認できるはず

以下のコマンドを実行

```sql
create index async on xpoints.customers (phone_num);
```

```bash
           job_id           
----------------------------
 iludea4dtzgtbanhrt6uq6tepa
(1 row)
```

続いて以下のクエリを実行する

```sql
select * from sys.jobs;
```

以下は実行結果

```bash
           job_id           |  status   | details |  job_type   | class_id | object_id |                 object_name                 |       start_time       |      update_time       
----------------------------+-----------+---------+-------------+----------+-----------+---------------------------------------------+------------------------+------------------------
 p563k7emmbftjjoiiqdunomltq | completed |         | INDEX_BUILD |     1259 |     26391 | xpoints.customers_username_idx              | 2025-07-28 10:41:44+00 | 2025-07-28 10:41:53+00
 zei5tszlprd7dhrlxistbao7lq | completed |         | INDEX_BUILD |     1259 |     26403 | xpoints.catalog_images_item_id_idx          | 2025-07-28 10:41:44+00 | 2025-07-28 10:41:58+00
 psyvbyxepzdkbay7jfnpdncryi | completed |         | INDEX_BUILD |     1259 |     26409 | xpoints.shopping_cart_items_customer_id_idx | 2025-07-28 10:41:45+00 | 2025-07-28 10:41:54+00
 f5t6rcwqabdfvmimxstommpfgq | completed |         | INDEX_BUILD |     1259 |     26422 | xpoints.transactions_customer_id_idx        | 2025-07-28 10:41:45+00 | 2025-07-28 10:41:53+00
 g4yhzdiq4ffl7ahcyfqy2ztgoe | completed |         | INDEX_BUILD |     1259 |     26428 | xpoints.order_items_tx_id_idx               | 2025-07-28 10:41:46+00 | 2025-07-28 10:41:59+00
 urwdwvwi5nh45gizsrbwebp77y | completed |         | ANALYZE     |     1259 |     26429 | xpoints.images                              | 2025-07-28 10:45:25+00 | 2025-07-28 10:45:34+00
 rcclsf6g2zgzng6rmo4ul6p5km | completed |         | ANALYZE     |     1259 |     26385 | xpoints.customers                           | 2025-07-28 10:45:26+00 | 2025-07-28 10:45:35+00
 h4gqeb6sffb3bpqg55odxtcyly | completed |         | ANALYZE     |     1259 |     26410 | xpoints.points_balances                     | 2025-07-28 10:45:26+00 | 2025-07-28 10:45:34+00
 wax6tiokdzhj3hmjd3hf4jxck4 | completed |         | ANALYZE     |     1259 |     26392 | xpoints.catalog_items                       | 2025-07-28 10:45:27+00 | 2025-07-28 10:45:40+00
 p54d4jeajfghpdnk5ol2tros7a | completed |         | ANALYZE     |     1259 |     26398 | xpoints.catalog_images                      | 2025-07-28 10:45:27+00 | 2025-07-28 10:45:50+00
 wckiu3g3fvan5ajmudttc5bvci | completed |         | ANALYZE     |     1259 |     26415 | xpoints.transactions                        | 2025-07-28 10:45:27+00 | 2025-07-28 10:45:40+00
 rtsxhrf6qzbaxh57chiwzqksze | completed |         | ANALYZE     |     1259 |     26423 | xpoints.order_items                         | 2025-07-28 10:45:28+00 | 2025-07-28 10:45:41+00
 yowjnpjkcjebxfvwgzlw2ewcfa | completed |         | ANALYZE     |     1259 |     26404 | xpoints.shopping_cart_items                 | 2025-07-28 10:45:28+00 | 2025-07-28 10:45:35+00
 iludea4dtzgtbanhrt6uq6tepa | completed |         | INDEX_BUILD |     1259 |     26460 | xpoints.customers_phone_num_idx             | 2025-07-28 10:49:20+00 | 2025-07-28 10:49:27+00
(14 rows)
```

以下のクエリでインデックスのステータスを確認する

```sql
select indisvalid from pg_index where indexrelid ='xpoints.customers_phone_num_idx'::regclass::oid;
```

tであればインデックスが使用可能であるということ

```bash
 indisvalid 
------------
 t
(1 row)
```

テーブルの説明のためにインデックスを表示

```bash
\d xpoints.customers
```

```bash
                            Table "xpoints.customers"
   Column    |          Type          | Collation | Nullable |      Default      
-------------+------------------------+-----------+----------+-------------------
 id          | uuid                   |           | not null | gen_random_uuid()
 username    | character varying(50)  |           |          | 
 first_name  | character varying(50)  |           |          | 
 last_name   | character varying(50)  |           |          | 
 maiden_name | character varying(50)  |           |          | 
 gender      | character varying(10)  |           |          | 
 email       | character varying(50)  |           |          | 
 phone_num   | character varying(20)  |           |          | 
 age         | integer                |           |          | 
 address     | character varying(100) |           |          | 
 city        | character varying(50)  |           |          | 
 state       | character varying(25)  |           |          | 
 state_code  | character varying(20)  |           |          | 
 postal_code | character varying(10)  |           |          | 
Indexes:
    "customers_pkey" PRIMARY KEY, btree_index (id) INCLUDE (username, first_name, last_name, maiden_name, gender, email, phone_num, age, address, city, state, state_code, postal_code)
    "customers_phone_num_idx" btree_index (phone_num)
    "customers_username_idx" btree_index (username)
```

インデックスが作成されていることを確認する

```sql
explain select phone_num from xpoints.customers limit 1;
```

```bash
                                                QUERY PLAN                                                 
------------------------------------------------------------------------------------------------------------
 Limit  (cost=725.03..725.08 rows=1 width=12)
   ->  Index Only Scan using customers_phone_num_idx on customers  (cost=725.03..943.03 rows=5000 width=12)
         Projected via pushdown compute engine: phone_num
(3 rows)
```

Aurora DSQLでは以下のようなSQLもサポートしている

```sql
SELECT name, sum(unit_cnt) AS sum_items
FROM xpoints.catalog_items
LEFT JOIN xpoints.order_items
ON xpoints.catalog_items.id = xpoints.order_items.cat_item_id
GROUP BY name
HAVING sum(unit_cnt) > 0
ORDER BY sum_items DESC, name;
```

```sql
EXPLAIN SELECT * FROM xpoints.customers limit 1;
```

Aurora DSQLのテーブルはヒープテーブルではなく、クラスタ化インデックステーブルです。

同時実行制御

https://catalog.workshops.aws/aurora-dsql/ja-JP/02-working-with-aurora-dsql/05-concurrency-control

**Aurora DSQLの楽観的同時実行制御では、最初にコミットしたトランザクションが勝ちます。**    
他の競合するトランザクションは、たとえ開始が先であっても、コミットを試みるとエラーで失敗します。

以下のコマンドでトランザクションを開始します。

```sql
begin;
select address from xpoints.customers where username = 'carlos_salazar_125@example.edu';
update xpoints.customers set address = '123 Main Street' where username = 'carlos_salazar_125@example.edu';
```

次にもう片方のリージョンでもトランザクションを開始する

```sql
begin;
select address from xpoints.customers where username = 'carlos_salazar_125@example.edu';
```

続きをする

```sql
update xpoints.customers set address = '201 Rocky Blvd' where username = 'carlos_salazar_125@example.edu';
commit;
```

もう片方のリージョンでコミットしようとするとエラーが出るはず

```sql
commit;
```

```bash
ERROR:  change conflicts with another transaction, please retry: (OC000)
```

以下のコマンドでどのクエリが成功したかを確認する

```sql
select address from xpoints.customers where username = 'carlos_salazar_125@example.edu';
```

```bash
    address     
----------------
 201 Rocky Blvd
(1 row)
```

同時実行保護は同時トランザクションにのみ適用される

以下のクエリを実行する

```sql
begin;
update xpoints.customers set address = '221B Baker Street' where username = 'carlos_salazar_125@example.edu';
commit;
```

もう片方のリージョンで以下のクエリを実行する

```sql
begin;
update xpoints.customers set address = '999 Wrong Way' where username = 'carlos_salazar_125@example.edu';
commit;
```

以下のクエリで最終的な結果を確認する

```sql
select address from xpoints.customers where username = 'carlos_salazar_125@example.edu';
```

以下のようになるはず

```bash
    address    
---------------
 999 Wrong Way
(1 row)
```

読み取り専用トランザクションの管理

以下のsqlステートメントを実行する

```sql
begin;
select age from xpoints.customers where username = 'carlos_salazar_125@example.edu';
```

もう片方のリージョンで以下のSQLを実行する

```sql
update xpoints.customers set age = 99 where username = 'carlos_salazar_125@example.edu';
```

元のリージョンに戻って以下のSQLを実行する

```sql
select age from xpoints.customers where username = 'carlos_salazar_125@example.edu';
commit;
```

値が変わっていない

```bash
 age 
-----
  65
(1 row)
```

以下のsqlを実行する

```sql
select age from xpoints.customers where username = 'carlos_salazar_125@example.edu';
```

↑ これは新しいトランザクションなので最新の値を返す

```bash

``` age 
-----
  99
(1 row)
```

Aurora DSQLはデータを更新しないトランザクションに対して同時実行チェックを実行しません。

次に書き込みスキューの管理

```sql
select cust.id, cust.username, bal.points_balance
from xpoints.customers cust
inner join xpoints.points_balances bal on bal.customer_id = cust.id
where cust.username in ('martha_rivera_88@example.com', 'carlos_salazar_11@example.au');
```

以下のような結果が返ってくるはず

```bash
                  id                  |           username           | points_balance 
--------------------------------------+------------------------------+----------------
 84c0b418-f3ad-4b8c-b51e-78a882a7d004 | martha_rivera_88@example.com |            283
 cde00641-e5e0-4987-b9c6-2957d0ed9b87 | carlos_salazar_11@example.au |            288
(2 rows)
```

次にカルロスのインセンティブを確認

```sql
begin;

select sum(points_balance) as household_balance from xpoints.points_balances
where customer_id in ('84c0b418-f3ad-4b8c-b51e-78a882a7d004', 'cde00641-e5e0-4987-b9c6-2957d0ed9b87');

-- The combined balance of 571 is less than 1,000, so boost Carlos' balance!

insert into xpoints.transactions (customer_id, tx_type, points, tx_description)
values ('cde00641-e5e0-4987-b9c6-2957d0ed9b87', 'BOOST', 429, 'Incentive boost!');

update xpoints.points_balances set points_balance = points_balance + 429 where customer_id = 'cde00641-e5e0-4987-b9c6-2957d0ed9b87';
```

次にもう片方のリージョンに切り替えてマーサのインセンティブを更新する

```sql
begin;

select sum(points_balance) as household_balance from xpoints.points_balances
where customer_id in ('84c0b418-f3ad-4b8c-b51e-78a882a7d004', 'cde00641-e5e0-4987-b9c6-2957d0ed9b87');

-- The combined balance of 571 is less than 1,000, so boost Martha's balance!

insert into xpoints.transactions (customer_id, tx_type, points, tx_description)
values ('84c0b418-f3ad-4b8c-b51e-78a882a7d004', 'BOOST', 429, 'Incentive boost!');

update xpoints.points_balances set points_balance = points_balance + 429 where customer_id = '84c0b418-f3ad-4b8c-b51e-78a882a7d004';

commit;
```

もう片方のリージョンに戻ってトランザクションをコミットさせる

```sql
commit;
```

以下のsqlを実行する

```sql
select cust.id, cust.username, bal.points_balance
from xpoints.customers cust
inner join xpoints.points_balances bal on bal.customer_id = cust.id
where cust.username in ('martha_rivera_88@example.com', 'carlos_salazar_11@example.au');
```

```bash
                  id                  |           username           | points_balance 
--------------------------------------+------------------------------+----------------
 84c0b418-f3ad-4b8c-b51e-78a882a7d004 | martha_rivera_88@example.com |            712
 cde00641-e5e0-4987-b9c6-2957d0ed9b87 | carlos_salazar_11@example.au |            717
(2 rows)
```

以下のクエリも実行する

```sql
select * from xpoints.transactions
where customer_id in ('84c0b418-f3ad-4b8c-b51e-78a882a7d004', 'cde00641-e5e0-4987-b9c6-2957d0ed9b87');
```

実行結果

```bash
                  id                  |             customer_id              | tx_type | points |           tx_dt            |  tx_description  
--------------------------------------+--------------------------------------+---------+--------+----------------------------+------------------
 3e8164dc-53e9-4ade-a4a6-52bd403e9e27 | 84c0b418-f3ad-4b8c-b51e-78a882a7d004 | BOOST   |    429 | 2025-07-28 11:14:27.09038  | Incentive boost!
 b19b78d2-e246-4016-afd3-f4d86b6ac875 | cde00641-e5e0-4987-b9c6-2957d0ed9b87 | BOOST   |    429 | 2025-07-28 11:13:37.697556 | Incentive boost!
(2 rows)
```

以下のクエリで元に戻す

```sql
begin;

delete from xpoints.transactions where customer_id in ('84c0b418-f3ad-4b8c-b51e-78a882a7d004', 'cde00641-e5e0-4987-b9c6-2957d0ed9b87');
update xpoints.points_balances set points_balance = 283 where customer_id = '84c0b418-f3ad-4b8c-b51e-78a882a7d004';
update xpoints.points_balances set points_balance = 288 where customer_id = 'cde00641-e5e0-4987-b9c6-2957d0ed9b87';

commit;

select cust.id, cust.username, bal.points_balance
from xpoints.customers cust
inner join xpoints.points_balances bal on bal.customer_id = cust.id
where cust.username in ('martha_rivera_88@example.com', 'carlos_salazar_11@example.au');
```

以下のようになっているはず

```bash
                  id                  |           username           | points_balance 
--------------------------------------+------------------------------+----------------
 84c0b418-f3ad-4b8c-b51e-78a882a7d004 | martha_rivera_88@example.com |            283
 cde00641-e5e0-4987-b9c6-2957d0ed9b87 | carlos_salazar_11@example.au |            288
(2 rows)
```

書き込みスキューから保護するために、Aurora DSQLはSELECTステートメントでFOR UPDATE句をサポートしています。悲観的同時実行制御を持つリレーショナルデータベースでは、これは選択された行をロックします。ただし、Aurora DSQLは悲観的ロックをサポートしていません。代わりに、Aurora DSQLでは、FOR UPDATEは読み取り行を同時実行チェック用にフラグ付けします。前述のように、Aurora DSQLは通常、読み取りレコードに対して同時実行チェックを実行しません。これはスナップショット分離の正しい動作です。SELECT...FOR UPDATEはこの動作を変更します。

上記を踏まえてもう一度カルロスの状態を更新する

select文の最後に for updateがついている

```sql
begin;

select points_balance from xpoints.points_balances where customer_id = '84c0b418-f3ad-4b8c-b51e-78a882a7d004' for update;
select points_balance from xpoints.points_balances where customer_id = 'cde00641-e5e0-4987-b9c6-2957d0ed9b87' for update;

-- The combined balance of 571 is less than 1,000, so boost Carlos' balance!

insert into xpoints.transactions (customer_id, tx_type, points, tx_description)
values ('cde00641-e5e0-4987-b9c6-2957d0ed9b87', 'BOOST', 429, 'Incentive boost!');

update xpoints.points_balances set points_balance = points_balance + 429 where customer_id = 'cde00641-e5e0-4987-b9c6-2957d0ed9b87';
```

もう片方のリージョンで実行する

```sql
begin;

select points_balance from xpoints.points_balances where customer_id = '84c0b418-f3ad-4b8c-b51e-78a882a7d004' for update;
select points_balance from xpoints.points_balances where customer_id = 'cde00641-e5e0-4987-b9c6-2957d0ed9b87' for update;

-- The combined balance of 571 is less than 1,000, so boost Martha's balance!

insert into xpoints.transactions (customer_id, tx_type, points, tx_description)
values ('84c0b418-f3ad-4b8c-b51e-78a882a7d004', 'BOOST', 429, 'Incentive boost!');

update xpoints.points_balances set points_balance = points_balance + 429 where customer_id = '84c0b418-f3ad-4b8c-b51e-78a882a7d004';

commit;
```

もう片方のリージョンに戻ってトランザクションをコミットさせようとするとエラーが起きる

```sql
commit;
```

```bash
ERROR:  change conflicts with another transaction, please retry: (OC000)
```

最終的な残高を確認すると

```bash
select cust.id, cust.username, bal.points_balance from xpoints.customers cust
inner join xpoints.points_balances bal on bal.customer_id = cust.id
where cust.username in ('martha_rivera_88@example.com', 'carlos_salazar_11@example.au');
```

以下になるはず(合計値がちゃんと1,000以内におさまっている！)

```bash
                  id                  |           username           | points_balance 
--------------------------------------+------------------------------+----------------
 84c0b418-f3ad-4b8c-b51e-78a882a7d004 | martha_rivera_88@example.com |            712
 cde00641-e5e0-4987-b9c6-2957d0ed9b87 | carlos_salazar_11@example.au |            288
(2 rows)
```

両方のリージョンで以下のコマンドを実行

```sql
\q
```

次にデータベースロールの作成を行います

Aurora DSQLでは、データベース認証にAWS Identity and Access Management (IAM) )を使用します。データベースロールはデータベースオブジェクトへのアクセス権限を定義します。IAMポリシーとロールはデータベースへの接続権限を付与します。データベースロールはAurora DSQLクラスター内のIAMロールにリンクされています。このセクションでは、読み取り専用ロールと読み書きロールを設定して、Aurora DSQLデータベースへのアクセス権限の付与方法を示します。

```bash
aws sts get-caller-identity
```

次に、Aurora DSQLへの接続権限を付与するIAMポリシーを作成します。us-east-1とus-east-2の両方のリージョンにあるAurora DSQLクラスターのARNが必要です。

IAMポリシーを作成していきます

actionの部分を以下のようにします。

```json
"Action": [
    "dsql:DbConnect",
    "dsql:GetCluster"
],
```

resourceの部分を以下のようにします。

```json
"Resource": [
    "arn:aws:dsql:us-east-1:796032104877:cluster/7aabuioufvzv7efit7d3hz2r6a",
    "arn:aws:dsql:us-west-2:796032104877:cluster/w4abuiouh2lg7cszsxlx7mc25u"
]
```

最終的に以下のようになればOK!

```json
{
	"Version": "2012-10-17",
	"Statement": [
		{
			"Sid": "Statement1",
			"Effect": "Allow",
			"Action": [
                "dsql:DbConnect",
                "dsql:GetCluster"
            ],
			"Resource": [
                "arn:aws:dsql:us-east-1:796032104877:cluster/7aabuioufvzv7efit7d3hz2r6a",
                "arn:aws:dsql:us-west-2:796032104877:cluster/w4abuiouh2lg7cszsxlx7mc25u"
            ]
		}
	]
}
```

次にIAMロールを作成します。

プリンシパルの部分を以下のようにします。

```json
{
	"Version": "2012-10-17",
	"Statement": [
		{
			"Sid": "Statement1",
			"Effect": "Allow",
			"Principal": {
                "AWS": "arn:aws:iam::000000000000:role/YourRoleNameHere"
            },
			"Action": "sts:AssumeRole"
		}
	]
}
```

作成したIAMロールのArn

arn:aws:iam::796032104877:role/RewardsReadOnlyAccess

以下でクラスターにログイン

```bash
~/workshop/bin/psqli.sh $CLUSTER_ENDPOINT
```

次に、xpointsスキーマ内のすべてのテーブルに対する読み取り専用アクセス権を持つreportsというデータベースロールを作成します

```sql
create role reports with login;
grant usage on schema xpoints to reports;
grant select on all tables in schema xpoints to reports;
```

次に、reportsデータベースロールを、上記で作成したRewardsReadOnlyAccess IAMロールにリンクします。

```bash
aws iam grant reports to 'arn:aws:iam::796032104877:role/RewardsReadOnlyAccess';
```

reportsロールとしてAurora DSQLにログインして、設定をテストします。us-east-1のCloudShellターミナルで、以下のコマンドを実行してRewardsReadOnlyAccessロールを引き受けます。

```bash
aws sts assume-role --role-session-name readonly --role-arn arn:aws:iam::796032104877:role/RewardsReadOnlyAccess
```

うまくいけば以下の3つのクレデンシャル情報が出力されるはずなので環境変数として設定する

```bash
export AWS_ACCESS_KEY_ID=<<value of AccessKeyId above>>
export AWS_SECRET_ACCESS_KEY=<<value of SecretAccessKey above>>
export AWS_SESSION_TOKEN=<<value of SessionToken above>>
```

aws sts get-caller-identityを実行して設定を確認します。Arnフィールドに読み取り専用ロールのARNが表示されるはずです。

```bash
export PGPASSWORD=$(aws dsql generate-db-connect-auth-token --hostname $CLUSTER_ENDPOINT)
```

以下のコマンドを実行してクラスターにログイン

```bash
psql --host $CLUSTER_ENDPOINT --username reports --dbname postgres 
```

読み取り権限のみ付与しているので以下は成功する

```sql
select first_name, last_name, age from xpoints.customers where id = '00152be0-72d3-4dfc-be63-aaa0596991b9';
```

以下は更新系なので失敗する

```sql
update xpoints.customers set age = 70 where id = '00152be0-72d3-4dfc-be63-aaa0596991b9';
```

```bash
ERROR:  permission denied for table customers
```

以下のコマンドを実行して読み取り専用ロールの引き受け効果を元に戻します：

```bash
unset AWS_ACCESS_KEY_ID
unset AWS_SECRET_ACCESS_KEY
unset AWS_SESSION_TOKEN
```

以下のコマンドで引き受ける

```bash
aws sts assume-role --role-session-name readwrite --role-arn arn:aws:iam::000000000000:role/RewardsReadWriteAccess
```

再度設定

```bash
export AWS_ACCESS_KEY_ID=<<value of AccessKeyId above>>
export AWS_SECRET_ACCESS_KEY=<<value of SecretAccessKey above>>
export AWS_SESSION_TOKEN=<<value of SessionToken above>>
```

以下のコマンドで新しいパスワードを生成

```bash
export PGPASSWORD=$(aws dsql generate-db-connect-auth-token --hostname $CLUSTER_ENDPOINT)
```

operationalロールとしてデータベースにログインする

```bash
psql --host $CLUSTER_ENDPOINT --username operations --dbname postgres 
```

以下のクエリを実行する

```sql
select first_name, last_name, age from xpoints.customers where id = '00152be0-72d3-4dfc-be63-aaa0596991b9';
```

```bash
 first_name | last_name | age 
------------+-----------+-----
 Nikki      | Wolf      |  54
(1 row)
```

更新系のコマンドを実行する

```sql
update xpoints.customers set age = 70 where id = '00152be0-72d3-4dfc-be63-aaa0596991b9';
```

もう一度selectする

```sql
select first_name, last_name, age from xpoints.customers where id = '00152be0-72d3-4dfc-be63-aaa0596991b9';
```

以下のようになるはず

```bash
first_name | last_name | age
------------+-----------+-----
Nikki      | Wolf      |  70
(1 row)
```

以下でクレデンシャル情報は解除させる

```bash
unset AWS_ACCESS_KEY_ID
unset AWS_SECRET_ACCESS_KEY
unset AWS_SESSION_TOKEN
```

## ハンズオンページ
[Amazon Aurora DSQL Immersion Day](https://catalog.workshops.aws/aurora-dsql/ja-JP/01-getting-started)
