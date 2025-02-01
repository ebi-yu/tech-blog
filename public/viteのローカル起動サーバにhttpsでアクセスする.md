開発において、viteのローカル起動サーバにhttpsで接続したいケースがあったので、その方法を調べました。  
  
## 方法1. オレオレ証明書を発行し、設定する  
  
[mkcert](https://github.com/FiloSottile/mkcert)などでlocalhost用の証明書を発行し、それをvite.configに適用します。  
  
https://github.com/FiloSottile/mkcert  
  
### 1. 証明書発行  
  
#### macの場合  
  
brewを使うのが楽そう。  
  
```sh
$ brew install mkcert
$ mkcert install
$ mkcert localhost
```  
  
#### windowsの場合  
  
chocoを使うのが楽そう。  
  
```sh
$ choco install mkcert
$ mkcert install
$ mkcert localhost
```  
  
chocoのinstallはこちらから  
https://chocolatey.org/install  
  
### 2. 証明書設定  
  
vite.config.tsに以下のように設定します。  
(証明書が`${プロジェクト直下}/cert`にあることを想定してます)  
  
```ts
import fs from 'fs';

export default defineConfig(() => {
        server: {
            https: {
                key: fs.readFileSync('./cert/localhost-key.pem'),
                cert: fs.readFileSync('./cert/localhost.pem'),
            },
        }
}
```  
  
## 方法2 vite-plugin-basic-sslを使う  
  
[vite-plugin-basic-ssl](https://github.com/vitejs/vite-plugin-basic-ssl)  
  
pluginを設定するだけで、https接続できます。  
ただ、公式では推奨されていなく、ブラウザが警告を出すので、方法1の方が良さそうです。  
  
> 基本的なセットアップでは、自己署名証明書を自動的に作成しキャッシュする   
> @vitejs/plugin-basic-ssl をプロジェクトのプラグインに追加することもできます。  
> しかし、独自の証明書を作成することを推奨します。  
  
[引用](https://ja.vitejs.dev/config/server-options.html#server-https)  
  
### 設定方法  
  
pluginに設定するだけです。  
  
```sh 
$ pnpm install @vitejs/plugin-basic-ssl -D
```  
  
```ts
// vite.config.js
import basicSsl from '@vitejs/plugin-basic-ssl'

export default defineConfig(() => {
  plugins: [
    basicSsl()
  ]
}
```  
  
## まとめ  
  
手間もそんなにかからないので、証明書を設定してやる方法を使うのが良さそう。  
httpsの設定はviteのサーバオプションなので、本番viteサーバには影響はなさそうでした。  
