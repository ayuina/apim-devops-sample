# GitHub Actions を使用した DevOps 編

さて一連の作業の流れが確認出来たので、ここからは Github Actions を使用して自動化していきます。

## ブランチと環境の相互作用

Resource Kit ではリポジトリのフォークを使用した開発フローを想定しているようでしたが、本サンプルでは少し簡単に単一リポジトリ内でのブランチで制御していきたいと思います。

![api management devops flow](./images/apim-devops-flow.png)

ワークフローとして自動化する内容とイベントを整理していくと以下のようになるでしょうか。

|イベント|開発者の作業|GitHub Actions トリガー|GitHub Actions ワークフロー|
|---|---|---|---|
|開発作業の開始|手動実行|workflow_dispatch|開発環境の API Management のバックアップを行う|
|本番環境へ移送|main ブランチへのプルリクエスト|pull_request|本番環境の API Management のバックアップを行い、対象のリビジョンをデプロイする|
|本番稼働|main ブランチへのマージ|push|本番環境の API Management のリビジョンを切り替える|


## Github Actions から Azure に接続する

前述の様な GitHub Actions ワークフローでは Azure 環境を操作することになりますので、Azure Active Directory に対してサービスプリンシパルとしてログインできる必要があります。
本サンプルではフェデレーション ID 資格情報を使用していますので、以下の設定が必要です。

- Azure Active Directory にアプリケーションを登録する
- 登録したアプリケーションが GitHub Repository を信頼するように設定する
- Azure サブスクリプションに対して必要な権限を持つ共同作成者ロールなどに、登録したアプリケーションを割り当てる
- GitHub Actions Secret に Azure AD テナント、Azure AD アプリケーション、Azure サブスクリプションに ID を登録する

![github actions federation](./images/github-actions-federation.png)

設定方法の詳細は下記をご参照ください。

- [GitHub リポジトリを信頼するようにアプリを構成する (プレビュー)](https://docs.microsoft.com/ja-jp/azure/active-directory/develop/workload-identity-federation)
- [GitHub Actions を使用して Azure に接続する](https://docs.microsoft.com/ja-jp/azure/developer/github/connect-from-azure?tabs=azure-portal%2Clinux)」


## 開発作業の開始

Github Actions を使用したワークフローは[こちら](./.github/workflows/backup-apim.yml)にサンプルが置いてあります。
基本的に PowerShell 編で紹介したスクリプトファイルを実行するだけですね。
このワークフローの手動実行時に GitHub Environment を指定できるようになっており、選択した Environment 名を含むパラメータファイルを読み込むようになっています。

![backup-apim-workflow](./images/backup-apim-workflow.png)

このワークフローを実行するためには事前に下記の設定が必要です。

- GitHub Environment （```dev``` および ```prod```）の追加
- Azure AD アプリケーションの信頼するエンティティとして ```dev``` 環境を追加
- API Management に対して必要な権限を持つ共同作成者などのロールを Azure AD アプリケーションを割り当てる

## 本番環境へ移送

Github Actions を使用したワークフローは[こちら](./.github/workflows/deploy-api-production.yml)にサンプルが置いてあります。
こちらは PowerShell 編で紹介したスクリプトの内容を YAML の中にインラインスクリプトで実装しています。


## 本番稼働