# 初めに  
  
ElasticSearchについて何となくしか理解してなかったので、色々と調べてまとめます。  
間違っているところがあれば、ご指摘いただけると幸いです、  
  
# ElasticSearchとは  
  
ElasticSearchとは、「高速で柔軟な検索を可能にするインデックス型の全文検索エンジン」です。  
Elasticsearch を使用すると、あらゆるサイズ、形式のデータ​​をほぼリアルタイムで検索、作成、保存、分析できます。  
  
- **ログ解析**: システムやアプリケーションログを素早く検索。  
- **全文検索**: 大量の文書やデータを対象とした検索機能の実装。  
- **ECサイトの商品検索**: ユーザーが入力したキーワードに基づくリアルタイム検索。  
  
https://www.elastic.co/guide/en/elasticsearch/reference/current/elasticsearch-intro-what-is-es.html  
  
# 全文検索とは  
  
全文検索とは、文書全体を対象にして特定のキーワードやフレーズを探し出す検索方法です。  
全文検索には、主に**grep型**と**インデックス型**の二つの形式があります。  
  
### grep型  
  
**grep型**は、データ全体を順番に検索し、条件に一致する箇所を探す方法です。  
例えば、UNIXのgrepコマンドはこの方式を採用しています。  
この方式は実装がシンプルで、小規模なデータには有効ですが、大量のデータを処理する場合は非効率的です。  
  
### index型  
  
一方、**index型**は、データを事前に構造化して保存し、検索時にはそのインデックスを参照する方式です。  
この方法では、膨大なデータからでも必要な情報を高速に取得できます。  
ElasticSearchはこのインデックス型を採用しており、その高速性を支える仕組みが**転置インデックス**です。  
  
# 転置インデックス  
  
転置インデックスは、検索エンジンが情報を効率的に検索できるようにするデータ構造です。  
単語ごとに「その単語がどこに登場するか」を記録しており、検索時にはこのインデックスを参照するだけで関連するデータを見つけることができます。  
  
例えば、図書館で「りんご」という言葉が含まれる本を探したいとします。  
すべての本を1冊ずつ開いて「りんご」が出てくるか確認する方法では時間がかかります。  
一方で、あらかじめ「りんご」という言葉が出てくる本のリストを作っておけば、そのリストを参照するだけですぐに見つけることができます。  
この「あらかじめ作っておいたリスト」が転置インデックスにあたります。  
  
### 転置インデックスの仕組み  
  
### 文章のトークン化  
  
入力された文章を単語（トークン）に分割します。  
   
```md
文章1: "I love ElasticSearch"
-> 分割結果：`["I", "love", "ElasticSearch"]`
文章2: "ElasticSearch is powerful"
-> 分割結果：`["ElasticSearch", "is", "powerful"]`
文章3: "I love programming"
-> 分割結果：`["I", "love", "programming"]`
```  
  
#### インデックス作成  
  
各単語とその出現位置を記録します。  
  
| 単語            | 出現場所                          |  
|------------------|----------------------------------|  
| I               | 文章1:位置1,  文章3:位置1                    |  
| love            | 文章1:位置2, 文章3:位置2                    |  
| ElasticSearch   | 文章1:位置3, 文章2 :位置1,                   |  
| is              | 文章2:位置2                            |  
| powerful        | 文章2:位置3                            |  
| programming     | 文章3:位置3                            |  
  
ElasticSearchでは、検索クエリが「love」の場合、転置インデックスを参照して「文章1」「文章3」がすぐに返却されます。  
  
# ElasticSearchの基本コンセプト  
  
ElasticSearchはスケーラブルな分散型検索エンジンであり、複数のサーバーにデータを分散して管理することを前提とした設計が特徴です。  
基本構成要素は以下の通りです  
  
- **Cluster**  
ElasticSearch全体の管理単位で、複数のNodeから構成されます。  
- **Node**  
ElasticSearchのプロセスが動作するサーバです。  
 Kubenetesで言うPodです。  
- **Index**  
 Documentの集合体の論理概念です。  
