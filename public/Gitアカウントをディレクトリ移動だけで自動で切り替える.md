---
title: Gitアカウントをディレクトリ移動だけで自動で切り替える
tags:
  - 'Git'
  - '開発環境'
private: false
updated_at: ''
id: null
organization_url_name: null
slide: false
ignorePublish: false
---

## やったこと

これまで `.gitconfig` や `.ssh/config` を手動で書き換えてGitアカウントを切り替えていましたが、ディレクトリ移動だけで自動で切り替わるようにした。

```bash
# ディレクトリ移動だけでGitアカウントを自動で切り替わる
$ cd ~/git/work/work-repo/
$                                         💼 Work
$ cd ~/git/private/private-repo/
$                                         🏠 Private
```

## アプローチの概要

- 自動化: ディレクトリの場所に基づいて、Git 設定を自動で読み替える (`includeIf`)
- SSH鍵: `~/.ssh/config` に依存せず、Git 設定内で直接 SSH 鍵を指定する (`core.sshCommand`)
- 可視化（オプショナル）: Fish シェルのプロンプトに現在のアカウント状態を表示する

## Step1: Git リポジトリのディレクトリ構成の整理

Gitアカウントごとにディレクトリを分けておく。こうしておくことで後述の `includeIf` の条件をシンプルにできる。

```bash
~/git/
├── work/      # 仕事用リポジトリを集約
└── private/   # プライベート用リポジトリを集約
```

## Step2: Git 設定ファイルの分離

`~/.gitconfig` の一番下に以下を追加する。`includeIf` を使って仕事用とプライベート用の設定ファイルを分けて読み込むようにする。さらに `core.sshCommand` を使ってそれぞれのSSH鍵を指定する。
一番下に設定することで既存の設定は上書きされる。

### `~/.gitconfig`

```ini:.gitconfig
# git アカウントをディレクトリごとに自動切り替え
[includeIf "gitdir:~/git/work/"]
    # 仕事用設定ファイルのパス
    path = .gitconfig-work
[includeIf "gitdir:~/git/private/"]
    # プライベート用設定ファイルのパス
    path = .gitconfig-private
```

それぞれの設定ファイルを作成する。

### `~/.gitconfig-work`

```ini:.gitconfig-work
[user]
    # 仕事用のメールアドレスと名前を指定
    email = work@company.com
    name = takiguchi-yu
[core]
    # 仕事用のSSH鍵を指定
    sshCommand = "ssh -i ~/.ssh/id_rsa_work -F /dev/null"
```

### `~/.gitconfig-private`

```ini:.gitconfig-private
[user]
    # プライベート用のメールアドレスと名前を指定
    email = private@gmail.com
    name = takiguchi-yu
[core]
    # プライベート用のSSH鍵を指定
    sshCommand = "ssh -i ~/.ssh/id_rsa_private -F /dev/null"
```

ここでまで設定すれば、ディレクトリ移動だけでGitアカウントが自動で切り替わるようになる。

## Step3: Fishシェルのプロンプトにアカウント情報を表示する（オプショナル）

Fishシェルを使っている場合、プロンプトに現在のGitアカウント情報を表示することで、どのアカウントが有効になっているかを一目で確認できるようにする。
（当方が fish 使いなので zsh でもできるだろうけど調べてない）

### `~/.config/fish/functions/fish_right_prompt.fish`

```bash:~/.config/fish/functions/fish_right_prompt.fish
# メイン関数（右側に何か表示するシステム既定のフック）
function fish_right_prompt
    render_git_identity
end

# 現在の Github アカウントを表示
function render_git_identity
    # Gitリポジトリ内でなければ何もしない
    if not type -q git; or not git rev-parse --is-inside-work-tree >/dev/null 2>&1
        return
    end

    set -l current_email (git config user.email)

    switch $current_email
        # 仕事用
        case "work@company.co.jp" "work@company.com"
            set_color 00d7ff # Cyan
            echo "💼 Work "

        # プライベート用
        case "private@gmail.com"
            set_color ffaf00 # Yellow
            echo "🏠 Private "

        # その他
        case '*'
            set_color red
            echo "❓ $current_email "
    end

    set_color normal
end
```

ファイル生成後に、Fishシェルを再起動するか、以下のコマンドを実行して関数を再読み込みする。

```bash
source ~/.config/fish/functions/fish_right_prompt.fish
```
