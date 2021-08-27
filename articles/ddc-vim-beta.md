---
title: "新世代の自動補完プラグイン ddc.vim"
emoji: "🖤"
type: "tech"
topics: ["vim", "neovim", "denops"]
published: true
---

## 始めに

ここ数ヵ月力を入れてddc.vimの開発をしていて、ようやく一通りの機能がそろいましたので、広くユーザーに使ってもらうフェーズに進めたいと考えています。
ユーザーからの要望やバグ報告に対応が終わってから正式版の1.0となりますのでよろしくお願いします。

https://github.com/Shougo/ddc.vim


## ddc.vim の特徴

### Vim8/neovim 両対応

deoplete.nvimでもvim-hug-neovim-rpcを用いることによりVimに対応していました。
しかしインストールが煩雑になる、neovimと比較してパフォーマンスに問題がある、安定性への問題がありました。

ddc.vimではdenops.vimというフレームワークを利用することにより、Vim/neovimの両方で安定して動作するようになっています。
昨今新しく開発される自動補完プラグインはneovimに特化したものが多いので、両対応しているのは大きな利点だと思います。


### Python 非依存、Deno 依存

deoplete.nvimはプラグインの実装にPythonを利用していました。Pythonはライブラリが充実しておりよい言語であるのですが、あまりにいろんな環境で使われているがゆえに特殊な環境だとプラグインがうまく動作しないということがよくありました。
deoplete.nvimを開発した当時はPythonでリモートプラグインという選択は間違っていなかったと思うのですが、deoplete.nvimの開発から6年も経過しておりデメリットを無視するのも難しくなってきました。
neovim開発やユーザーがLuaでのプラグイン開発に傾倒しており、今後のリモートプラグインの改善に期待できなくなったのも理由です。Vimの開発もVim scriptを強化してVim9 scriptの推進をしていくようですので、Vimでの外部インタフェース機能についても同じ道をたどっています。
かといって、プラグイン開発をLuaやVim 9に移行するわけにはいきません。
Lua, Vim9への移行は移植性が低下しVim/neovim両方で動作することが難しくなりますし、それらはあくまで高速なVim scriptの立ち位置です。
Vim script部分がいくら高速になろうとも、処理を別のプロセスに委譲しない限りテキストエディタの負荷を下げることはできません。Vim scriptを高速化するその道は私がすでにneocomplete.nvim（2014/01開発）で実践し挫折しています。

Denoはnode.jsの後継というだけあり、node.jsに似ていますがインストールアップデートが楽、依存関係の自動解決、スクリプト言語のような使い心地、環境を汚しにくいという
利点があります。ライブラリが自動的にインストールされるのはプラグイン開発にとって有用であり、これからに期待できます。私の目的にとってはDenoが最適であり、他の言語は選択肢になかったのです。

Pythonの代わりにDeno（denops.vim）に依存するのは欠点に見えるかもしれません。
しかし今後Denoで作られたツールやdenops.vimで作られたプラグインが増えることを考えると、この依存は無駄にはなりません。

Denoを用いたことにより、ddc.vim本体やsource, filterの実装はTypeScriptになりました。使用する正規表現もTypeScriptで扱えるものが基本となっています。Web 技術者にとってはコードを作りやすくなりうれしいかもしれません。


### 非同期処理

ddc.vimはdeoplete.nvimと同様に別プロセスで処理をしているだけでなく、プラグイン内部で非同期処理を扱うことで処理を高速化しています。これはプラグイン実装にDenoを用いたことによる利点です。
もちろんdeoplete.nvimでも非同期処理を扱うことはできたのですが、さまざまな制限が存在し自由に非同期処理を扱えるとはほど遠いものでした。これは私がdeoplete.nvimの開発を終了した理由のひとつとなります。


### 設定項目、設定方法の整理

ddc.vimではdeoplete.nvim, denite.nvimでの設定項目をより整理して汎用化することにしました。ddc.vimでは以下の設定項目があります。

