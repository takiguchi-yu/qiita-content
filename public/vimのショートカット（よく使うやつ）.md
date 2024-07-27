---
title: vimのショートカット（よく使うやつ）
tags:
  - Vim
private: false
updated_at: '2024-07-27T12:54:50+09:00'
id: 85f9e55006d200c868ea
organization_url_name: null
slide: false
ignorePublish: false
---

普段は VSCode を使うけど、操作感は vim にしたいときは以下の vim プラグインを入れてみる。

- https://marketplace.visualstudio.com/items?itemName=vscodevim.vim

以下のページを参考に VSCode の設定を変更する。

- https://note.com/5mingame2/n/n88d7ceae5a95

```sh
"vim.autoSwitchInputMethod.defaultIM": "com.apple.keylayout.ABC",
"vim.autoSwitchInputMethod.obtainIMCmd": "/usr/bin/true",
"vim.autoSwitchInputMethod.switchIMCmd": "/usr/local/bin/im-select {im}",
"vim.autoSwitchInputMethod.enable": true,
```

- https://github.com/VSCodeVim/Vim#input-method

vim のショートカットを覚えればホームポジションでコーディングができるようになるのでメモしておく。

### カーソル移動

| コマンド | 内容               |
| -------- | ------------------ |
| w        | 次の単語に移動     |
| b        | 前の単語に移動     |
| %        | 対応する括弧に移動 |
| nG       | n行目にジャンプ    |

### 削除

| コマンド | 内容                           |
| -------- | ------------------------------ |
| x        | １文字削除                     |
| dw       | １単語削除                     |
| dd       | １行削除                       |
| D        | 現在のカーソルから行末まで削除 |

### コピペ

| コマンド | 内容         |
| -------- | ------------ |
| yw       | １単語コピー |
| yy       | １行コピー   |
| p        | ペースト     |

### 検索

| コマンド | 内容                     |
| -------- | ------------------------ |
| \*       | カーソルの単語を下に検索 |
| #        | カーソルの単語を上に検索 |

### 編集

| コマンド              | 内容                                                                     |
| --------------------- | ------------------------------------------------------------------------ |
| u                     | 元に戻す                                                                 |
| ctrl+r                | 進む                                                                     |
| o                     | 行末にカーソル移動して改行する                                           |
| ctrl + v -> shift + i | 矩形選択して編集                                                         |
| g -> b -> shift + i   | マルチカーソルして編集。visual mode でカーソル移動する場合は、 g → b → v |

### 便利

| コマンド | 内容                                                      |
| -------- | --------------------------------------------------------- |
| J        | 複数行を連結<br>n + J で n 行分をまとめて連結する         |
| =        | インデントを直す。該当行を修正する場合は = を２回入力する |

### その他

| コマンド                | 内容                         |
| ----------------------- | ---------------------------- |
| :edit ++encoding=euc-jp | 指定した文字コードに変換する |

### 複数行頭に文字を挿入する

```sh
0 で行頭にフォーカスを移動する。
Ctrl-v で矩形選択モードにし、j を数回押して、対象範囲の行の行頭がすべて選択された状態にする。
Shift-i で行頭への挿入モードに入り、任意の文字を入力。
Esc で挿入モードを抜ける。
```

### 参考

その他基本的なところは以下を参考。

https://qiita.com/HiromuMasuda0228/items/5ef842ee50e7ac5e85f2
