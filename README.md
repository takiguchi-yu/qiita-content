# qiita-content


<!-- @import "[TOC]" {cmd="toc" depthFrom=2 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [Qiita CLI のインストール](#qiita-cli-のインストール)
- [Qiita CLI のセットアップ](#qiita-cli-のセットアップ)
- [記事の作成](#記事の作成)
- [Qiita CLI の詳細](#qiita-cli-の詳細)
- [参考資料](#参考資料)

<!-- /code_chunk_output -->


## Qiita CLI のインストール

```sh
npm install @qiita/qiita-cli --save-dev
npx qiita version
```

## Qiita CLI のセットアップ

```sh
npx qiita init
npx qiita login
# login するときは以下でトークンを発行する
# https://qiita.com/settings/tokens/new?read_qiita=1&write_qiita=1&description=qiita-cli
```

## 記事の作成

```sh
npx qiita new [ファイル名]
```

## Qiita CLI の詳細

- https://github.com/increments/qiita-cli

## 参考資料

- [https://qiita.com/Qiita/items/32c79014509987541130](Qiitaの記事をGitHubリポジトリで管理する方法)

