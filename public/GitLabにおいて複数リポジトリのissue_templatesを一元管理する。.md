# はじめに  
  
複数のプロジェクトを管理していると、各リポジトリで異なる `issue_template` を管理するのは手間がかかります。  
  
そこで、CI/CDを用いて自動的に1つのマスタリポジトリから複数のリポジトリに共通のテンプレートを自動で展開するようにしたので紹介します。  
  
## issue_templates とは  
  
`issue_templates` は、GitLabで issue を作成する際に利用できるテンプレートです。  
  
バグ報告や新機能提案の際、統一されたフォーマットでの入力を促すことで、開発チームの効率が上がります。  
  
GitLabでは、リポジトリ内に `.gitlab/issue_templates/` ディレクトリを作成し、そこにテンプレートファイルを配置することでテンプレートを利用できます。  
  
使用イメージは公式の[GitLab Issue Templates](https://gitlab.com/gitlab-org/gitlab/-/tree/master/.gitlab/issue_templates)を参考にしてみてください。  
  
https://gitlab-docs.creationline.com/ee/user/project/description_templates.html  
  
## .gitLab-ci.yml とは  
  
`.gitLab-ci.yml`はGitLab CI/CDを実行するために必要な実行設定ファイルです。  
  
`.gitLab-ci.yml`はリポジトリに一つ配置し、  
  
- CI/CDパイプラインの定義  
- CI/CDによって実行するスクリプト  
- CI/CD実行時に渡される引数や依存権系  
- CI/CDを実行する環境(Dockerイメージ)  
   
を定義します。  
  
https://gitlab-docs.creationline.com/ee/ci/yaml/gitlab_ci_yaml.html  
  
# 処理の概要  
  
やっていることは単純で、マスタリポジトリで管理している `issue_templates` をCI/CDで各リポジトリに配っているだけです。  
  
1. **マスタリポジトリでテンプレートを管理**  
   1つのマスタリポジトリに、各プロジェクトで共通して使用する `issue_templates` を集約します  
  
2. **自動的に各リポジトリのCI/CDをキック**  
   マスタリポジトリでテンプレートに変更が加わると、各リポジトリのCI/CDを自動でトリガーします。このために、マスタリポジトリの `.gitlab-ci.yml` を設定し、複数のリポジトリへ通知します  
  
3. **各リポジトリでCI/CDパイプラインを実行し、マスタリポジトリから最新テンプレートを取得**  
   各リポジトリはCI/CDを通じて、マスタリポジトリから最新のテンプレートを取ってきて、自身のリポジトリに反映させます  
  
# 実装  
  
以下実際の実装について説明します。  
  
## 1. マスタリポジトリに`issue_template`を配置  
  
`.gitlab/issue_templates`だけを管理するリポジトリを作成し、そこに使用したいissue_templateを配置します。  
  
## 2. 各リポジトリに `.gitlab-ci.yml` を追加  
  
共通テンプレートを使用する各リポジトリに以下の `.gitlab-ci.yml` を追加します。  
  
これにより、CI/CDが実行されるたびにマスタリポジトリのテンプレートが各リポジトリに反映されます。  
  
```yml
stages:
  - update_issue_templates

update_issue_templates:
  stage: update_issue_templates
  image: {{使用するDockerイメージを指定}}
  script:
    - git checkout master
    - rm -rf .gitlab/issue_templates
    - mkdir -p .gitlab/issue_templates
    - git clone {{マスタリポジトリのGitLab URL}}
    - git add .gitlab/issue_templates
    - git commit -m "[CI] Update issue templates"
    - git push origin master
  tags:
    - docker
```  
  
- `git checkout master`でリポジトリの `master` ブランチに切り替えます  
- `.gitlab/issue_templates` ディレクトリを削除し、マスタリポジトリから `git clone` でマスタリポジトリの最新のテンプレートを取得します  
- `git add` で変更をステージングし、コミットしてリポジトリにプッシュします  
  
## 2. マスタリポジトリに `.gitlab-ci.yml` を追加  
  
マスタリポジトリに `.gitlab-ci.yml` を追加します。  
  
これにより、マスタリポジトリに変更が加わった際、自動的に各リポジトリのCI/CDパイプラインがキックされます。  
  
そして、[前項](##-2.各リポジトリに.gitlab-ci.ymlを追加)で作成した`.gitlab-ci.yml`に定義されたジョブが実行され、テンプレートが更新されます。  
  
```yml
stages:
  - notify_repos

notify_repos:
  stage: notify_repos
  script:
  # 新規追加方法 : URLのidとtokenを変えて追加
  # token = gitlab > setting >  ci/cd > Pipeline trigger tokens
  # projects/&{id} = gitlab > setting > general
   - >
      curl --request POST \
      "https://gitlab.com/api/v4/projects/${Project ID}/trigger/pipeline?token=${TRIGGER_TOKEN}&ref=master"
```  
  
##### Project ID  
  
トリガーする対象のプロジェクトIDです。  
  
共通テンプレートを使用するプロジェクトのページの「Setting > General」から確認します  
  
![スクリーンショット 2024-10-20 13.07.55.png](/image/cdca47ec-88d9-7eca-9746-0122ea6b9187.png)  
  
  
##### TRIGGER_TOKEN  
  
CI/CDパイプラインをトリガーして実行するためのトークンです。  
  
共通テンプレートを使用するプロジェクトの「Settings > CI / CD > Pipeline triggers」でトークンを生成し、指定します。  
  
![スクリーンショット 2024-10-20 13.17.13.png](/image/54732268-0dc4-b133-da46-771bb38075ee.png)  
  
https://gitlab-docs.creationline.com/ee/ci/triggers/  
  
# 最後に  
  
今回紹介した仕組みを導入することで、1つのマスタリポジトリで issue_templates を一括管理し、複数のリポジトリに自動で展開することが可能になります。  
  
これにより、テンプレート管理の手間が大幅に削減され、各プロジェクトで統一されたフォーマットを維持できます。  
  
質問やフィードバックがあれば、ぜひコメントをお寄せください！  
