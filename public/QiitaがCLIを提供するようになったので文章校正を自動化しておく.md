---
title: QiitaがCLIを提供するようになったので文章校正を自動化しておく
tags:
  - QiitaCLI
private: false
updated_at: '2023-08-20T22:08:26+09:00'
id: 0ee440ec499f840ee738
organization_url_name: null
slide: false
ignorePublish: false
---

Qiita CLI が誕生したことで Qiita の記事を GitHub で管理できるようになり、github actions で記事公開までができるようになりました。

詳しくはこちら。

- [Qiitaの記事をGitHubリポジトリで管理する方法](https://qiita.com/Qiita/items/32c79014509987541130)

文章校正には [textlint](https://github.com/textlint/textlint) を使用します。
`textlint` は技術文書のルールプリセット `textlint-rule-preset-ja-technical-writing` を使用します。

#### パターン① github actions

`main` ブランチに対して Pull Request すると github actions で `textlint` と `reviewdog` を動かします。`reviewdog` は `textlint` での指摘事項を Pull Request に書き込んでくれる仕組みです。

詳しくはこちら。（ほぼ丸々流用させていただきました。）

- [ZOZO TECH BLOGを支える技術 #2 執筆をサポートするCI/CD](https://techblog.zozo.com/entry/techblog-writing-support-by-ci-cd)

<details><summary>github actions 詳細</summary>

```yml
name: 'textlint & reviewdog'

on:
  pull_request:
    branches:
      - main
    paths:
      - 'public/*.md'

jobs:
  linter:
    permissions:
      checks: write
      contents: read
      pull-requests: write
    runs-on: ubuntu-latest
    steps:
      - name: 'Checkout'
        uses: actions/checkout@v3
        with:
          fetch-depth: 0

      - name: 'Setup nodejs'
        uses: actions/setup-node@v3
        with:
          node-version: 18
          cache: 'npm'

      - name: 'Setup reviewdog'
        uses: reviewdog/action-setup@v1
        with:
          reviewdog_version: latest

      - name: 'Install node dependencies'
        run: npm ci

      - name: 'textlint & reviewdog'
        env:
          REVIEWDOG_GITHUB_API_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        if: ${{ (github.event_name == 'pull_request') }}
        run: |
          DIFF_FILES=`git diff --name-status origin/main --diff-filter=MA | grep -E "public/.*.md" | cut -f2`
          if [ -z "${DIFF_FILES}" ]; then exit 0; fi
          npx textlint -f checkstyle $(echo ${DIFF_FILES}) | reviewdog -f=checkstyle -name="textlint" -reporter=github-pr-review --fail-on-error=true -filter-mode=added
```

</details>

#### パターン② husky pre-commit

記事を個人で管理している場合、いちいち Pull Request するのが面倒臭いです。
その場合、[husky](https://typicode.github.io/husky/) で `pre-commit` を起動し `textlint` を実行する方法も紹介します。なお `textlint` は変更のあった markdown (\*.md) のみを対象にしています。

<details><summary>pre-commit 詳細</summary>

.husky/pre-commit

```sh
#!/usr/bin/env sh
. "$(dirname "$0")/_/husky.sh"

echo "Run textlint..."
DIFF_FILES=`git diff --name-status origin/main --diff-filter=MA | grep -E ".*.md" | cut -f2`
if [ -z "${DIFF_FILES}" ]; then exit 0; fi
npx textlint $(echo ${DIFF_FILES})
```

</details>

#### その他

自分のエディタに prettier も設定しておくとさらに快適になります。（ここでは割愛）

textlintrc.json の設定は Qiita なので少し緩めに設定しておきます。

<details><summary>textlintrc.json 詳細</summary>

```json
{
  "plugins": {},
  "filters": {},
  "rules": {
    // 技術文書向けのtextlintルールプリセット
    // See https://github.com/textlint-ja/textlint-rule-preset-ja-technical-writing
    "preset-ja-technical-writing": {
      "max-kanji-continuous-len": {
        "max": 6,
        "allow": ["倍精度浮動小数点数"]
      },
      "sentence-length": {
        "max": 125 // １行あたりの文字数
      },
      "ja-no-mixed-period": {
        "allowPeriodMarks": [":"], // 文末が ":" で終わることを許可
        "allowEmojiAtEnd": true, // 文末が絵文字で終わることを許可
        "forceAppendPeriod": true
      },
      "no-exclamation-question-mark": false // 感嘆符（!！）、疑問符（?？）で文章が終わることを許可
    }
  }
}
```

</details>
