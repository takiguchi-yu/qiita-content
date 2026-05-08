---
title: Nixのプロジェクトごとの環境構築 (flake.nix + direnv)
tags:
  - 'Nix'
  - 'direnv'
private: false
updated_at: ''
id: null
organization_url_name: null
slide: false
ignorePublish: false
---

初期セットアップは[HomebrewからNixに移行した](https://qiita.com/takiguchi-yu/items/82c554054575c3289004)で完了している前提で、プロジェクトごとの環境構築方法を紹介します。

Home ManagerはグローバルなCLIツールの管理に最適ですが、特定のプロジェクト（リポジトリ）ごとに異なるバージョンの言語やツール（例: 特定のプロジェクトだけ Go 1.22 と PostgreSQL 17 クライアントを使いたい）を管理したい場合は、`flake.nix` と `direnv` を組み合わせます。

これにより、プロジェクトのディレクトリに移動（`cd`）した瞬間に、自動で必要な環境が立ち上がるようになります。（`asdf` や `mise` の代替となります）

## 1. direnv の導入

ディレクトリ移動時に環境を自動で切り替えるため、Home Manager を使って `direnv` と、Nix環境の読み込みを高速化する `nix-direnv` をインストールします。

先ほどの `home.nix` に以下を追記し、`home-manager switch` を実行します。

```nix: ~/.config/home-manager/home.nix
  # (前略)

  programs.direnv = {
    enable = true;
    enableZshIntegration = true; # zshを使っている場合 (bashの場合は enableBashIntegration)
    nix-direnv.enable = true;
  };

  # (後略)

```

💡 **豆知識:** `nix-direnv` を有効にしておくと、現在のプロジェクトで使っている環境がNixの自動削除（ガベージコレクション）に巻き込まれるのを防ぎ、ディレクトリ移動時のロードが劇的に高速化されます。

## 2. プロジェクトルートに `flake.nix` を作成

対象のプロジェクトディレクトリに移動し、`flake.nix` を作成します。
ここではバックエンド・インフラ開発を想定し、Go、PostgreSQL 17、AWS CLI、AWS CDK を揃えるテンプレートを用意しました。

```nix: flake.nix
{
  description = "Backend development environment";

  inputs = {
    nixpkgs.url = "github:nixos/nixpkgs/nixpkgs-unstable";
  };

  outputs = { self, nixpkgs }:
    let
      # お使いの環境に合わせて指定します（M系Macなら aarch64-darwin）
      system = "aarch64-darwin";
      pkgs = nixpkgs.legacyPackages.${system};
    in
    {
      devShells.${system}.default = pkgs.mkShell {
        # ここに必要なパッケージを列挙します
        buildInputs = with pkgs; [
          go
          postgresql_17
          awscli2
          aws-cdk
        ];

        # 環境に入った時に実行されるスクリプト（環境変数やエイリアスなどもここで設定可能）
        shellHook = ''
          export PGHOST="localhost"
          export PGUSER="postgres"

          # プロジェクト固有のDBデータディレクトリを指定
          export PGDATA="$PWD/.local/pgdata"

          # よく使う起動コマンドをエイリアス化しておく
          alias dev-db-init="initdb -E UTF8"
          alias dev-db-start="pg_ctl start"
          alias dev-server="go run main.go"

          echo "🚀 開発環境がロードされました"
        '';
      };
    };
}
```

## 3. 手動で環境に入る (`nix develop`)

direnv で自動化する前に、Nix標準のコマンドで環境が正しく構築できるか確認してみましょう。プロジェクトルートで以下のコマンドを実行します。

```sh
nix develop
```

このコマンドを実行すると、`flake.nix` で定義した開発専用のシェルに入ります。この中では、まだシステムにインストールしていないはずの `go` や `aws` コマンドが使えるようになっています。作業が終わったら `exit` または `Ctrl+D` で元のシェルに戻れます。

また、**「シェルには入りたくないが、その環境で1回だけコマンドを実行したい」** という場合は `-c` オプションが便利です。

```sh
# 開発環境の Go を使ってバージョンを確認するだけ
nix develop -c go version
```

これは CI/CD のパイプラインや、特定のツールだけを隔離して実行したいスクリプト内で非常に重宝します。

## 4. `.envrc` の作成と有効化

手動で入れることが確認できたら、ディレクトリ移動時の自動化を設定します。
同じディレクトリに `.envrc` というファイルを作成し、Nix Flakes を使用する宣言を1行だけ記述します。ここで記述する `use flake` は、内部的に `nix develop` を呼び出して環境変数を現在のシェルにインポートする指示です。

```sh: .envrc
use flake
```

ファイルを保存すると、direnv がセキュリティの警告を出すので、以下のコマンドを実行して許可（allow）します。

```sh
direnv allow
```

⚠️ **注意:** direnvを有効にすると、プロジェクト内に `.direnv/` ディレクトリが生成されます。誤ってコミットしないよう、プロジェクトの `.gitignore` に `.direnv/` を追加しておきましょう。

## 5. 動作確認とチーム開発での運用 (`flake.lock`)

許可した瞬間にパッケージのダウンロードと環境の構築が走り、`shellHook` に記述した「🚀 開発環境がロードされました」というメッセージが表示されれば成功です。

以降は、このディレクトリに入るだけで自動的に指定したバージョンの Go や PostgreSQL コマンドが使えるようになり、ディレクトリを抜けると元の環境に戻るようになります。

### チーム開発における `flake.lock` の重要性

`flake.nix` を実行すると、自動的に `flake.lock` というファイルが生成されます。これは npm の `package-lock.json` のようなもので、パッケージのバージョンを厳密に固定する役割を持ちます。

チーム開発では、**`flake.nix` と `flake.lock` の両方を Git にコミット**して共有してください。これにより、OS（macOSやLinux）が違っても、チーム全員が「全く同じバージョンの開発環境」を一瞬で再現できるようになります。「私の環境では動くのに…」というトラブルはこれで劇的に減るはずです。

（※パッケージのバージョンを最新に更新したい場合は `nix flake update` を実行します）
