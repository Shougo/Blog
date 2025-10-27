---
title: "バイナリ編集プラグインの歴史と新世代のバイナリ編集プラグイン ddx.vim"
emoji: "🔎"
type: "tech"
topics: ["vim", "neovim", "denops"]
published: true
---

# 始めに

`ddt.vim` の開発が一通り終了し、次に作成するプラグインについて検討しました。
自分が欲しかったプラグインをどんどん作ってきた私ですが、そろそろ最後の本丸であるバイナリ編集プラグインの開発にとりかかったほうがよいのではないかと考えたのです。

テキストファイルはバイナリファイルでもあります。バイナリファイルを編集できるとはテキストファイルも編集できるということであり、あらゆるファイルをテキストエディタで編集可能ということです。

バイナリ編集というのはテキストエディタに出会う前、私の原点でもありました。
私はプログラミングを学び、パソコンの全てを知りたいと思っていました。パソコンの全てを知るためには機械語の知識、バイナリの知識は欠かせません。
バイナリの世界は私を魅了するのに十分なものでした。私は昔からツール類が好きであり、バイナリを自在に編集するためのツールは私の憧れでした。

ちなみにその後私はテキストエディタと出会い、バイナリと違う道を辿っていくことになります。
私がテキストエディタと出会っていなければ違う道を歩んでいたのでしょうね。


# バイナリ編集への憧れ


# バイナリ編集プラグインの歴史

まずは私が利用・開発していたバイナリ編集プラグインたちについて紹介することにしましょう。

## xxd

私は当初 Vim の標準バイナリ編集ツールである xxd を用いてバイナリを編集していました。
これは確かに便利でしたが、いちいち Vim とは別のコマンドを入力するというのは編集体験が悪いです。
Vim と xxd を統合する Vim script も書くことができましたが、制限が多い使い勝手でした。
私は完全にテキストエディタと統合されたバイナリ編集をやりたいと思っていました。
しかし、誰もやる人がいません。そうしてプラグインを自作することになったのです。


## vinarise.vim (2010 年)

https://github.com/Shougo/vinarise.vim

`vinarise.vim` は Vim script ではなく Python インタフェースで実装されました。
Vim script は残念ながらバイナリ編集に難があるし速度も遅いからです。

`vinarise.vim` はある程度成功したのかなと思っています。
`vinarise.vim` は後に開発されたバイナリ編集プラグインよりも機能が豊富でした。

`vinarise.vim` の特徴は以下の通りです。

* メモリマッピングにより巨大ファイルに対応

* ファイルの編集が可能

* 日本語エンコーディングに対応

* ファイル解析に対応


## deorise.nvim (未完成)

https://github.com/Shougo/deorise.nvim

`vinarise.vim` には実装上の問題点が多かったので、それらを改善しリモートプラグインとして作りなおした `deorise.nvim` を作成しようと計画していました。
しかし他のプラグインの開発作業が忙しく、そうこうしているうちに `denops.vim` が出てくるなどしたので未完成のまま開発放棄されました。


## ddx.vim (2025 年)

https://github.com/Shougo/ddx.vim

`ddx.vim` は `denops.vim` を用いて開発したバイナリ編集プラグインになります。`ddx.vim` については `denops.vim` の存在を知ってから、いつかは開発をしようと思っていました。
しかし、プラグイン開発は忙しく後回しとなっていました。`ddc.vim`, `ddu.vim`, `ddp.vim`, `ddx.vim` と私にはやることが多すぎたためです。

最近になって `denops.vim` の知識も深まり、他のプラグインも安定化し満を持して開発開始となったのが `ddx.vim` となります。


# ddx.vim の特徴

`ddx.vim` の特徴は以下の通りです。

* UI 機能をコアから分離する

* ファイル解析機能は拡張機能として提供

* TypeScript と Vim script のハイブリッド実装

* 変数の廃止

* Deno に依存する代わりに環境依存は最小限


## UI 機能の分離

`vinarise.vim` は UI が本体に統合されていましたが、`ddx.vim` では他のプラグインのように UI を本体から分離しました。それにより、各コンポーネントの責任やインタフェースが明確になり保守しやすくなったと感じています。

