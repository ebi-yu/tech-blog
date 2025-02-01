# 概要  
  
- 今までnginxを使うときはgzip圧縮形式でデータを配信していた。  
- `Brotli`という高速圧縮術を用いた圧縮があったのでそれを試してみた。  
- 簡単に導入するだけなら、下記の`DockerImage`を使ってみても良いかも。  
  
https://hub.docker.com/r/kiweeteam/nginx-brotli  
  
## Brotliとは  
  
**Brotliは、Webサイトのファイルサイズを縮小するためにGoogleが開発したオープンソースの圧縮アルゴリズムです。Gzipと比較して、平均20％高い圧縮率を誇り、大きなファイル圧縮に最適化されています。﻿**  
  
圧縮レベルは0から11まで12段階が用意されており、数字が大きくなるほど圧縮率が高くなります。  
ただし、圧縮率が高くなるほどより多くの計算リソースを消費するため、ブラウザのレンダリングやCDN側の負荷が高くなる可能性があります。  
  
- BrotliはGzipに比べて15%ほど圧縮できていることがわかりました。  
- BrotliはGzipと比較して、同じ速度でより小さなファイルに圧縮し、同じ速度で解凍します。  
- 元ファイルが大きければ大きいほどBrotliの効果が高くでています。  
- Brotliをレベル11で圧縮することで、ユーザーはファイルサイズを最高のGzip圧縮レベルと比較して19%小さくすることができます。  
- gzipはほとんどの場合Brotliに速度で勝るものの、圧縮するレベルによって結果が異なります。﻿  
  
https://github.com/google/brotli  
  
## 導入方法  
  
Brotliは標準ではNginxにサポートされていないので、追加のモジュールを導入する必要があります。  
  
### DockerFile  
  
```DockerFile
# ベースイメージを指定
FROM ubuntu:22.04 as builder

# Brotli導入に必要なパッケージをインストール
RUN apt update && apt install -y \
    libpcre3 libpcre3-dev zlib1g zlib1g-dev openssl libssl-dev wget git gcc make libbrotli-dev \
    && rm -rf /var/lib/apt/lists/*
    
WORKDIR /app

# Nginxとngx_brotliモジュールのビルド
RUN wget https://nginx.org/download/nginx-1.25.3.tar.gz \
    && tar -zxf nginx-1.25.3.tar.gz 
RUN git clone --recurse-submodules https://github.com/google/ngx_brotli 
RUN nginx-1.25.3 \
    && ./configure --with-compat --add-dynamic-module=../ngx_brotli 
RUN make modules

# Nginxのベースイメージを使用
FROM nginx:1.25.3

# ビルドしたモジュールをコピー
COPY --from=builder /app/nginx-1.25.3/objs/ngx_http_brotli_static_module.so /etc/nginx/modules/
COPY --from=builder /app/nginx-1.25.3/objs/ngx_http_brotli_filter_module.so /etc/nginx/modules/

# Brotliの設定を追加
RUN { \
    echo 'brotli on;'; \
    echo 'brotli_comp_level 6;'; \
    echo 'brotli_static on;'; \
    echo 'brotli_types application/atom+xml application/javascript application/json application/rss+xml'; \
    echo '          application/vnd.ms-fontobject application/x-font-opentype application/x-font-truetype'; \
    echo '          application/x-font-ttf application/x-javascript application/xhtml+xml application/xml'; \
    echo '          font/eot font/opentype font/otf font/truetype image/svg+xml image/vnd.microsoft.icon'; \
    echo '          image/x-icon image/x-win-bitmap text/css text/javascript text/plain text/xml;'; \
    } > /etc/nginx/conf.d/brotli.conf
```  
  
**1. 準備**  
  
Brotli圧縮をサポートするために、Nginxをビルドする前に、**libbrotli-dev**を含む依存パッケージをインストールする必要があります。  
  
```DockerFile
# Brotli導入に必要なパッケージをインストール
RUN apt update && apt install -y \
    libpcre3 libpcre3-dev zlib1g zlib1g-dev openssl libssl-dev wget git gcc make libbrotli-dev \
    && rm -rf /var/lib/apt/lists/*
```  
  
**2.  Nginxのソースコードのダウンロード**  
   - Nginxの公式サイトから最新バージョン（または必要なバージョン）のソースコードをダウンロードします。  
  
```DockerFile
RUN wget https://nginx.org/download/nginx-1.25.3.tar.gz \
    && tar -zxf nginx-1.25.3.tar.gz \
```  
  
**3.  ngx_brotliモジュールのクローン**  
   - Brotli圧縮機能を提供する**ngx_brotl**モジュールのソースコードをGitHubからクローンします。  
   - このモジュールは、Brotli圧縮と展開の両方をNginxで実現します。  
  
```DockerFile
RUN git clone --recurse-submodules https://github.com/google/ngx_brotli \
```  
  
**4. ビルド設定の実行**  
   - Nginxのソースディレクトリに移動し、./configurスクリプトを実行してビルドを設定します。  
   - この際に、**--add-dynamic-module**オプションを使用して、ngx_brotliモジュールのパスを指定します。  
  
```DockerFile
RUN cd nginx-1.25.3 \
 ./configure --with-compat --add-dynamic-module=../ngx_brotli \
```  
  
**5. Nginxのビルドとモジュールのコンパイル**  
   - 設定に基づき、makコマンドを実行してNginxとngx_brotlモジュールをビルドします**make module**コマンドにより、指定したモジュールのみをコンパイルすることもできます。  
  
```DockerFile
RUN make modules
```  
  
-----------  
  
### Nginx  
  
```Nginx
user  nginx;
worker_processes auto;

pid        /var/run/nginx.pid;

load_module modules/ngx_http_brotli_filter_module.so;
load_module modules/ngx_http_brotli_static_module.so;

http {
       ~~ 省略 ~~ 
        brotli on;
        brotli_types text/css application/javascript application/json application/font-woff application/font-tff image/gif image/png image/jpeg application/octet-stream;
    }
}
```  
  
**1. モジュールの読み込み**  
- Brotli圧縮を提供するための追加モジュールを読み込むことを指示します。  
- ngx_http_brotli_filter_moduleは動的なコンテンツの圧縮に使用されます。  
- ngx_http_brotli_static_moduleは事前に圧縮された静的コンテンツの配信に使用されます。  
  
```Nginx
load_module modules/ngx_http_brotli_filter_module.so;
load_module modules/ngx_http_brotli_static_module.so;
```  
  
**2.Brotli圧縮の設定**  
  
- brotli on : Brotli圧縮を有効にします  
- brotli_types ... : Brotli圧縮を適用するMIMEタイプを指定します。  
  
```Nginx
brotli on;
brotli_types text/css application/javascript application/json application/font-woff application/font-tff image/gif image/png image/jpeg application/octet-stream;
```  
  
  
## 実際の挙動  
  
- Content-Encodingがbrになっている。  
- 大きいファイルサイズの配信をしないと、gzipとの速度差は感じない。  
  
![clip-20240201183957.png](/image/da5b0400-0e85-9fd7-184a-2951161c6966.png)  
  
https://admin.dev.lakeelcloud.com/visual-mosaic-microfrontends/ebi-test-3  
  
## まとめ  
  
- Brotliは大容量ファイルの配信に向いている圧縮技術  
- ファイルサイズが大きくない場合は、gzipとそんなに差がなさそう  
  
  
## 参考  
  
https://hidekatsu-izuno.hatenablog.com/entry/2016/07/06/003803  