データの作成、更新、検索、削除はすべてIndexを単位として操作します。  
- **Shard**  
 Indexで操作する実際のデータが保存される物理単位です。  
 Indexを分割して複数のNodeに分散配置されます。  
- **Document**  
 ElasticSearchで保存されるデータの最小単位です。  
 データはJSON形式で管理されます。  
```json
{
    "id": 1,
    "name": "John Doe",
    "age": 30,
    "tags": ["developer", "engineer"]
}
```  
  
詳細については、[メルカリの記事](https://engineering.mercari.com/blog/entry/20220311-97aec2a2f8/)が詳しいので参考にしてください。  
  
https://engineering.mercari.com/blog/entry/20220311-97aec2a2f8/  
  
# 実際に触ってみる   
  
ローカル環境で実際にElasticSearchのAPIを叩いて色々試してみます。  
  
## ローカルのDockerにElasticSearchを導入する。  
  
ローカルのDockerにElasticSearchと「ElasticSearhの可視化ツール」である**Kibana** を導入してシングルノード構成を構築します。  
  
基本、公式の手順に従いますが、ElasticSerarchのコンテナ起動の際に   
`-e "discovery.type=single-node"` をつける必要があるので、注意してください。  
  
```sh
#  1. ElasticSearchのDockerコンテナをpullして起動する。
docker network create elastic
docker pull docker.elastic.co/elasticsearch/elasticsearch:8.16.1
docker run --name es01 --net elastic -p 9200:9200 -it -m 2GB --rm -e "discovery.type=single-node" docker.elastic.co/elasticsearch/elasticsearch:8.16.1

# 2. KibanaのDockerコンテナをpullして起動する。
docker pull docker.elastic.co/kibana/kibana:8.16.1
docker run --name kib01 --net elastic -p 5601:5601 docker.elastic.co/kibana/kibana:8.16.1
```  
  
https://www.elastic.co/guide/en/elasticsearch/reference/current/docker.html#_pull_the_elasticsearch_docker_image  
  
## ElasticSearchのAPIを叩いてみる   
  
KibanaからElasticSearchのAPIを叩き、色々試して見ます。  
Kibanaには<http://localhost:5601>からアクセスできるはずです、  
アクセス時にトークン情報やユーザID/PWを要求されるので、下記を参考に情報を取得してください。  
  
```sh
# Kibanaアクセス時にアクセス時に要求される登録トークンを取得(再生成)する
docker exec -it es01 /usr/share/elasticsearch/bin/elasticsearch-create-enrollment-token -s kibana
# Kibanaのelasticユーザのパスワードを取得(再生成)する
docker exec -it es01 /usr/share/elasticsearch/bin/elasticsearch-reset-password -u elastic
```  
  
Kibanaにアクセスできたら、`Management > Dev Tools`を開くきます。  
  
![](https://storage.googleapis.com/zenn-user-upload/3f06ba30e266-20241208.png)  
  
-------  
  
### IndexとDocumentの作成  
  
`PUT /$index名`と`POST /$index名/_doc`を実行するとindexとdocumentが作成されます。  
  
indexは`POST /$index名/_doc`時に存在しないと自動で作成されるため、`PUT /$index名`は実行しなくても問題ありません。  
  
**リクエスト**  
```sh
# indexの作成 (なくてもよい)
# PUT /$index名
PUT /my-index
# documentの作成
# POST /$index名/_doc
# bodyに保存したいjsonを含める
POST /my-index/_doc
{
    "id": "park_rocky-mountain",
    "title": "Rocky Mountain",
    "description": "Bisected north to south by the Continental Divide, this portion of the Rockies has ecosystems varying from over 150 riparian lakes to montane and subalpine forests to treeless alpine tundra."
}
```  
  
作成されたdocumentには自動的に_idフィールド(document_id)が付与されます.  
  
<details><summary>リスポンス</summary>  
  
```sh
{
  "_index": "my-index",
  "_id": "eq02ppMBaR5nPaZzuWLZ",
  "_version": 1,
  "result": "created",
  "_shards": {
    "total": 2,
    "successful": 1,
    "failed": 0
  },
  "_seq_no": 0,
  "_primary_term": 1
}
```  
![](https://storage.googleapis.com/zenn-user-upload/3d519a9b0727-20241208.png)  
  
</details>  
  
Documentの作成は`Bulk API`を使用することで、複数同時に実行することもできます。  
  
**リクエスト**  
```sh
POST /_bulk
{ "index": { "_index": "my-index" } }
{ "title": "Rocky Mountain", "description": "Bisected north to south by the Continental Divide, this portion of the Rockies has ecosystems varying from over 150 riparian lakes to montane and subalpine forests to treeless alpine tundra." }
{ "index": { "_index": "my-index" } }
{ "title": "Appalachian Mountains", "description": "Stretching from Alabama to Canada, these mountains are among the oldest in the world, characterized by their gentle rolling slopes." }
{ "index": { "_index": "my-index" } }
{ "title": "Himalayas", "description": "Home to the tallest mountains on Earth, including Mount Everest, the Himalayas span five countries in South Asia." }
```  
  
<details><summary>リスポンス</summary>  
  
```sh
{
  "errors": false,
  "took": 0,
  "items": [
    {
      "index": {
        "_index": "my-index",
        "_id": "fq2kppMBaR5nPaZzPmJL",
        "_version": 1,
        "result": "created",
        "_shards": {
          "total": 2,
          "successful": 1,
          "failed": 0
        },
        "_seq_no": 1,
        "_primary_term": 1,
        "status": 201
      }
    },
    {
      "index": {
        "_index": "my-index",
        "_id": "f62kppMBaR5nPaZzPmJL",
        "_version": 1,
        "result": "created",
        "_shards": {
          "total": 2,
          "successful": 1,
          "failed": 0
        },
        "_seq_no": 2,
        "_primary_term": 1,
        "status": 201
      }
    },
    {
      "index": {
        "_index": "my-index",
        "_id": "gK2kppMBaR5nPaZzPmJL",
        "_version": 1,
        "result": "created",
        "_shards": {
          "total": 2,
          "successful": 1,
          "failed": 0
        },
        "_seq_no": 3,
        "_primary_term": 1,
        "status": 201
      }
    }
  ]
}
```  
![](https://storage.googleapis.com/zenn-user-upload/8a64f4769092-20241208.png)  
  
</details>  
  
-------  
  
### Documentの検索  
  
documentの検索は以下のように行います。  
matchクエリは指定した単語があるドキュメントを検索します。  
  
```sh
# documentの検索
# GET /$index名/_seatch
# { query: { { $検索メソッド : $documentのフィールド : $値 } }
GET /my-index/_search
{
    "query": {
        "match": {
          "title": "Appalachian"
        }
    }
}
```  
  
<details><summary>リスポンス</summary>  
  
```sh
{
  "took": 1,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 1,
      "relation": "eq"
    },
    "max_score": 1.1374959,
    "hits": [
      {
        "_index": "my-index",
        "_id": "f62kppMBaR5nPaZzPmJL",
        "_score": 1.1374959,
        "_source": {
          "title": "Appalachian Mountains",
          "description": "Stretching from Alabama to Canada, these mountains are among the oldest in the world, characterized by their gentle rolling slopes."
        }
      }
    ]
  }
}
```  
![](https://storage.googleapis.com/zenn-user-upload/48625ce276d6-20241208.png)  
  
</details>  
  
また、matchクエリは以下のように、完全一致しない検索ワードでも、ドキュメントを検索できます。  
  
```sh
# documentの検索
GET /my-index/_search
{
    "query": {
        "match": {
          "title": "Appalachian is not high"
        }
    }
}
```  
  
<details><summary>リスポンス</summary>  
  
```sh
{
  "took": 3,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 1,
      "relation": "eq"
    },
    "max_score": 1.1374959,
    "hits": [
      {
        "_index": "my-index",
        "_id": "f62kppMBaR5nPaZzPmJL",
        "_score": 1.1374959,
        "_source": {
          "title": "Appalachian Mountains",
          "description": "Stretching from Alabama to Canada, these mountains are among the oldest in the world, characterized by their gentle rolling slopes."
        }
      }
    ]
  }
}
```  
![](https://storage.googleapis.com/zenn-user-upload/e56acabab8cd-20241208.png)  
</details>  
  
このような検索ができる理由は、ElasticSearchにより検索ワードが転置インデックスの検索に適した形に変換されているからです。  
  
アナライザーAPIを使うと、ElasticSearchがどのように検索ワードを解析しているのかが確認できます。  
  
```sh
# POST /$index名/_analyze
POST /my-index/_analyze
{
  "analyzer": "standard",
  "text": "Appalachian is not high"
}
```  
  
<details><summary>リスポンス</summary>  
  
```sh
{
  "tokens": [
    {
      "token": "appalachian",
      "start_offset": 0,
      "end_offset": 11,
      "type": "<ALPHANUM>",
      "position": 0
    },
    {
      "token": "is",
      "start_offset": 12,
      "end_offset": 14,
      "type": "<ALPHANUM>",
      "position": 1
    },
    {
      "token": "not",
      "start_offset": 15,
      "end_offset": 18,
      "type": "<ALPHANUM>",
      "position": 2
    },
    {
      "token": "high",
      "start_offset": 19,
      "end_offset": 23,
      "type": "<ALPHANUM>",
      "position": 3
    }
  ]
}
```  
  
![](https://storage.googleapis.com/zenn-user-upload/68b5fc242c40-20241208.png)  
</details>  
  
  
検索クエリにはmatch検索以外にも、termクエリ(完全一致)など、様々な方法があります。  
  
| **検索方法**       | **特徴**                                 | **使用例**                     |  
|--------------------|------------------------------------------|--------------------------------|  
| `match`            | 柔軟な部分一致検索                      | 自然言語の検索                 |  
| `term`             | 完全一致検索                            | キーワード型フィールドの検索   |  
| `match_phrase`     | フレーズの完全一致検索                  | 固有フレーズの検索             |  
| `bool` クエリ      | 複数条件の組み合わせ                   | 完全一致と部分一致の複合検索   |  
| `wildcard`         | ワイルドカードを使用した検索            | 部分パターンの検索             |  
| `prefix`           | 指定した接頭辞で始まる検索              | インクリメンタルサーチ         |  
| `regexp`           | 正規表現による検索                     | 複雑なパターン検索             |  
  
  
https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-match-query.html  
  
  
  
### Index/Documentの削除  
  
`document_id`を指定することで、特定のdocumentを削除できます。  
  
```sh
# DELETE /$index名/_doc/$document_id
DELETE /my-index/_doc/f62kppMBaR5nPaZzPmJL
```  
  
<details><summary>リスポンス</summary>  
  
```sh
{
  "_index": "my-index",
  "_id": "f62kppMBaR5nPaZzPmJL",
  "_version": 2,
  "result": "deleted",
  "_shards": {
    "total": 2,
    "successful": 1,
    "failed": 0
  },
  "_seq_no": 5,
  "_primary_term": 1
}
```  
  
![](https://storage.googleapis.com/zenn-user-upload/6e404acd1830-20241208.png)  
</details>  
  
indexはindex名を指定することで削除できます。  
  
```sh
# DELETE /$index名
DELETE /my-index
```  
  
<details><summary>リスポンス</summary>  
  
```sh
{
  "acknowledged": true
}
```  
  
![](https://storage.googleapis.com/zenn-user-upload/8efa4dfd5d61-20241208.png)  
  
</details>  
  
# まとめ  
  
本記事ではElasticSearchの概念と基本的な使い方を見ていきました。  
次の記事では、`Mapping`や`Aggregations`についてまとめていきます。  
  
# 参考  
  
https://www.elastic.co/guide/en/elasticsearch/reference/current/getting-started.html  
  
https://engineering.mercari.com/blog/entry/20220311-97aec2a2f8/  
  
https://qiita.com/Schott_man/items/4707973f0d6fe1d0a61f  
  