`ddu-ui-hex` はよくある 16 進数表示を行う UI ですが、あなたが独自の UI を定義することも可能となります。


## ファイル解析機能は拡張機能として提供

`vinarise.vim` には特定のファイルタイプを解析して `unite.vim` による表示を行うことが可能でした。
これは Vim script でパーサを書き解析を行う仕組みとなっています。理論上はどのようなファイルであっても解析することが可能です。

`vinarise.vim` と同等の機能を実現するため、`ddx.vim` では TypeScript による `Analyzer` を定義することで拡張可能としました。

以下の `ddx-analyzer-zip` は Vim script で書かれていた解析器を TypeScript に実装しなおしたものとなります。

https://github.com/Shougo/ddx-analyzer-zip


# ddx.vim の使用方法

## 設定について

`ddx.vim` の設定のインタフェースは `ddu.vim` をベースにしています。
`ddu.vim` を使用している人なら違和感がなく設定できるかと思います。

```vim
call ddx#custom#patch_global({
    \   'ui': 'hex',
    \   'analyzers': ['zip'],
    \ })
```

`ddx#custom#patch_global()` によりグローバル設定を変更します。

`ui` オプションに `hex` を指定することにより、`ddx-ui-hex` を読み込むという意味になります。

`analyzers` オプションには解析に使用する `Analyzer` を指定します。`zip` を指定することにより `ddx-analyzer-zip` が使用されます。

`uiParams` により UI 固有の設定をします。

プラグインの初期化が終わったら `call ddx#start()` を実行すると、デフォルトの設定で起動します。

```vim
autocmd FileType ddx-hex call s:ddx_my_settings()
function! s:ddx_my_settings() abort
  nnoremap <buffer> q
  \ <Cmd>call ddx#ui#hex#do_action('quit')<CR>
  nnoremap <buffer> r
  \ <Cmd>call ddx#ui#hex#do_action('change')<CR>
  nnoremap <buffer> i
  \ <Cmd>call ddx#ui#hex#do_action('insert')<CR>
  nnoremap <buffer> x
  \ <Cmd>call ddx#ui#hex#do_action('remove')<CR>
  nnoremap <buffer> S
  \ <Cmd>call ddx#ui#hex#do_action('save')<CR>
  nnoremap <buffer> u
  \ <Cmd>call ddx#ui#hex#do_action('undo')<CR>
endfunction
```

`ddx-ui-hex` は filetype `ddx-hex` のバッファーを生成するので、それを利用してキーマッピングの設定を行います。

`ddx#ui#do_action()` は UI 固有のアクションを実行するための機能です。
`quit` は UI を終了することを表し、`change` はカーソルのバイトを変更します。
`vinarise.vim` では実装できなかった `undo` 機能も `ddx.vim` では実装できています。

これで最低限の解説は終了です。基本は分かったはずですので、あとは `ddx.vim` を設定していきながら学んでいきましょう。


## ファイルの編集について

`ddx.vim` は標準でコマンドを提供していませんが、`ddx-commands.vim` を用いると利用しやすいです。
`ddx-commands.vim` をインストールすると `:Ddx` コマンドが利用可能となります。

`:Ddx` コマンドを用いて path オプションを指定することで任意のパスを引数としてファイルを開くことが可能となります。

```vim
:Ddx -path=foo.txt
```

これは以下と同じですが、`:Ddx` コマンドを用いたほうが理解しやすいのではないかと思います。

```vim
:call ddx#start(#{ path: 'foo.txt' })
```


# これからについて

`ddx.vim` はようやく一通りの機能が動作するようになった段階です。`ddx.vim` に足りない機能はいろいろありますが、そこは実践に投入して確認していきたいと思っています。
現在、バイナリ編集でなにができると楽しいのか情報を募集中です。


# GitHub sponsors について

今回の `ddx.vim` プラグインの開発は GitHub sponsors の皆さんの支援によって行われました。
何年もかけて開発してきたプラグインを作り直すのはとてもエネルギーの必要な作業です。
皆さんからの支援がなければ、到底実現できなかっただろうと思います。

https://zenn.dev/shougo/articles/github-sponsors
