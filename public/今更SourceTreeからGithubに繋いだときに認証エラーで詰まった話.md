# 概要  
  
久しぶりにSourceTreeからGitHubのリポジトリに変更を`push`しようとした際、エラーが発生して少し詰まったので、備忘録的に解決方法を共有します。  
  
# 解決方法まとめ  
  
- **SourceTree**からGitHubに接続するために[Personal Access Tokens](https://github.com/settings/tokens?type=beta)が必要  
- SourceTreeの`Advanced`設定から、**事前にGitHubアカウント情報を削除**  
- **Personal Access Token**発行時の推奨権限：  
  - **Contents**：Read and write  
  - **Commit Statuses**：Read and write  
  - **Pull Requests permissions**：Read and write  
  - **Workflows permissions**：Read and write  
  
# 背景  
  
SourceTreeからGitHubへの`push`を試みた際に、**認証エラー**が発生し接続ができなくなりました。  
  
- **SourceTree**経由の`pull`は成功  
- コマンドライン上での`git push`も正常  
- しかし、**SourceTreeからの`push`だけ失敗**  
  
# 原因  
  
GitHubではセキュリティ向上のため、**パスワードでの認証を廃止**し、**Personal Access Token**の使用が必須になっているみたいです。(多分結構前から、、、)  
  
**Personal Access Token**の発行時には適切な権限を設定する必要があります。  
  
# 対応手順  
  
## 1. 古い認証情報をSourceTreeから削除する。  
  
`SourceTree > Settings > Advanced`から、GitHubの**アカウント情報を削除**します。  
  
**日本語設定**の場合、「`Advanced`」ボタンが押せないことがあるため、その場合は**言語を英語**に変更します。  
  
![スクリーンショット 2024-10-14 13.22.21.png](/image/2d6b86aa-711a-e7f5-0b2e-83a26ade957d.png)  
  
## 2. Personal Access Tokens発行画面にアクセス  
  
[GithubのPersonal Access Tokens発行画面](https://github.com/settings/tokens?type=beta)にアクセスし、`Fine-grained Tokens`の`Generate new token`を選びます。  
  
![スクリーンショット 2024-10-14 13.28.59.png](/image/41a9f6e7-5293-7acb-2d29-f127333c53ad.png)  
  
## 3. Personal Access Tokensの権限を選択  
  
`Repository Access`を`All repositories`に設定します。  
その後、`Permission > Repository permissions`から下記の権限を選択します。  
  
| 名前 | レベル | 概要 |  
| ---- | ---- | ---- |  
| Contents | Read and write | git clone, git pull, git push に必要|  
| Metadata |Read-only | リポジトリメタデータの取得に必要（git ls-remote など）|  
| Pull requests |Read and write  | プルリクの作成・操作に必要 |  
| Workflows permissions |Read and write  | `git push`でGitHub Actionsをトリガーする場合必要 |  
  
![スクリーンショット 2024-10-14 13.38.28.png](/image/d36384be-5976-0943-fd39-9bba632b4885.png)  
![スクリーンショット 2024-10-14 13.39.00.png](/image/365c69ab-e231-046c-9f58-3a478e9f58fc.png)  
![スクリーンショット 2024-10-14 13.40.35.png](/image/60220c08-2107-69c3-6728-ee9a35da58ee.png)  
![スクリーンショット 2024-10-14 13.40.58.png](/image/89a8b2ac-b536-235f-a235-186c918f8166.png)  
  
## 3. 発行したトークンをSourceTreeに設定  
  
SourceTreeで`push`を行った際に、認証を求められるので、先ほど、発行したトークンで認証を行ってください。  
  
![スクリーンショット 2024-10-14 13.42.44.png](/image/8eeefcda-7a30-8719-110f-99d2073a362c.png)  
  
# FAQ  
  
## `git pull`時に403エラーがでる。  
  
権限設定がたりていません。  
  
[3. Personal Access Tokensの権限を選択](https://qiita.com/Nicasdream0/items/a9f20dc4e3dd880bc3d4#3-personal-access-tokens%E3%81%AE%E6%A8%A9%E9%99%90%E3%82%92%E9%81%B8%E6%8A%9E)を見直してください。  
  
## `git pull`時に400エラーがでる。  
  
`Personal access tokens (classic)`を使っていた場合、400エラーがでる場合があります。  
  
トークン発行時に`Tokens (classic)`ではなく、`Fine-grained Toknes`を選択してください。  
  
# 感想  
  
エンジニア歴五年目でこんなところに躓くとは思わなかった、、、  
  
  
  
  
  
  
  
