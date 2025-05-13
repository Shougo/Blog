---
title: "端末プラグインの歴史と新世代の端末プラグイン ddt.vim"
emoji: "🐚"
type: "tech"
topics: ["vim", "neovim", "denops"]
published: true
---

# 始めに

`dpp.vim` の開発が一通り終了し、次に作成するプラグインについて検討しました。
私は過去作成したプラグインをどんどんリメイクしているのですが、端末プラグインもそろそろリメイクする時ではないかと思いました。

そして`ddt.vim` の開発が開始したのです。今回は現在開発中のプラグインである `ddt.vim` について解説します。

https://github.com/Shougo/ddt.vim


# 端末プラグインの必要性

そもそも、なぜ Vim からコマンドを実行する必要があるのでしょうか。
Vim からコマンドを実行するのは長らく邪道と思われていました。`:help design-not` に槍玉として挙げられていたほどです。

Vim からコマンドを実行すると出力の加工が容易になります。端末でこれをやるにはマウスを用いたり screen, tmux といったものに頼ることになると思います。

Vim なので Vim の設定やプラグインと連携ができます。Vim のカレントディレクトリをそのまま使えたり自動補完も Vim のものを用いることができるのは便利です。
なによりも拡張が容易になります。みなさんはシェルスクリプトよりも Vim script のほうが「自然な」プログラミング言語であり馴染みがあるはずです。

以上より Vim のヘビーユーザーであるほど Vim から外部コマンドを実行するほうがよく、端末プラグインはそれを助けてくれるのです。


# 端末プラグインの歴史

まずは私が利用・開発していた端末プラグインたちについて紹介することにしましょう。

## Shell.vim

https://www.vim.org/scripts/script.php?script_id=118

2007 年から 2008 年頃私が最初に使っていたのは Shell.vim でした。このプラグインは専用のバッファを作成し、バッファに入力したコマンドを実行します。
現代の皆さんはわざわざそんなプラグインを使わず、そのまま端末からコマンドを実行すればよいのではないかと思うかもしれません。
私はそのとき「Windows」を使っていました。あとは言うまでもないでしょう。Windows のコマンドラインは当時かなり苦痛でしたので Vim から実行したかったのです。
もしも当時の私が Linux を使っていたら別の未来もあったかもしれませんね。

## vimshell.vim (2009 年)

https://github.com/Shougo/vimshell.vim

`Shell.vim` は残念ながら開発が止まっていました。私は当時 Emacs の `eshell.el` に憧れており、その使い勝手を Vim で再現したいと思っていました。

https://www.gnu.org/software/emacs/manual/html_mono/eshell.html

`vimshell.vim` は Vim で快適なコマンドライン編集を実現するために作成したプラグインです。

`vimshell.vim` の特徴は Vim script でシェルを再現したことにあります。
Vim script で端末のエスケープシーケンスを解釈し、非同期でコマンドの実行も可能となっています。
当時、Vim 本体に非同期コマンド実行機能はなかったので非同期実行には `vimproc.vim` を用いています。

https://github.com/Shougo/vimproc.vim


## deol.nvim (2016 年)

https://github.com/Shougo/deol.nvim

`vimshell.vim` は当時にしてはよくできたプラグインであり、今でも根強いファンがいるほどでした。
しかし、`vimshell.vim` には大きな問題がありました。
`vimshell.vim` は巨大な Vim scirpt で構築されておりメンテナンスが大変ということです。

丁度そのころ、Neovim に端末機能(`:terminal`) が実装されました。Neovim に遅れて Vim にも同様の機能が実装されました。
本体の端末機能を利用すればわざわざ Vim scirpt でシェルを実装しないで済むだろうと思い、本体の端末機能を利用した `deol.nvim` が生まれたのです。
`deol.nvim` は本体の機能をそのまま利用しているので、実装がコンパクトであり使いやすくなりました。
外部シェルをそのまま実行するので自作シェル特有の互換性の問題もありませんでした。

::: message
Vim の端末機能は Emacs でいう `term.el` に相当する機能です。
つまり当時の私は `eshell.el` から `term.el` に移行をしたようなものです。
:::


## ddt.vim (2024 年)

https://github.com/Shougo/ddt.vim

私は本体の端末機能を利用すれば全てがうまくいくと思っていたのですが、 `deol.nvim` にも問題が発生しました。

* 端末機能は Vim/Neovim で微妙に機能が違う

* Windows 環境での使いづらさ

* パフォーマンスが悪い

* カスタマイズの制限

よくも悪くも Vim の端末機能というのは端末のコマンドをそのまま使えるようにしています。
端末処理は直接 UI を弄れる Vim script よりもどう考えても非効率ですし、コマンドラインが貧弱な Windows では動作が特に遅いです。
Vim だとノーマルモードにいなければ画面が更新されないという問題もあります。

これらの問題から、全てのコマンドライン作業を Vim の端末機能を用いて実行するのは困難なのではないかと思いました。
それならば Vim の端末機能と自作シェル、両方共シームレスに使えるようになればよいのではないかと思ったのです。

`ddt.vim` は従来のコマンドラインプラグインの問題を解決します。
`ddt.vim` は `ddu.vim` のように UI を切り替えることが可能です。
`ddt.vim` の UI には端末機能を用いる `ddt-ui-terminal` と自作シェルである `ddt-ui-shell` が用意されています。
これにより Vim の端末機能のほうがよい場合には端末を、通常の軽い作業は自作シェルと使いわけることができます。

