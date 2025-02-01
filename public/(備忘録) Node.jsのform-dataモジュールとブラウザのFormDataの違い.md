安易にAPIのform-dataを消したら、エラーが起きたので、備忘録的に残します。  
  
## 概要  
  
- `FormData`はブラウザで使い、Blob型でデータを渡す  
  - Blobはファイルやバイナリデータをブラウザ側で操作する際の標準的なオブジェクト形式  
- `form-data`はサーバサイドで使い、buffer型でデータを渡す  
  - BufferはNode.jsでバイナリデータを操作するための標準的な形式  
- `form-data`は明示的にインポートする必要があるが、`FormData`は標準で使えるので、インポートする必要なし  
  
## FormDataとform-dataの違い  
  
| 特徴                 | form-data (Node.js)                                                | FormData (ブラウザ)                                      |  
|----------------------|--------------------------------------------------------------------|----------------------------------------------------------|  
| 利用環境             | Node.jsサーバーサイド専用                                          | ブラウザ環境                                              |  
| サポートデータ型     | Buffer、ファイルストリーム、通常のテキスト                         | Blob、Fileオブジェクト、テキスト                          |  
| getHeadersメソッド   | あり（ヘッダー生成が自動）                                        | なし（fetchやXHRが自動で生成）                             |  
| 典型的な使用場面     | サーバーから別のAPIへのファイルアップロード                         | ブラウザからサーバーへのフォームデータ送信                 |  
| 明示的なインポート   | 必要 | 必要なし                         |  
  
## 使用例  
  
### Node.js (form-dataモジュール)  
  
  ```ts  
// 明示的にimport  
import FormData from 'form-data';  
import fs from 'fs';  
  
const fileData = new FormData();  
fileData.append('file', fs.createReadStream('/path/to/file.txt'), 'file.txt');  
  
// リクエストに必要なヘッダーを取得  
const headers = fileData.getHeaders();  
  
axios.post('<https://example.com/upload>', fileData, { headers });  
  ```  
  
###  ブラウザ (FormData)  
  
```ts
// ブラウザのFile型 (Blobのサブクラス)
const fileInput = document.querySelector('input[type="file"]');
const formData = new FormData();
formData.append('file', fileInput.files[0]);

fetch('https://example.com/upload', {
 method: 'POST',
 body: formData
})
.then(response => response.json())
.then(data => console.log(data));
```  
  
## 参考  
  
https://developer.mozilla.org/en-US/docs/Web/API/FormData/FormData  
  
https://www.npmjs.com/package/form-data  