* 本体の設定
* source/filterの設定
* source/filter独自の設定

本体の設定はdeoplete.nvimでいう `deoplete#custom#option()` に相当する機能です。

source/filterの設定はdeoplete.nvimでいう `deoplete#custom#source()` に相当する機能です。

source/filter独自の設定はdeoplete.nvimでいう `deoplete#custom#var()` に相当する機能です。source/filter固有のパラメータを制御するときに使用します。

ddc.vimの設定には変数は一切存在せず、すべての設定は以下のcustom APIを用いて行うことになります。


* グローバル設定 (`ddc#custom#patch_global()`)
* filetype local設定 (`ddc#custom#patch_filetype()`)
* バッファlocal設定 (`ddc#custom#patch_buffer()`)

deoplete.nvim時代には、一部オプションがfiletypeを解釈していました(例：`keyword_patterns`)。
ddc.vimにはそのようなオプションはありません。オプションをfiletype local設定することで同じ役割を行います。

```vim
call ddc#custom#patch_filetype(['vim'], {
\ 'keywordPattern': '[a-zA-Z_][a-zA-Z_0-9:#]*',
\ })
```

これは以下に相当します。

```vim
call deoplete#custom#option('keyword_patterns', {
\ 'vim': '[a-zA-Z_][a-zA-Z_0-9:#]*',
\})
```

source/filterの設定に関してですが、ddc.vimにはsource固有のデフォルト値は存在しません。すべてのsourceが決まったデフォルト値を持っており、ユーザーが自由に
カスタマイズできます。deoplete.nvimではsourceがユーザーのsource固有でないカスタマイズ値を上書きしてデフォルト値を設定できるという仕様でした。つまりsourceの推奨設定をsource側でセットできたのです。たしかにこの仕様だとユーザーの設定を減らすことができると思いますが、ユーザーが自分の思う通りにsourceを制御したいと思ったときに、source側のコードを確認しないといけません。これはかなりの手間です。

ddc.vimでは、source側が何か推奨する設定をユーザーにしてほしいと思った場合、sourceのドキュメントにてサンプルを提示することが推奨されます。
ユーザーがその指示に従うかどうかは自由です。ユーザーが選択できることこそが価値なのです。


### source, filter の分離

ddc.vimでは標準のsource, filterを用意せず、すべて分離しています。
これによりユーザーの設定が若干増える、それぞれのsource, filterを管理する手間がかかってくるのですが利点があります。
ユーザーは好きなsource, filterを組み合わせられるということです。
キーマッピングについてもそうですが、何を標準とするかは人によって異なります。開発者が標準を押し付けるべきではない、機能を選択できることこそVimの強みなので
す。
ほかは開発上の利点として、本体の開発に集中できること、個別のsourceの問題で本体のissuesが汚れないことがあげられます。
これに伴い、ddc.vimでは本体だけでは何も動かなくなっています。source, filterを適切にインストールし設定をしなければいけませんが特に難しいことはないはずで
す。
中にはすべてのsourceを同梱する思想の補完プラグインもあります。YouCompleteMeがその代表です。この方式には利点があって、中央集権的に管理できてsourceの仕様変更にもすぐに対応が可能です。


## 廃止した機能について

ddc.vimでは時代に合わせて一部機能を廃止しています。要望されても実装しません。


### omni_patterns

deopleteから生のオムニ補完(`<C-x><C-o>`)を呼び出す機能です。機能制限はあるもののオムニ補完sourceでは呼び出せない副作用のあるomnifuncを自動的に呼ぶのに重宝していました。昨今ではLSP server/clientの完成度が上がったため、昔のようにオムニ補完関数を直接呼ぶことは少なくなっているので廃止としました。


### 補完候補の動的な更新

