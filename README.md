# qiita-content

[![textlint & reviewdog](https://github.com/takiguchi-yu/qiita-content/actions/workflows/textlint.yml/badge.svg)](https://github.com/takiguchi-yu/qiita-content/actions/workflows/textlint.yml) [![Publish articles](https://github.com/takiguchi-yu/qiita-content/actions/workflows/publish.yml/badge.svg)](https://github.com/takiguchi-yu/qiita-content/actions/workflows/publish.yml)

<!-- @import "[TOC]" {cmd="toc" depthFrom=2 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [ローカル環境のセットアップ](#ローカル環境のセットアップ)
- [記事の作成](#記事の作成)
- [記事の削除](#記事の削除)
- [文章校正について](#文章校正について)
  - [パターン① github actions](#パターン1-github-actions)
  - [パターン② husky pre-commit](#パターン2-husky-pre-commit)
- [参考資料](#参考資料)

<!-- /code_chunk_output -->

## ローカル環境のセットアップ

```sh
git clone git@github.com:takiguchi-yu/qiita-content.git
cd qiita-content
npm ci
# 以下のコマンドでバージョンが表示されればセットアップは完了
npx qiita version
```

Qiita CLI に関する詳細は以下の README を参照してください。

- https://github.com/increments/qiita-cli

## 記事の作成

```sh
npx qiita new [ファイル名]
```

## 記事の削除

削除は [Qiita](https://qiita.com/takiguchi-yu) 上からのみ行えます。

## 文章校正について

Qiita に記事をアップする前に文章校正をします。2パターンの仕組みを取り入れてます。どちらの仕組みも `textlint` の技術文書のルールプリセットである `textlint-rule-preset-ja-technical-writing` を使用しています。

##### パターン① github actions

main ブランチに対して Pull Request すると github actions で `textlint` と `reviewdog` を動かしています。`reviewdog` は `textlint` での指摘事項を Pull Request に書き込んでくれる仕組みです。

##### パターン② husky pre-commit

Pull Request が面倒臭いとき、直接 main ブランチを操作して push したいときがあります。その場合、`husky` で `pre-commit` を起動し `textlint` を実行するようにしています。なお textlint は変更のあった markdown のみを対象にしています。

## 参考資料

- [Qiitaの記事をGitHubリポジトリで管理する方法](https://qiita.com/Qiita/items/32c79014509987541130)
- [ZOZO TECH BLOGを支える技術 #2 執筆をサポートするCI/CD](https://techblog.zozo.com/entry/techblog-writing-support-by-ci-cd)
