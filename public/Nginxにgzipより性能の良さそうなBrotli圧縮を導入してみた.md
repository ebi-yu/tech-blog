# 概要  
  
- Nginxで今までデータをgzip圧縮形式で配信していた。  
- **Brotli**という高速圧縮術を用いた圧縮があったのでそれを設定してみた。  
  
# Brotliとは  
  
**Brotliは、Webサイトのファイルサイズを縮小するためにGoogleが開発したオープンソースの圧縮アルゴリズムです。Gzipと比較して、平均20％高い圧縮率を誇り、大きなファイル圧縮に最適化されています。﻿**  
  
Brotliは、HTMLやJavaScriptで頻出されるフレーズが含まれる静的辞書を用いて圧縮しています。  
圧縮レベルは0から11まで12段階が用意されており、数字が大きくなるほど圧縮率が高くなります。  
ただし、圧縮率が高くなるほどより多くの計算リソースを消費するため、ブラウザのレンダリングやCDN側の負荷が高くなる可能性があります。﻿  
  
- BrotliはGzipに比べて15%ほど圧縮できていることがわかりました。  
- BrotliはGzipと比較して、同じ速度でより小さなファイルに圧縮し、同じ速度で解凍します。  
- 元ファイルが大きければ大きいほどBrotliの効果が高くでています。  
- Brotliをレベル11で圧縮することで、ユーザーはファイルサイズを最高のGzip圧縮レベルと比較して19%小さくすることができます。  
- gzipはほとんどの場合Brotliに速度で勝るものの、圧縮するレベルによって結果が異なります。﻿  
  
# 導入方法  
  
## DockerFile  
  
```DockerFile
# ベースイメージを指定
FROM ubuntu:22.04 as builder

# 必要なパッケージをインストール
RUN apt update && apt install -y \
    libpcre3 libpcre3-dev zlib1g zlib1g-dev openssl libssl-dev wget git gcc make libbrotli-dev \
    && rm -rf /var/lib/apt/lists/*

# Nginxとngx_brotliモジュールのビルド
WORKDIR /app
RUN wget https://nginx.org/download/nginx-1.25.3.tar.gz \
    && tar -zxf nginx-1.25.3.tar.gz \
    && git clone --recurse-submodules https://github.com/google/ngx_brotli \
    && cd nginx-1.25.3 \
    && ./configure --with-compat --add-dynamic-module=../ngx_brotli \
    && make modules

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
  
**1.  準備**  
   - 必要な依存関係のインストール: Brotli圧縮をサポートするために、Nginxをビルドする前に、**libbrotli-dev**を含む依存パッケージをインストールする必要があります。  
  
```sh
RUN apt update && apt install -y \
    libpcre3 libpcre3-dev zlib1g zlib1g-dev openssl libssl-dev wget git gcc make libbrotli-dev \
    && rm -rf /var/lib/apt/lists/*
```  
  
**2.  Nginxのソースコードのダウンロード**  
   - Nginxの公式サイトから最新バージョン（または必要なバージョン）のソースコードをダウンロードします。  
  
```sh
RUN wget https://nginx.org/download/nginx-1.25.3.tar.gz \
    && tar -zxf nginx-1.25.3.tar.gz \
```  
  
**3.  ngx_brotliモジュールのクローン**  
   - Brotli圧縮機能を提供する**ngx_brotl**モジュールのソースコードをGitHubからクローンします。このモジュールは、Brotli圧縮と展開の両方をNginxで実現します。  
  
```sh
 git clone --recurse-submodules https://github.com/google/ngx_brotli \
```  
  
**4. ビルド設定の実行**  
   - Nginxのソースディレクトリに移動し、./configurスクリプトを実行してビルドを設定します。  
この際に、**--add-dynamic-module**オプションを使用して、ngx_brotliモジュールのパスを指定します。  
  
```sh
cd nginx-1.25.3 \
 ./configure --with-compat --add-dynamic-module=../ngx_brotli \
```  
  
**5. Nginxのビルドとモジュールのコンパイル**  
   - 設定に基づき、makコマンドを実行してNginxとngx_brotlモジュールをビルドします**make module**コマンドにより、指定したモジュールのみをコンパイルすることもできます。  
  
```sh
make modules
```  
  
## Nginx  
  
```conf
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
  
## 実際の挙動  
  
Content-Encodingがbrになっている。  
速度的にはちょっと速くなっているかも(微妙)  
  
![](https://storage.googleapis.com/zenn-user-upload/57e8b9706dde-20241121.png)  
  
# まとめ  
  
- どのアルゴリズムが最強というより、いずれも一長一短ある。  
- ファイルのサイズが大きくなるほど、Brotliの効果が出そう  
  
![](https://storage.googleapis.com/zenn-user-upload/ceacf5896bc7-20241121.png)  
  
※画像は以下より引用  
  
https://hidekatsu-izuno.hatenablog.com/entry/2016/07/06/003803  
