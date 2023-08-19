# qiita-content

[![Publish articles](https://github.com/takiguchi-yu/qiita-content/actions/workflows/publish.yml/badge.svg)](https://github.com/takiguchi-yu/qiita-content/actions/workflows/publish.yml)

<!-- @import "[TOC]" {cmd="toc" depthFrom=2 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [ローカル環境のセットアップ](#ローカル環境のセットアップ)
- [記事の作成](#記事の作成)
- [記事の削除](#記事の削除)
- [文章校正について](#文章校正について)
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

Qiita CLI に関する詳細は以下の README を参照

- https://github.com/increments/qiita-cli

## 記事の作成

```sh
npx qiita new [ファイル名]
```

## 記事の削除

削除は [Qiita](https://qiita.com/takiguchi-yu) 上から行う。

## 文章校正について

mainブランチに pull request すると textlint & reviewdog が走るようになっている。

## 参考資料

- [Qiitaの記事をGitHubリポジトリで管理する方法](https://qiita.com/Qiita/items/32c79014509987541130)
- [ZOZO TECH BLOGを支える技術 #2 執筆をサポートするCI/CD](https://techblog.zozo.com/entry/techblog-writing-support-by-ci-cd)
