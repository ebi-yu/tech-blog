# 概要  
  
ウェブサイトの制作や運営をしていると、画像ファイルの管理が面倒になることがあります。  
  
そこで、CSSに画像を埋め込んで、HTML側でクラス指定をすることで、画像の管理を簡略化する方法を紹介します  
  
この方法を使えば、画像ファイルを外部に保存する必要がなく、CSSファイル内に直接画像データを埋め込むことができます  
  
# 手順  
  
## ⓪ 事前準備  
  
CSSには基本的にsvg形式の画像しか埋め込めないで他形式の画像は  
[Image Magic](https://imagemagick.org/index.php)で、svg形式に変換しておきます。  
  
以下のコマンドでPNGファイルをSVGに変換します。  
  
```sh
convert input.png output.svg
```  
  
https://imagemagick.org/script/download.php  
  
  
## ① svg画像をcss形式に置き換え  
  
下記サイトにsvgのコードを貼って、css形式に変換します。  
  
https://yoksel.github.io/url-encoder/  
  
![スクリーンショット 2024-10-17 22.14.36.png](/image/fcd0b816-37d6-a016-0a0b-73a078802b1f.png)  
  
## ② cssにsvg画像のクラスを作成する。  
  
cssのクラスに先ほどコピーしたcssを貼り付けます。  
  
```css
.svg-search-icon {
  background: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' height='100px' viewBox='0 -960 960 960' width='100px' fill='%23000000'%3E%3Cpath d='M784-120 532-372q-30 24-69 38t-83 14q-109 0-184.5-75.5T120-580q0-109 75.5-184.5T380-840q109 0 184.5 75.5T640-580q0 44-14 83t-38 69l252 252-56 56ZM380-400q75 0 127.5-52.5T560-580q0-75-52.5-127.5T380-760q-75 0-127.5 52.5T200-580q0 75 52.5 127.5T380-400Z' /%3E%3C/svg%3E");
  width: 100px;
  height: 100px;
  background-size: cover;
}
```  
  
- `background-size: cover`を設定しておくと、svg画像が要素全体に表示されます  
- `background`または`background-image`どっちを使っても良いです  
  
#### htmlで読み込むとこんな感じ  
  
```html
<div class="svg-search-icon"></div>
```  
  
![スクリーンショット 2024-10-17 22.12.28.png](/image/30258d5d-caa2-20d2-452a-b419bcfba4f2.png)  
  
# サンプルコード  
  
以下にサンプルのhtmlとcssを置いておきます。  
  
https://github.com/ebi-yu/playground/tree/main/qiita/css-svg  
  
# 追記 : 2024-10-17  
  
`background`を使ってsvg画像を読み込むと、svg画像の色の変更ができません。  
`mask`を使うと`、background-color`を用いて画像の色を変更できるようになります。  
  
```css
.svg-search-icon {
  mask: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' height='100px' viewBox='0 -960 960 960' width='100px' fill='%23000000'%3E%3Cpath d='M784-120 532-372q-30 24-69 38t-83 14q-109 0-184.5-75.5T120-580q0-109 75.5-184.5T380-840q109 0 184.5 75.5T640-580q0 44-14 83t-38 69l252 252-56 56ZM380-400q75 0 127.5-52.5T560-580q0-75-52.5-127.5T380-760q-75 0-127.5 52.5T200-580q0 75 52.5 127.5T380-400Z' /%3E%3C/svg%3E");
  width: 100px;
  height: 100px;
  background-size: cover;
}
```  
  
#### htmlで読み込むとこんな感じ  
  
```html
<!-- 後から画像の色を指定 -->
<div style="background-color: red;" class="svg-search-icon-mask"></div>
```  
  
![スクリーンショット 2024-10-17 22.26.43.png](/image/4f268450-fd30-9f23-0a46-4b3da42df770.png)  