https://github.com/Shougo/ddt-ui-terminal/

https://github.com/Shougo/ddt-ui-shell/

::: message
例えば、私は以下のように UI を切り替えるようにしています。
`name` を terminal と shell で分けることで別々の UI を同時に使用が可能です。

```vim
nnoremap <Space>s  <Cmd>call ddt#start(#{
      \   name: t:->get('ddt_ui_shell_last_name',
      \                 'shell-' .. win_getid()),
      \   ui: 'shell',
      \ })<CR>
nnoremap <Space>t  <Cmd>call ddt#start(#{
      \   name: t:->get('ddt_ui_terminal_last_name',
      \                 'terminal-' .. win_getid()),
      \   ui: 'terminal',
      \ })<CR>
```
:::


# ddt.vim の特徴

`ddt.vim` の特徴は以下の通りです。

* 共通機能を除くほとんど全ての機能をコアから分離する

* 端末機能とシェル機能は拡張機能として提供

* TypeScript と Vim script のハイブリッド実装

* 変数の廃止

* Deno に依存する代わりに環境依存は最小限


# ddt.vim の使用方法

## 設定について

`ddt.vim` の設定のインタフェースは `ddu.vim` をベースにしています。
`ddu.vim` を使用している人なら違和感がなく設定できるかと思います。

```vim
call ddt#custom#patch_global(#{
      \   ui: 'shell',
      \   uiParams: #{
      \     shell: #{
      \       noSaveHistoryCommands: ['history'],
      \       prompt: '%',
      \       promptPattern: '\w*% \?',
      \     },
      \     terminal: #{
      \       command: ['zsh'],
      \       promptPattern: has('win32') ? '\f\+>' : '\w*% \?',
      \     },
      \   },
      \ })
```

`ddt#custom#patch_global()` によりグローバル設定を変更します。

`ui` オプションに `shell` を指定することにより、`ddt-ui-shell` を読み込むという意味になります。

`uiParams` により UI 固有の設定をします。

プラグインの初期化が終わったら `call ddt#start()` を実行すると、デフォルトの設定で起動します。

```vim
autocmd FileType ddt-shell call s:ddt_my_settings()
function! s:ddt_my_settings() abort
  nnoremap <buffer> <C-n>
        \ <Cmd>call ddt#ui#do_action('nextPrompt')<CR>
  nnoremap <buffer> <C-p>
        \ <Cmd>call ddt#ui#do_action('previousPrompt')<CR>
  nnoremap <buffer> <C-y>
        \ <Cmd>call ddt#ui#do_action('pastePrompt')<CR>
  nnoremap <buffer> <CR>
        \ <Cmd>call ddt#ui#do_action('executeLine')<CR>
endfunction
```

`ddt-ui-shell` は filetype `ddt-shell` のバッファーを生成するので、それを利用してキーマッピングの設定を行います。

`ddt#ui#do_action()` は UI 固有のアクションを実行するための機能です。

これで最低限の解説は終了です。基本は分かったはずですので、あとは `ddt.vim` を設定していきながら学んでいきましょう。


# ddt.vim の活用

私が普段どのように ddt.vim を活用しているのかについて解説をします。

私は tmux や screen といったソフトウェアは使っていません。テキストエディタ外での機能追加は余計な副作用と考えているためです。
テキストエディタは普段一つで作業をしています。
各プロジェクトの切り替えにはタブを用いています。各タブそれぞれにカレントディレクトリを持たせています。
タブの切り替えには `ddt.vim` の持つ `ddt_tab` source を用います。

私は git 操作の Vim プラグインも使用していません。git 操作は Vim から直接コマンドを発行しています。

基本的には軽い `ddt-ui-shell` でコマンドを発行しています。端末機能が必要になったときは `ddt-ui-terminal` に切り替えて作業をします。


# ddt-ui-shell の機能制限

`ddt-ui-shell` には現在機能の制限があります。具体的にはコマンドラインの解析については最小限の実装であり、未実装の機能がとても多いです。
現状は単純なコマンドを実行できるだけのものと考えればよいでしょう。
これは、できるだけ早く実用性のあるものにするために構文解析以外の機能を優先して実装を進めていたためです。
いずれはちゃんとした構文解析を実装したいと考えていますが未定です。

`ddt-ui-shell` は多くのエスケープシーケンスを解釈しません。解釈できないエスケープシーケンスはそのまま削除するようになっています。
エスケープシーケンスの処理が必要ならば基本的には `ddt-ui-terminal` を使ってください。

ただし、最近の `ddt-ui-shell` は ANSI エスケープシーケンスによる色付けが可能となりました。
エスケープシーケンスの解析には以下のライブラリを利用しています。

https://github.com/lambdalisue/deno-ansi-escape-code


# これからについて

私の今年の目標として、Emacs を本気で勉強するというのがあります。
とはいえ、Emacs に移行するというのではなく Emacs の文化を学ぶことでそれを Vim の世界に取り入れる予定です。


# GitHub sponsors について

今回の `ddt.vim` プラグインの開発は GitHub sponsors の皆さんの支援によって行われました。
何年もかけて開発してきたプラグインを作り直すのはとてもエネルギーの必要な作業です。
皆さんからの支援がなければ、到底実現できなかっただろうと思います。

https://zenn.dev/shougo/articles/github-sponsors
