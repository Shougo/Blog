---
title: "新世代のファイラー UI ddu-ui-filer"
emoji: "✨"
type: "tech"
topics: ["vim", "neovim", "denops", "プラグイン", "ファイラー"]
published: true
---

# 始めに

::: message
この記事は「[新世代の UI 作成プラグイン ddu.vim](https://zenn.dev/shougo/articles/ddu-vim-beta)」の続編となります。
`ddu-ui-filer` は `ddu.vim` の UI にしかすぎません。`ddu.vim` の知識を前提とした話をするので、先に `ddu.vim` についてはよく理解してください。
:::

`ddu.vim` の開発が一通り終了し、ようやく `ddu-ui-filer` の開発にとりかかることができました。
ここにきて一通りの機能がそろいましたので、広くユーザーに使ってもらうフェーズに進めたいと考えています。

https://github.com/Shougo/ddu-ui-filer

私が以前に作成したファイラープラグインである `defx.nvim` は既に開発を終了しました。
私自身は既に `defx.nvim` から `ddu-ui-filer` に移行しています。`ddu-ui-filer` に `defx.nvim` の一部機能はまだ実装されていませんが、自分が使うぶんには十分です。


# ファイラープラグイン開発の歴史

私はこれまでいくつかのファイラープラグインを開発してきました。その歴史を振り返ってみることにしましょう。


## vimfiler.vim 2010/01 頃開発

私が一番最初に作成したファイラープラグインは `vimfiler.vim` です。
Windows で私が使用している二画面ファイラーの使い勝手を目指して開発されました。

https://github.com/Shougo/vimfiler.vim

もともと `vimfiler.vim` は `unite.vim` と独立して開発されていましたが、その後統合したという経緯があります。
`unite.vim` と統合した理由ですが、`unite.vim` の source として開発することで source が作りやすくなり `vimfiler.vim` でいろんなものを編集できるようになるだろうと考えたからです。

しかし `unite.vim` と `vimfiler.vim` を統合した副作用により、 `unite.vim` の実装がとても複雑になってしまいました。
それなのに `vimfiler.vim` 用の source はほとんど開発がされず、あまり統合の意味がないまま開発が終わってしまいます。

`vimfiler.vim` の特徴は以下の通りです。

* 二画面モードをサポート

* 多数のオプションに対応

* `unite.vim` との統合

* ツリー表示に対応

`vimfiler.vim` は画面を分割して二画面でのコピーペーストをサポートする珍しいファイラープラグインです。もちろん一画面でも使用することは可能です。

`vimfiler.vim` は Vim script のみで実装されており外部インタフェースに依存していません。ただし Vim script が対応していない機能を実装するために、一部のファイル操作は外部コマンドに依存しています。

今となっては `vimfiler.vim` より優れたファイラープラグインは沢山ありますが、今から 10 年以上前にこのプラグインが世に出ていたという事実が重要でしょう。


## defx.nvim 2017/01 頃開発

`defx.nvim` は `vimfiler.vim` を `denite.nvim` の開発経験によって 0 から作り直すために開発されました。

https://github.com/Shougo/defx.nvim

`defx.nvim` は `denite.nvim` 同様に Python のリモートプラグインとして実装されていますが、`denite.nvim` と実装を統合することは行っていません。
`vimfiler.vim` での経験上、無理なプラグインの統合は大して便利にならず、悲劇を生むことが分かったからです。

`defx.nvim` の特徴は以下の通りです。

* 内部クリップボードを利用した疑似二画面ファイル操作

* デフォルトキーマッピングの廃止

* `denite.nvim` 非依存

* マルチルート対応

* View の分離

* Python 依存

* 高速なファイルリスト表示

* ユーザー定義カラム

`defx.nvim` は `vimfiler.vim` のような二画面ファイル操作は搭載していない代わり、内部クリップボードでコピーペーストが可能です。
内部クリップボードはグローバルで、全ての `defx` バッファで共通しています。

`defx.nvim` では Python を採用することによりファイル操作の実装が簡単になりました。外部コマンドへの依存は Python 以外にはほぼありません。

`defx.nvim` はファイルリストの高速表示に特化して実装しており、10000 ファイルも数秒で表示可能です。

`defx.nvim` の珍しい特徴としては、VSCode のようにマルチルートに対応しているところでしょう。複数のディレクトリルートが同時に表示可能です。

`defx.nvim` では自由にユーザーがカラムを作成できるようになりました。これにより拡張性が上がっています。

::: message
ちなみにデフォルトキーマッピングの廃止は `denite.nvim` よりも `defx.nvim` の方が先に行われました。
:::


## ddu-ui-filer 2022/03 頃開発

`defx.nvim` はかなり優秀なプラグインで、私も気に入っていましたが Python リモートプラグインであるということが最大の欠点でした。

::: message
Python 依存を嫌い、未だに vimfiler を使用する人がいるほどです。
:::

Python 環境はあまりに広く使われているので、Python 環境に依存した謎のエラーは対応に苦労するものでした。
それを解消させるため私は `ddu.vim` の UI として `ddu-ui-filer` の開発に着手しました。

https://github.com/Shougo/ddu-ui-filer

`ddu-ui-filer` では `defx.nvim` においては避けていた `ddu.vim` との統合を行っています。
これが実現できたのは、`ddu.vim` では多くの機能がプラグインに分割されていて機能が整理されているので統合がしやすかったことが挙げられます。


# ddu-ui-filer の機能比較

`ddu-ui-filer` の代表的な機能についてまとめると以下のようになります。

https://zenn.dev/lambdalisue/articles/3deb92360546d526381f#%E5%90%84%E3%83%97%E3%83%A9%E3%82%B0%E3%82%A4%E3%83%B3%E3%81%AE%E5%A4%A7%E9%9B%91%E6%8A%8A%E3%81%AA%E6%A9%9F%E8%83%BD%E6%AF%94%E8%BC%83

| 機能説明 | Defx | ddu-ui-filer |
| ---- | ---- | ---- |
| ウィンドウスタイル | o | o |
| サイドバースタイル | o | o |
| 複数ウィンドウ | o | o |
| ツリー表示 | o | o |
| リモートファイル | △ | - |
| 圧縮ファイル | - | - |
| ファイル操作 | o | o |
| ゴミ箱 | o | △ |
| システム実行 | o | o |
| 一括リネーム | o | o |
| セレクター | - | - |
| 非同期実行 | △ | o |
| 表示フィルター | o | o |
| Quickfix 出力 | - | o |
| セッション | o | - |
| プロパティ | o | - |
| カラム表示 | o | o |
| Git 表示 | o | - |
| Git 操作 | o | - |
| アイコン | o | - |
| ハイジャック | - | - |
| Jumplist | ? | ? |
| Alternate file | o | ? |


# ddu-ui-filer の特徴

`ddu-ui-filer` は `defx.nvim` の機能を `ddu.vim` を用いて実現することをコンセプトに設計されたので、`ddu-ui-filer` 独自の機能というのは実は少ないです。
それでも従来のファイラープラグインとは異なる特徴をいくつか持っています。

一般的なファイラープラグインについては以下の記事を見るとよいでしょう。

https://zenn.dev/lambdalisue/articles/3deb92360546d526381f


## Vim8/neovim 両対応、環境依存が少なくインストールが容易

`ddu-ui-filer` は `denops.vim` を用いているので Vim8/neovim 両方に対応し環境依存も少ないです。

::: message
ただし Deno がうまく動作しない特殊環境では使いにくい可能性があります。それでも Python よりは環境構築が容易です。
:::


## ddu.vim との統合

`vimfiler.vim` でも `unite.vim` との統合を実現していましたが、それは中途半端なものでした。
`unite.vim` は内部で `vimfiler.vim` の機能を呼び出しており、両者が内部的に合体したような歪な実装になっています。

既存のファジーファインダーにもファイル操作のアクションを追加することで、ファイル操作を実現しているものはあります。
しかし `ddu-ui-filer` の場合は `ddu.vim` と完全な統合を実現しています。
完全な統合とはどういうことかというと、`ddu.vim` 側にファイラー機能が実装されているということです。そして `ddu.vim` 側に `ddu-ui-filer` に依存した機能は一つもありません。
そのため `ddu-ui-filer` の実装は既存の UI である `ddu-ui-ff` と同等で簡潔になっています。
`ddu-ui-filer` の実装を `vimfiler.vim` や `defx.nvim` の実装の複雑さと比較すると分かりやすいかと思います。

::: message
たとえば、カラム機能が `ddu-ui-filer` ではなく `ddu.vim` 側で実装されているということも重要です。
:::

`ddu-ui-filer` はあくまで `ddu.vim` の UI を `defx.nvim` 風にしたものであり、ファイラーに必要な基本的な機能は `ddu.vim` に実装されています。
あなたが独自のファイラー UI を作ることも可能です。


## 非同期処理

`defx.nvim` でも一部非同期処理が導入されていますが、UI の描画部分や候補の取得は同期処理になっていました。
`ddu-ui-filer` は `ddu.vim` の非同期処理をそのまま使っているので描画部分、候補の取得を含めて全て非同期で実行されます。
大量の候補があるディレクトリの閲覧やツリーの再帰的展開でその実力は発揮されます。


## 改善されたパフォーマンス

`defx.nvim` では Python を用いることで Vim script で書かれた `vimfiler.vim` よりパフォーマンスが改善していました。
`ddu-ui-filer` では TypeScript が実装に使われており、Python よりも処理が高速化しています。


## 設定項目や設定方法の整理

`ddu-ui-filer` では `ddu.vim` のインタフェースをそのまま用いて設定を行います。
他のプラグインとは根本的な思想が異なるので最初は面喰うかもしれませんが、`ddu.vim` の知識さえあれば簡単に設定できると思います。


## `columns` オプション

`ddu.vim` には `columns` オプションが実装されました。`columns` オプションは `sourceOptions` です。
source に `columns` オプションを設定すると、通常の `display` による候補表示の代わりに指定したカラムによる候補表示が行われます。


## カラムの分離

`ddu-ui-filer` は `defx.nvim` とは異なり、カラム機能も分離されています。
例えば、`ddu-ui-filer` でファイル名を表示させるには `ddu-column-filename` を利用します。

https://github.com/Shougo/ddu-column-filename

あなたが独自のカラムを作成することも可能です。


## source による拡張

`ddu-ui-filer` で扱うのはあくまでも `ddu.vim` の source そのものです。`ddu.vim` の source をそのまま `ddu-ui-filer` で使用することができます。
つまり、`ddu.vim` が候補のツリー表示や展開に対応しています。
ファイラーがツリー表示に対応したり、ファジーファインダーがファイル操作を行えるというのはよくあっても、ファジーファインダーがツリー表示や展開に対応した例は少ないと思います。これは設計と実装の難易度が高いからです。

::: message
ただし使用する source はツリー展開に対応する必要があります
:::


# 廃止した機能について

`ddu-ui-filer` では `defx.nvim` にあった一部の機能を廃止しています。


## 可変長カラム機能

`defx.nvim` では可変長のカラムを定義することが可能でした。
可変長のカラムを用いることでカラムの表示位置をある程度調整することができましたが、それにより実装が複雑になりすぎる問題がありました。
`ddu.vim` では可変長カラム機能をサポートしていません。

可変長のカラムは固定長のカラムの中で複数のカラムを表示させることで擬似的に実現可能です。


# ddu-ui-filer の使用方法

詳しい使用方法はドキュメントを参照してください、としたいのですが簡単に説明をしておきます。

前述の通り、`ddu-ui-filer` の導入には Deno と `denops.vim` が必須となります。
`ddu-ui-filer` はあくまで `ddu.vim` の UI の一つであるため、`ddu.vim` も必要です。

- <https://deno.land/>
- <https://github.com/vim-denops/denops.vim>
- <https://github.com/Shougo/ddu.vim>

ここではプラグインと依存関係はすでにインストール済み、ロード済みとして話を続けます。

必須の依存関係のほかにも、`ddu.vim` ではデフォルトの source, kind, column を廃止しているため、それらのインストールが必須となります。
自分が必要となるものをインストールしましょう。以下のリンクから探すと良いです。

https://github.com/topics/ddu-source
https://github.com/topics/ddu-kind
https://github.com/topics/ddu-column

ここでは、比較的一般的なものである `ddu-source-file`, `ddu-kind-file`, `ddu-column-filename` を使用するものとします。

https://github.com/Shougo/ddu-source-file
https://github.com/Shougo/ddu-kind-file
https://github.com/Shougo/ddu-column-filename

`ddu.vim` の設定は以下のように行います。

```vim
call ddu#custom#patch_global({
    \   'ui': 'filer',
    \   'sources': [{'name': 'file', 'params': {}}],
    \   'sourceOptions': {
    \     '_': {
    \       'columns': ['filename'],
    \     },
    \   },
    \   'kindOptions': {
    \     'file': {
    \       'defaultAction': 'open',
    \     },
    \   }
    \ })
```

`ddu.vim` では `ddu#custom#patch_global()` によりグローバル設定を変更します。

`ui` オプションに `filer` を指定することにより、`ddu-ui-filer` を読み込むという意味になります。

デフォルトでは source は何も指定されていないため、`file` source を利用するために `sources` オプションを設定しています。

`sourceOptions` により source 固有の設定をします。source 名に `_` を与えることでデフォルトの設定を変更します。ここでは `columns` として `filename` をセットしています。

`ddu.vim` 特有の設定として、kind のデフォルトアクションをユーザーが指定する必要があります。ここでは選択したファイルを開く `open` アクションを指定しています。

プラグインの初期化が終わったら `call ddu#start({})` を実行すると、デフォルトの設定で source を起動します。この場合は `file` source が起動して以下のような表示になるはずです。

::: message
`call ddu#start({})` は絶対に Vim/neovim の初期化時に実行しないでください。`denops.vim` の初期化が終わっていないのでエラーになります。
:::

![ddu-ui-filer](/images/ddu-ui-filer.png)

`ddu-ui-filer` ではアイテムの一番上にルートディレクトリが表示され、その下に通常のアイテムが表示されます。
アイテムのソートは UI 独自に行われており、`filter` の設定は無視されるので注意してください。

`ddu-ui-filer` にはデフォルトキーマッピングが存在しないので、`ddu-ui-filer` のウインドウが表示されてもこれだけでは操作ができません。
以下のようにキーマッピングを設定する必要があります。これは `defx.nvim` でも同様の仕様のため、 `defx.nvim` の設定に慣れている人ならば理解しやすいでしょう。

```vim
autocmd FileType ddu-filer call s:ddu_my_settings()
function! s:ddu_my_settings() abort
  nnoremap <buffer><silent> <CR>
        \ <Cmd>call ddu#ui#filer#do_action('itemAction')<CR>
  nnoremap <buffer><silent> <Space>
        \ <Cmd>call ddu#ui#filer#do_action('toggleSelectItem')<CR>
  nnoremap <buffer> o
        \ <Cmd>call ddu#ui#filer#do_action('expandItem',
        \ {'mode': 'toggle'})<CR>
  nnoremap <buffer><silent> q
        \ <Cmd>call ddu#ui#filer#do_action('quit')<CR>
endfunction
```

`ddu-ui-filer` は filetype `ddu-filer` のバッファーを生成するので、それを利用してキーマッピングの設定を行います。

`ddu#ui#filer#do_action()` は UI 固有のアクションを実行するための機能です。

`itemAction` は選択した item またはカーソル上の item の item アクションを実行する機能です。`defx.nvim` でいうと `do_action` に相当します。
このアクションは引数に item アクション名をとりますが、省略するとデフォルトアクションとなります。

`ddu-ui-filer` において特に注意しないといけないことは、`ddu-ui-filer` はあくまで `ddu.vim` の UI であるため UI 固有の設定は全て `uiOptions` や `uiParams` に記述するということです。
ユーザーはそれが何の設定であるか把握しておく必要があります。
例えば、neovim の floating window 機能で `ddu-ui-filer` のウインドウを表示したい場合以下のように設定をします。

```vim
call ddu#custom#patch_global({
    \   'uiParams': {
    \     'filer': {
    \       'split': 'floating',
    \     }
    \   },
    \ })
```

`split` は `ddu-ui-filer` ウインドウの分割設定を変更する `ddu-ui-filer` 固有の設定なので `uiParams` の `filer` をキーに設定しています。

`ddu-ui-filer` の応用例は無限大であるということを少し示します。
ファイラーでよくある、隠しファイルを表示非表示にするには以下のように定義します。

```vim
nnoremap <buffer> >
\ <Cmd>call ddu#ui#filer#do_action('updateOptions', {
\   'sourceOptions': {
\     '_': {
\       'matchers': ToggleHidden(),
\     },
\   },
\ })<CR>

function! ToggleHidden()
  let current = ddu#custom#get_current(b:ddu_ui_name)
  let source_options = get(current, 'sourceOptions', {})
  let source_options_all = get(source_options, '_', {})
  let matchers = get(source_options_all, 'matchers', [])
  return empty(matchers) ? ['matcher_hidden'] : []
endfunction
```

上記のコードでは `matchers` を動的に変更し、`matcher_hidden` を使うことで実現していることが分かるかと思います。
ファイラーとして基本的な機能も `ddu-ui-filer` ではなく、あくまで `ddu.vim` の知識だけで実現できるということがポイントです。

これで最低限の解説は終了です。基本は分かったはずですので、あとは `ddu-ui-filer` を設定していきながら学んでいきましょう。


# これからのプラグイン開発について

`ddu-ui-filer` の開発は一通り完了しました。今後は機能の安定化に尽力します。

次は `vinarise.vim` の後継となるバイナリ処理を行うプラグインの開発を考えています。

https://github.com/Shougo/ddx.vim


# GitHub sponsors について

今回の `ddu-ui-filer`, `ddu.vim` プラグインの開発は GitHub sponsors の皆さんの支援によって行われました。
何年もかけて開発してきたプラグインを作り直すのはとてもエネルギーの必要な作業です。
皆さんからの支援がなければ、到底実現できなかっただろうと思います。

https://zenn.dev/shougo/articles/github-sponsors

https://github.com/sponsors/Shougo/

GitHub sponsors での支援は確実に自分のプラグインの開発意欲の向上に役立っているので、プラグインのユーザーに広く支援をお願いします。
