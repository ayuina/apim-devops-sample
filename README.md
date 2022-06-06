# API Management の DevOps サンプル

Azure API Management には
[バージョン](https://docs.microsoft.com/ja-jp/azure/api-management/api-management-versions)
と
[リビジョン](https://docs.microsoft.com/ja-jp/azure/api-management/api-management-revisions)
という変更を安全に適用するための機能が備わっています。
ただ実際に API Management やこれらの機能を触ってみると分かると思うのですが、割とあっさり API が壊れます。
新しいリビジョンを作成してから修正を加えるとか、新しいバージョンを作成してから変更するという作業をやり忘れるからです。

もちろんこれは私の不注意に起因する問題ではあるのですが、これが本番環境だったらと思うと割と夜も眠れません。
人間は間違う生き物なので、その注意深さに依存したプロセスには限界があり、もう少し安全な仕組みが必要です。
というわけで DevOps というか CI/CD パイプラインが必要です。
そのやり方は [Docs にも紹介されている](https://docs.microsoft.com/ja-jp/azure/api-management/devops-api-development-templates) のですが、
若干とっつきにくかったこと、日本語の情報が少なかったこと、GitHub Actions でやってみたかったなどの理由から、色々試してみたものを残しておこうと思います。

## 本レポジトリの内容

ここでは [Azure API Management DevOps resource kit](https://github.com/Azure/azure-api-management-devops-resource-kit) を使用して GitHub Actions による API Management を CI/CD するために検証した内容をサンプルとして公開しています。
ただ内容がわかりにくいと思うので、解説資料として以下の３つを作成しています。

- API Management の CI/CD に関わる[基礎知識編](./README-ApiManagement.md)
- Resource Kit を使用した一連の作業を手動で試す [PowerShell 編](./README-PowerShell.md)
- PowerShell で試した内容を自動化していく [GitHub Actions 編](./README-GitHubActions.md)

解説資料と同時に実際に動作するスクリプトや Github Actions ワークフローなども格納しています。
利用する Azure サービスのインスタンス名などは parameter ファイル等に記載しており、各スクリプトはそれを読み込んで動作しています。
実際に動かしてみたい方は本リポジトリを fork するなりコピーするなりして、適宜書き換えてご利用いただければと思います。

## 注意事項

本リポジトリの内容はあくまでも DevOps Resource Kit の利用方法を「サンプル」として公開しているものです。
参考にしていただいた結果まで責任を負うことは出来ませんので、その旨ご承知ください。

## お願い

なお資料に作成時点で Resource Kit は 1.0.0-beta.5 というバージョンが最新で、いろいろ Issue も残っている状態です。
個人的にも今後の開発の進捗に期待しているところですが、もし Resource Kit にご興味のある方は Feedback なり開発への参加などもご検討いただければと思います。


