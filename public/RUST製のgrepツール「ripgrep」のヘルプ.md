---
title: RUST製のgrepツール「ripgrep」のヘルプ
tags:
  - grep
private: false
updated_at: '2024-07-27T12:54:50+09:00'
id: 1a3ecb3f103f5239fb04
organization_url_name: null
slide: false
ignorePublish: false
---

[ripgrep](https://github.com/BurntSushi/ripgrep) (rg) は、現在のディレクトリを再帰的に検索し、正規表現パターンに一致する行を見つけます。デフォルトでは、ripgrepはgitignoreルールを尊重し、隠しファイル/ディレクトリおよびバイナリファイルを自動的にスキップします。

非常に高速であることが紹介されています。

| Tool                | Command                                                     | Line count | Time            |
| ------------------- | ----------------------------------------------------------- | ---------- | --------------- |
| ripgrep (Unicode)   | rg -n -w '[A-Z]+\_SUSPEND'                                  | 536        | 0.082s (1.00x)  |
| hypergrep           | hgrep -n -w '[A-Z]+\_SUSPEND'                               | 536        | 0.167s (2.04x)  |
| git grep            | git grep -P -n -w '[A-Z]+\_SUSPEND'                         | 536        | 0.273s (3.34x)  |
| The Silver Searcher | ag -w '[A-Z]+\_SUSPEND'                                     | 534        | 0.443s (5.43x)  |
| ugrep               | ugrep -r --ignore-files --no-hidden -I -w '[A-Z]+\_SUSPEND' | 536        | 0.639s (7.82x)  |
| git grep            | LC_ALL=C git grep -E -n -w '[A-Z]+\_SUSPEND'                | 536        | 0.727s (8.91x)  |
| git grep (Unicode)  | LC_ALL=en_US.UTF-8 git grep -E -n -w '[A-Z]+\_SUSPEND'      | 536        | 2.670s (32.70x) |
| ack                 | ack -w '[A-Z]+\_SUSPEND'                                    | 2677       | 2.935s (35.94x) |

## 基本的な使用方法

```css
rg [OPTIONS] PATTERN [PATH ...]
```

- PATTERN: 検索する正規表現パターン。
- PATH: 検索するファイルまたはディレクトリ。

## ripgrep (`rg`) オプションガイド

ripgrep (`rg`) は、現在のディレクトリを再帰的に検索し、正規表現パターンに一致する行を見つけます。

### オプション一覧

#### 入力オプション

| オプション                     | 説明                                                                |
| ------------------------------ | ------------------------------------------------------------------- |
| `-e PATTERN, --regexp=PATTERN` | パターンを指定して検索。複数回指定可能。                            |
| `-f FILE, --file=FILE`         | ファイルからパターンを読み込む。複数回指定可能。                    |
| `--pre=COMMAND`                | 各入力ファイルに対してCOMMANDを実行し、その出力を検索。             |
| `--pre-glob=GLOB`              | `--pre` フラグと併用して、特定のファイルに対してのみCOMMANDを実行。 |
| `-z, --search-zip`             | 圧縮ファイルを検索対象に含める。                                    |

#### 検索オプション

| オプション                         | 説明                                                           |
| ---------------------------------- | -------------------------------------------------------------- |
| `-s, --case-sensitive`             | 大文字小文字を区別して検索（デフォルト）。                     |
| `-i, --ignore-case`                | 大文字小文字を区別せずに検索。                                 |
| `--crlf`                           | CRLFを行終端として扱う。                                       |
| `--dfa-size-limit=NUM+SUFFIX?`     | 正規表現DFAのサイズ制限。                                      |
| `-E ENCODING, --encoding=ENCODING` | テキストエンコーディングを指定。                               |
| `--engine=ENGINE`                  | 使用する正規表現エンジンを指定（`default`, `pcre2`, `auto`）。 |
| `-F, --fixed-strings`              | リテラルとしてパターンを扱う。                                 |
| `-v, --invert-match`               | 一致しない行を表示。                                           |
| `-x, --line-regexp`                | 行全体が一致する場合のみ表示。                                 |
| `-m NUM, --max-count=NUM`          | 各ファイルにつき最大NUM行まで表示。                            |
| `--mmap`                           | メモリマップを使用して検索（デフォルトで有効）。               |
| `-U, --multiline`                  | 複数行にまたがる検索を有効にする。                             |
| `--multiline-dotall`               | 複数行検索時に`.`が改行文字に一致するように設定。              |
| `--no-unicode`                     | Unicodeモードを無効にする。                                    |
| `--null-data`                      | 行終端をNUL文字に変更。                                        |
| `-P, --pcre2`                      | PCRE2正規表現エンジンを使用。                                  |
| `--regex-size-limit=NUM+SUFFIX?`   | コンパイルされた正規表現のサイズ制限。                         |
| `-S, --smart-case`                 | パターンが全て小文字の場合、ケースインセンシティブ検索を行う。 |
| `--stop-on-nonmatch`               | 一致しない行に遭遇したら検索を停止。                           |

#### フィルターオプション

| オプション                       | 説明                                                                        |
| -------------------------------- | --------------------------------------------------------------------------- |
| `--binary`                       | バイナリファイルも検索対象に含める。                                        |
| `-L, --follow`                   | シンボリックリンクを辿る。                                                  |
| `-g GLOB, --glob=GLOB`           | 特定のパターンに一致するファイルやディレクトリを検索対象に含める/除外する。 |
| `--glob-case-insensitive`        | グロブパターンをケースインセンシティブに処理。                              |
| `-., --hidden`                   | 隠しファイルも検索対象に含める。                                            |
| `--iglob=GLOB`                   | ケースインセンシティブなグロブパターンを指定。                              |
| `--ignore-file=PATH`             | 無視ルールが記載されたファイルを指定。                                      |
| `--ignore-file-case-insensitive` | 無視ファイルをケースインセンシティブに処理。                                |
| `-d NUM, --max-depth=NUM`        | ディレクトリの探索深さをNUMに制限。                                         |
| `--max-filesize=NUM+SUFFIX?`     | NUMバイト以上のファイルを無視。                                             |
| `--no-ignore`                    | 無視ファイルを無視しない。                                                  |
| `--no-ignore-dot`                | `.ignore`や`.rgignore`ファイルを無視しない。                                |
| `--no-ignore-exclude`            | 手動で設定された無視ルールを無視しない。                                    |
| `--no-ignore-files`              | `--ignore-file`フラグを無視。                                               |
| `--no-ignore-global`             | グローバルな無視ルールを無視。                                              |
| `--no-ignore-parent`             | 親ディレクトリの無視ルールを無視。                                          |
| `--no-ignore-vcs`                | バージョン管理の無視ルールを無視。                                          |
| `--no-require-git`               | GitリポジトリでなくてもGitの無視ルールを適用。                              |
| `--one-file-system`              | ファイルシステムの境界を越えない。                                          |
| `-t TYPE, --type=TYPE`           | 特定のファイルタイプを検索対象に含める。                                    |
| `-T TYPE, --type-not=TYPE`       | 特定のファイルタイプを検索対象から除外する。                                |
| `--type-add=TYPESPEC`            | 新しいファイルタイプを追加。                                                |
| `--type-clear=TYPE`              | 既存のファイルタイプのグロブをクリア。                                      |
| `-u, --unrestricted`             | フィルタリングレベルを低減。3回まで繰り返し可能。                           |

#### 出力オプション

| オプション                              | 説明                                                 |
| --------------------------------------- | ---------------------------------------------------- |
| `-A NUM, --after-context=NUM`           | 一致行の後にNUM行表示。                              |
| `-B NUM, --before-context=NUM`          | 一致行の前にNUM行表示。                              |
| `--block-buffered`                      | ブロックバッファリングを使用。                       |
| `-b, --byte-offset`                     | 出力前にバイトオフセットを表示。                     |
| `--color=[never,auto,always]            | カラー出力の制御。                                   |
| `--colors=COLOR_SPEC`                   | 出力のカラー設定。                                   |
| `--column`                              | カラム番号を表示。                                   |
| `-C NUM, --context=NUM`                 | 一致行の前後にNUM行表示。                            |
| `--context-separator=SEPARATOR`         | コンテキスト行の区切り文字を設定。                   |
| `--field-context-separator=SEPARATOR`   | コンテキスト行のフィールド区切り文字を設定。         |
| `--field-match-separator=SEPARATOR`     | 一致行のフィールド区切り文字を設定。                 |
| `--heading`                             | ファイルパスをクラスターの上に表示。                 |
| `-h, --help`                            | ヘルプを表示。                                       |
| `--hostname-bin=COMMAND`                | ホスト名を決定するコマンドを指定。                   |
| `--hyperlink-format=FORMAT`             | 出力に使用するハイパーリンクのフォーマットを設定。   |
| `--include-zero`                        | 一致が0のファイルもカウントして表示。                |
| `--line-buffered`                       | 常に行バッファリングを使用。                         |
| `-n, --line-number`                     | 行番号を表示。                                       |
| `-N, --no-line-number`                  | 行番号を表示しない。                                 |
| `-M NUM, --max-columns=NUM`             | 指定されたカラム数を超える行を省略。                 |
| `--max-columns-preview`                 | 超過行のプレビューを表示。                           |
| `-0, --null`                            | ファイルパスの後にNULバイトを追加。                  |
| `-o, --only-matching`                   | 一致部分のみを表示。                                 |
| `--path-separator=SEPARATOR`            | パス区切り文字を設定。                               |
| `--passthru`                            | 一致する行と一致しない行の両方を表示。               |
| `-p, --pretty`                          | カラー出力、ヘッディング、行番号を常に表示。         |
| `-q, --quiet`                           | 標準出力に何も表示せず、一致が見つかった時点で停止。 |
| `-r REPLACEMENT, --replace=REPLACEMENT` | 一致部分を置換して表示。                             |
| `--sort=SORTBY`                         | 結果を昇順にソート。                                 |
| `--sortr=SORTBY`                        | 結果を降順にソート。                                 |
| `--trim`                                | 行頭のASCIIホワイトスペースを削除。                  |
| `--vimgrep`                             | Vimのgrep形式で結果を表示。                          |
| `-H, --with-filename`                   | 各一致行の前にファイル名を表示。                     |
| `-I, --no-filename`                     | ファイル名を表示しない。                             |
| `--sort-files`                          | ファイルパスで結果をソート（非推奨）。               |

#### 出力モードオプション

| オプション                 | 説明                         |
| -------------------------- | ---------------------------- |
| `-c, --count`              | 一致する行数を表示。         |
| `--count-matches`          | 一致するパターンの数を表示。 |
| `-l, --files-with-matches` | 一致するファイル名のみ表示。 |
| `--files-without-match`    | 一致しないファイル名を表示。 |
| `--json`                   | 結果をJSON Lines形式で表示。 |

#### ログオプション

| オプション             | 説明                                       |
| ---------------------- | ------------------------------------------ |
| `--debug`              | デバッグメッセージを表示。                 |
| `--no-ignore-messages` | 無視ファイルの解析エラーメッセージを抑制。 |
| `--no-messages`        | 特定のエラーメッセージを抑制。             |
| `--stats`              | 検索の統計情報を表示。                     |
| `--trace`              | トレースメッセージを表示。                 |

#### その他のオプション

| オプション        | 説明                                                   |
| ----------------- | ------------------------------------------------------ |
| `--files`         | 実際に検索を行わず、検索対象のファイルを表示。         |
| `--generate=KIND` | 特定の形式で出力を生成（例：`man`, `complete-bash`）。 |
| `--no-config`     | 設定ファイルを無視。                                   |
| `--pcre2-version` | PCRE2のバージョン情報を表示。                          |
| `--type-list`     | サポートされているファイルタイプとそのグロブを表示。   |
| `-V, --version`   | ripgrepのバージョン情報を表示。                        |
