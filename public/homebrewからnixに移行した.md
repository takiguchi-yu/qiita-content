---
title: HomebrewからNixに移行した
tags:
  - nix
private: false
updated_at: '2026-05-07T02:10:07+09:00'
id: 82c554054575c3289004
organization_url_name: null
slide: false
ignorePublish: false
---

# Nix とは

`Nix`は、LinuxやmacOSで動作するパッケージマネージャーです。

最近になってよく聞かれるようになった印象ですが、実は2003年から開発されている歴史のあるプロジェクトです。（※ [Wikipedia](<https://ja.wikipedia.org/wiki/Nix_(%E3%83%91%E3%83%83%E3%82%B1%E3%83%BC%E3%82%B8%E7%AE%A1%E7%90%86%E3%82%B7%E3%82%B9%E3%83%86%E3%83%A0)>))

もともと開発者が抱えていた課題として、ソフトウェアの依存関係が複雑化し、「AをインストールするとBが壊れる」といった「依存関係の地獄（Dependency Hell）」が深刻化していました。

そこで、「ソフトウェアのビルドを純粋関数（入力が同じなら出力は常に同じ）として扱い、**すべての依存関係をハッシュ値で一意に管理する**」というアプローチを提唱し、Nixパッケージマネージャを生み出しました。

ただし、これは不完全でした。

Nixはもともと「再現性」を売りにしたツールでしたが、旧来の仕組み（Channelsと呼ばれるもの）には一つだけ大きな弱点がありました。それは「Nixpkgs（パッケージの巨大なリポジトリ）のバージョンが、開発者のローカル環境に依存していた」ことです。

そこで 2021 年頃に、Nixpkgsのバージョンをローカル環境から切り離すための新しい仕組み「`Flakes`」が導入されました。

`Flakes` は、npm の package.json のような宣言的な構成ファイルを使用して、プロジェクトの依存関係を管理する仕組みです。Nix では `flake.nix` と `flake.lock` という、Gitと完全に統合されたロックメカニズムが導入されました。
これにより、プロジェクトごとに異なるNixpkgsのバージョンを指定できるようになり、再現性が大幅に向上しました。

# Nix のインストール

[Zero to Nix](https://zero-to-nix.com/start/install/) を参考に、Nixをインストールします。

Zero to Nix は Nix 開発者の Eelco Dolstra 氏らが設立した Determinate Systems 社が提供している Flakes 前提のモダンな Nix 教育ガイドです。

```sh
curl --proto '=https' --tlsv1.2 -sSf -L https://install.determinate.systems/nix | sh -s -- install
```

インストーラーが完了したら、新しいターミナルセッションを開くと、nix実行ファイルが$PATHに追加されているはずです。確認するには、以下を実行してください。

```sh
$ nix --version
nix (Determinate Nix 3.19.1) 2.34.6
```

# Home Manager の導入

`Home Manager` はNixの公式機能ではなく、「[nix-community](https://github.com/nix-community/home-manager)」という有志コミュニティによって開発されているエコシステムの一部です。

`Home Manager` を使うと、「どのツールを入れるか」だけでなく、「そのツールの設定ファイルの中身をどうするか」までをNix言語で記述し、Gitで完全に管理できるようになります。これが単なるパッケージマネージャであるHomebrewとの最大の違いであり、圧倒的に強力な点です。

## Home Manager のインストール

```sh
nix run home-manager/master -- init --switch
```

実行後、`~/.config/home-manager/` というディレクトリに以下の2つのファイルが作られます。

- `flake.nix`: 「どのバージョンのNixカタログ（Nixpkgs）とHome Managerを使うか」という大枠の依存関係を定義するファイル。
- `home.nix`: 「どのツールをインストールし、どんな設定にするか」を書くメインの設定ファイル。

たとえば、以下のように `home.nix` に記述することで、Gitのインストールと Git の設定が同時にできます。

```nix: home.nix
programs.git = {
  enable = true;
  userName = "名前";
  userEmail = "your.email@example.com";
  aliases = {
    st = "status";
    co = "checkout";
  };
};
```

## 必要なパッケージのインストール

現在 Homebrew でインストールしている CLI ツールをリストアップします

```sh
brew leaves
```

次に、先ほど作成された `~/.config/home-manager/home.nix` を編集します。

インフラやバックエンド開発でよく使われるパッケージ群を例にすると、以下のようになります。

```nix: ~/.config/home-manager/home.nix
{ config, pkgs, ... }:

{
  home.username = "あなたのユーザー名";
  home.homeDirectory = "/Users/あなたのユーザー名"; # macOSの場合

  # このバージョンはHome Managerのリリースバージョンなので、基本は変更しません
  home.stateVersion = "25.11"; # 2026/05/07 時点の最新バージョン

  home.packages = with pkgs; [
    # --- 開発言語・インフラツール ---
    go
    awscli2
    docker-client

    # --- データベース ---
    postgresql_16 # クライアントツール (psqlなど)

    # --- ターミナル・ユーティリティ ---
    gh        # GitHub CLI
    jq        # JSONプロセッサ
    ripgrep   # 高速なgrep
    tree
  ];

  # Home Manager自身を管理できるようにする設定（必須）
  programs.home-manager.enable = true;
}
```

※ 使いたいパッケージの正確な名前は [NixOS Search](https://search.nixos.org/packages) で検索できます。

## Home Manager 設定の適用

`home.nix` を保存したら、以下のコマンドを実行して状態を適用します。これが `brew install` や `brew upgrade` に相当するアクションです。

```sh
home-manager switch
```

たとえば `which gh` や `which aws` を実行して、パスが `/nix/store/...` のようなNixの領域を指していれば成功です。

# Homebrew 側の整理

Nix側でツールが動くことが確認できたら、Homebrewでインストールされていた重複するCLIツールを削除します。

```sh
brew uninstall go awscli postgresql
```

# 今後の運用

まずは、`brew upgrade` を実行する代わりに、「`home.nix` を編集して `home-manager switch` を叩く」 という運用サイクルに慣れるところからスタートしてみてください。  
設定ファイルとして管理されているため、万が一環境が壊れても以前の状態にすぐロールバックできるというNixの恩恵をすぐに実感できるはずです。