deoplete.nvimはdenite.nvimのように補完候補を動的に追加していくことが可能でした。これに対応するほうが便利だろうと思い導入したのですが、自動補完の場合動的に補完ウィンドウを更新するとチラつくうえ、これが必要なケースがとても少ないです。よってddc.vimでは廃止することにしました。
その代わりに後からコールバックにより補完候補を更新することが可能になっているので代用してください。コールバックによる更新は[ddc-nvim-lsp](https://github.com/Shougo/ddc-nvim-lsp) にて使用されています。


### prev_completion

prev_completionは補完の高速化を行うためのひとつの機能です。新しい補完の計算が終わるまで以前の補完結果を利用して補完候補の表示します。
ddc.vimではdeoplete.nvimより補完が高速化しており、このようなhackは必要ないと結論付けました。


### バックスペースによる補完継続

ddc.vimではバックスペースを行ったときに補完の継続しません。仕様を変更した理由はチラつきを抑えるのが困難だからです。deoplete.nvimではhackを用いて無理矢理実装していましたがddc.vimでは効果がないことが分かりました。この仕様変更で私は特に困っていませんが、どうしても補完を出したければ手動で呼び出すことができます。


### source の rank

ddc.vimにはdeoplete.nvimとは異なりsourceにrankが存在しません。では、どのように候補の順番を制御するかというと、sourcesオプションに与えた順番がそのまま利用されます。これはdenite.nvimの挙動とほとんど同じです。
なぜこのようにしたのかというと、実装を単純にするためとユーザーにとって分かりやすいからです。ddc.vimでは使用するすべてのsourceをリストで宣言する必要があるので、リストの順番通りに候補が表示されると設定が簡単になると考えました。


### ignore_sources

deoplete.nvimでは基本的にすべてのインストールしたsourceが有効になり、ignore_sourcesオプションによって特定のsourceを無視できました。
ddc.vimではすべての使用するsourceをユーザーが指定することになり、特定のsourceを無視するオプションは無意味なので廃止となりました。


## 自分では実装しない機能について

完全に廃止した機能ではないですが、諸事情により自分では実装しない機能について説明します。ユーザーは独自に実装できます。


### ファイルパス補完 source

ファイルパス補完 source は他のsourceと比較して重要度が高いsourceであることは私も分かっています。
しかし経験上LSP関連を除いた通常sourceにて一番トラブルのある補完sourceです（次点はオムニ補完sourceです）。もちろん、適当に実装するのは簡単です。

ファイルパス補完はファイルの探索でエラーになりやすい、ファイル補完開始位置の計算が難しい、他のsourceと補完開始位置が違うので容易にコンフリクト、といった問題点を抱えています。

トラブルが起こったときに対応するのがたいへんですので、私は実装しないという判断をしました。どうしても必要な人が実装してください。


### オムニ補完 source

`omni_patterns` の廃止と同じ理由です。現在オムニ補完に頼るべき場面はとても少ないうえにオムニ補完は計算中にVimを停止させるのでパフォーマンス上の問題があります。オムニ補完関数の実装により副作用があったり挙動が違うためサポートするのも大変です。


### include 補完 source, tag 補完 source

こちらもLSPの進化によりほとんど必要がなくなっています。過渡的な技術であったという印象です。tag補完sourceは巨大なタグがある場合にパフォーマンスの問題を引き起こします。


## ddc.vim の使用方法

詳しい使用方法はドキュメントを参照してください、としたいのですが簡単に説明をしておきます。

前述の通り、ddc.vimの導入にはDenoとdenops.vimが必須となります。先に導入をしておいてください。

- <https://deno.land/>
- <https://github.com/vim-denops/denops.vim>

ここではプラグインと依存関係はすでにインストール済み、ロード済みとして話を続けます。

必須の依存関係のほかにも、ddc.vimではデフォルトのsources, filtersを廃止しているためsources, filtersのインストールが必須となります。
自分が必要となるsource, filterをインストールしましょう。以下のリンクから探すと良いです。

https://github.com/topics/ddc-source
https://github.com/topics/ddc-filter

ここでは、比較的一般的なsourceであるddc-around, ddc-matcher_head, ddc-sorter_rankを使用するものとします。

https://github.com/Shougo/ddc-around
https://github.com/Shougo/ddc-matcher_head
https://github.com/Shougo/ddc-sorter_rank

ddc.vimの設定は以下のように行います。

```vim
call ddc#custom#patch_global('sources', ['around'])
call ddc#custom#patch_global('sourceOptions', {
      \ '_': {
      \   'matchers': ['matcher_head'],
      \   'sorters': ['sorter_rank']},
      \ })
call ddc#enable()
```

`ddc#custom#patch_global()` によりグローバル設定を変更します。デフォルトではsourceは何も指定されていないため、around sourceを利用するために `sources` オプションを設定する必要があります。

`sourceOptions` によりsource固有の設定をします。source名に `_` を与えることでデフォルトの設定を変更します。ここではsourceを設定しても入力の絞り込みを行う `matchers`、候補のソートを行う `sorters` をセットしています。

deoplete.nvimと同様にddc.vimもインストールするだけでは有効になりません。`ddc#enable()` を呼ぶことにより有効化します。

それでは続いてsourceを追加してみましょう。今回はddc-nextwordを用います。

https://github.com/Shougo/ddc-nextword

ddc.vimの設定を以下のように変更します。

```vim
call ddc#custom#patch_global('sources', ['around', 'nextword'])
call ddc#custom#patch_global('sourceOptions', {
      \ 'around': {'mark': 'A'},
      \ 'nextword': {'mark': 'nextword'},
      \ '_': {
      \   'matchers': ['matcher_head'],
      \   'sorters': ['sorter_rank']},
      \ })
call ddc#enable()
```

補完を動かしてみると、以下のような表示になるかと思います。

![ddc.vim](/images/ddc.png)

`sources` オプションにsource名のリストをセットしますが、このリストの順番がそのまま候補の順番となります。ここでは `['around', 'nextword']` ですので、around sourceの候補の次にnextword sourceの候補が必ず現れます。

`sourceOptions` にsource名を指定したのでsource固有にオプションの変更します。ここではaround sourceとnextword sourceの `mark` を変更します。

`mark` とはsourceを追加したときに、どれがどのsourceの補完候補か分からなくなるので、わかりやすく表示するための機能です。deoplete.nvimとは異なり、markはデフォルトではセットされていません。ユーザーが明示的にセットする必要があります。

ちなみに、`sources` や `matchers`, `sorters`, `converters` は名前のチェック機能が存在しており、インストールしていないものを指定するとエラーになります。

これで最低限の解説は終了です。基本は分かったはずですので、あとはddc.vimを設定していきながら学んでいきましょう。


## これからのプラグイン開発について

ddc.vimの開発は一通り完了し、自分が日常的に使う上で困らなくなりました。ddc.vimの今後は機能追加より機能の安定化に注力し、正式版のリリースを目指す予定です。

ddc.vimの次はdenops.vim製ファジーファインダーであるddu.vimの開発を進める予定となっています。
もうすぐ開発を開始できると思うので、お待ちください。
ddu.vimではddc.vimの経験をベースにより完成度の高いものを作りたいと考えています。私自身denite.nvimのパフォーマンスに納得がいっていないので、denite.nvimを 越えるものを作らなければ意味はありません。

https://github.com/Shougo/ddu.vim


## GitHub sponsors について

今回のddc.vimプラグインの開発はGitHub sponsorsの皆さんの支援によって行われました。

https://zenn.dev/shougo/articles/github-sponsors

GitHubのコミット数を見てもらえれば、GitHub sponsorsを開始した6月7月8月とコミット数が劇的に伸びているのを確認できるかと思います。

https://github.com/Shougo

https://github.com/sponsors/Shougo/

GitHub sponsorsでの支援は確実に自分のプラグインの開発意欲の向上に役立っているので、プラグインのユーザーに広く支援をお願いします。
