---
title: "新世代の自動補完プラグイン ddc.vim"
emoji: "🖤"
type: "tech"
topics: ["vim", "neovim", "denops"]
published: false
---

## はじめに

ddc.vim は最初は何もなく vaporware でした。
ここ数ヶ月力を入れて ddc.vim の開発をしていて、ようやく一通りの機能が揃いました
ので、広くユーザーに使ってもらうフェーズに進めたいと考えています。
ユーザーからの要望やバグ報告に一通り対応が終わってから正式版の 1.0 となりますの
でよろしくお願いします。


## ddc.vim の特徴

### Vim8/neovim 両対応

deoplete.nvim でも vim-hug-neovim-rpc を用いることにより Vim に対応していました
が、インストールが煩雑になる、neovim と比較してパフォーマンスに問題がある、安定
性への問題がありました。

ddc.vim では denops.vim というフレームワークを利用することにより、Vim/neovim の
両方で安定して動作するようになっています。
昨今新しく開発される自動補完プラグインは neovim に特化したものが多いので、両対
応しているのは大きな利点だと思います。


### Python 非依存、Deno 依存

deoplete.nvim はプラグインの実装に Python を利用していました。Python はライブラ
リが充実しておりよい言語であるのですが、あまりに色んな環境で使われているがゆえ
に特殊な環境だとプラグインがうまく動作しないということがよくありました。
deoplete.nvim を開発した当時は Python でリモートプラグインという選択は間違って
いなかったと思うのですが、deoplete.nvim の開発から 6 年も経過しておりデメリット
を無視するのも難しくなってきました。
neovim 開発やユーザーが Lua でのプラグイン開発に傾倒しており、今後のリモートプ
ラグインの改善に期待できなくなったのも理由です。Vim の開発も Vim script を強化
して Vim9 script の推進をしていくようなので、Vim での外部インタフェース機能につ
いても同じ道を辿っています。
かといって、プラグイン開発を Lua や Vim 9 に移行するわけにはいきません。
Lua, Vim9 への移行は移植性が低下し Vim/neovim 両方でプラグインが動作することが
難しくなりますし、それらはあくまで高速な Vim script の立ち位置です。
Vim script 部分がいくら高速になろうとも、処理を別のプロセスに移譲しない限りテキ
ストエディタの負荷を下げることはできません。Vim script を高速化するその道は私が
既に neocomplete.nvim(2014/01 開発) で実践し挫折しています。

Deno は npm の後継というだけあり、npm に似ていますがインストールアップデートが
楽、依存関係の自動解決、スクリプト言語のような使い心地、環境を汚しにくいという
利点があります。
ライブラリが自動的にインストールされるのはプラグイン開発にとって有用であり、こ
れからに期待できます。私の目的にとっては Deno が最適であり、他の言語は選択肢に
なかったのです。

Python の代わりに Deno(denops.vim) に依存するのは欠点に見えるかもしれませんが、
今後 Deno で作られたツールや denops.vim で作られたプラグインが増えることを考え
ると、この依存は無駄にはなりません。

Deno を用いたことにより、ddc.vim 本体や source, filter の実装は TypeScript にな
りました。使用する正規表現も TypeScript で扱えるものが基本となっています。Web
技術者にとってはコードを作りやすくなり嬉しいかもしれません。


### 非同期処理

ddc.vim は deoplete.nvim と同様に別プロセスで処理をしているだけでなく、プラグイ
ン内部で非同期処理を扱うことで処理を高速化しています。これはプラグイン実装に
Deno を用いたことによる利点です。
もちろん deoplete.nvim でも非同期処理を扱うことはできたのですが、様々な制限が存
在し自由に非同期処理を扱えるとはほど遠いものでした。これは私が deoplete.nvim の
開発を終了した理由の一つとなります。


### 設定項目、設定方法の整理

ddc.vim では deoplete.nvim, denite.nvim での設定項目をより整理して汎用化するこ
とにしました。ddc.vim では以下の設定項目があります。

* 本体の設定
* source/filter の設定
* source/filter 独自の設定

本体の設定は deoplete.nvim でいう `deoplete#custom#option()` に相当する機能で
す。

source/filter の設定は deoplete.nvim でいう `deoplete#custom#source()` に相当す
る機能です。

source/filter 独自の設定は deoplete.nvim でいう `deoplete#custom#var()` に相当す
る機能です。source/filter 固有のパラメーターを制御するときに使用します。

ddc.vim の設定には変数は一切存在せず、全ての設定は以下の custom API を用いて行
うことになります。


* グローバル設定 (`ddc#custom#patch_global()`)
* filetype local 設定 (`ddc#custom#patch_filetype()`)
* バッファー local 設定 (`ddc#custom#patch_buffer()`)

deoplete.nvim 時代には、一部オプションが filetype を解釈していましたが(例:
`keyword_patterns`)、ddc.vim にはそのようなオプションはありません。
オプションを filetype local 設定することで同じ役割を行います。

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

source/filter の設定に関してですが、ddc.vim には source 固有のデフォルト値は存
在しません。全ての source が決まったデフォルト値を持っており、ユーザーが自由に
カスタマイズすることが可能です。
deoplete.nvim では source がユーザーの source 固有でないカスタマイズ値を上書き
してデフォルト値を設定することができるという仕様でした。つまり source の推奨設
定を source 側でセットすることができたのです。確かにこの仕様だと ユーザーの設定
を減らすことができると思いますが、ユーザーが自分の思う通りに source を制御した
いと思ったときに、source 側のコードを確認しないといけません。これはかなりの手間
です。

ddc.vim では、source 側が何か推奨する設定をユーザーにしてほしいと思った場合、
source のドキュメントにてサンプルを提示することが推奨されます。
ユーザーがその指示に従うかどうかは自由です。ユーザーが選択できることこそが価値
なのです。


### source, filter の分離

ddc.vim では標準の source, filter を用意せず、全て分離しています。
これによりユーザーの設定が若干増える、それぞれの source, filter を管理する手間
がかかってくるのですが利点があります。
ユーザーは好きな source, filter を組み合わせられるということです。
キーマッピングについてもそうですが、何を標準とするかは人によって異なります。開
発者が標準を押し付けるべきではない、機能を選択できることこそ Vim の強みなので
す。
他は開発上の利点として、本体の開発に集中できること、個別の source の問題で本体
の issues が汚れないことがあげられます。
これに共ない、ddc.vim では本体だけでは何も動かなくなっています。source, filter
を適切にインストールし設定をしなければいけませんが 特に難しいことはないはずで
す。
中には全ての source を同梱する思想の補完プラグインもあります。YouCompleteMe が
その代表です。この方式には利点があって、中央集圏的に管理することができて source
の仕様変更にもすぐに対応が可能です。


## 廃止した機能について

ddc.vim では時代に合わせて一部機能を廃止しています。要望されても実装を行いませ
ん。


### omni_patterns

deoplete から生のオムニ補完(`<C-x><C-o>`)を呼び出す機能です。機能制限はあるもの
のオムニ補完 sourceでは呼び出せない副作用のある omnifunc を自動的に呼ぶのに重宝
していましたが、昨今ではLSP server/client の完成度が上がったため、昔のようにオ
ムニ補完関数を直接呼ぶことは少なくなっているので廃止としました。


### 補完候補の動的な更新

deoplete.nvim は denite.nvim のように補完候補を動的に追加していくことが可能でし
た。これに対応するほうが便利だろうと思い導入したのですが、自動補完の場合動的に
補完ウインドウを更新するとチラつくうえ、これが必要なケースがとても少ないです。
よって ddc.vim では廃止することにしました。
その代わりに後からコールバックにより補完候補を更新することが可能になっているの
で代用してください。コールバックによる更新は
[ddc-nvim-lsp](https://github.com/Shougo/ddc-nvim-lsp) にて使用されています。


### prev_completion

prev_completion は補完の高速化を行うための一つの機能です。新しい補完の計算が終
わるまで以前の補完結果を利用して補完候補の表示を行います。
ddc.vim では deoplete.nvim より補完が高速化しており、このような hack は必要ない
と結論付けました。


### バックスペースによる補完継続

ddc.vim ではバックスペースを行ったときに補完の継続を行いません。仕様を変更した
理由はチラつきを抑えるのが困難だからです。deoplete.nvim では hack を用いて無理
矢理実装していましたが ddc.vim では効果がないことが分かりました。
この仕様変更で私は特に困っていませんが、どうしても補完を出したければ手動で呼び
出すことができます。


### source の rank

ddc.vim には deoplete.nvim とは異なり source に rank が存在しません。では、どの
ように候補の順番を制御するかというと、sources オプションに与えた順番がそのまま
利用されます。これは denite.nvim の挙動とほとんど同じです。
なぜこのようにしたのかというと、実装を単純にするためとユーザーにとって分かりや
すいからです。ddc.vim では使用する全ての source をリストで宣言する必要があるの
で、リストの順番通りに候補が表示されるほうが設定が簡単になると考えました。


### ignore_sources

deoplete.nvim では基本的に全てのインストールした source が有効になり、
ignore_sources オプションによって特定の source を無視するこ とができました。
ddc.vim では全ての使用する source をユーザーが指定することになり、特定の source
を無視するオプションは無意味なので廃止となりました。


## 自分では実装しない機能について

完全に廃止した機能ではないですが、諸事情により自分では実装しない機能につい て説
明します。ユーザーが独自に実装することは可能です。


### ファイルパス補完 source

他の source と比較して重要度が高い source であることは私も分かっていますが、経
験上 LSP 関連を除いた通常 source にて一番トラブルのある補完 source です(次点は
オムニ補完 source です)。もちろん、適当に実装するのは簡単です。

ファイルパス補完はファイルの探索でエラーになりやすい、ファイル補完開始位置の計
算が難しい、他の source と補完開始位置が違うので容易にコンフリクト、といった問
題点を抱えています。

トラブルが起こったときに対応するのが大変なので、私は実装しないという判断をしま
した。どうしても必要な人が実装してください。


### オムニ補完 source

`omni_patterns` の廃止と同じ理由です。現在オムニ補完に頼るべき場面はとても少な
いうえにオムニ補完は計算中に Vim を停止させるのでパフォーマンス上の問題がありま
す。オムニ補完関数の実装により副作用があったり挙動が違うためサポートするのも大
変です。


### include 補完 source, tag 補完 source

こちらも LSP の進化によりほとんど必要がなくなっています。過渡的な技術であったと
いう印象です。tag 補完 source は巨大なタグがある場合にパフォーマンスの問題を引
き起こします。


## ddc.vim の使用方法

詳しい使用方法はドキュメントを参照してください、としたいのですが簡単に説明をし
ておきます。

前述の通り、ddc.vim の導入には Deno と denops.vim が必須となります。
先に導入をしておいてください。

- <https://deno.land/>
- <https://github.com/vim-denops/denops.vim>

ここではプラグインと依存関係は既にインストール済み、ロード済みとして話を続けま
す。

必須の依存関係の他にも、ddc.vim ではデフォルトの sources, filters を廃止してい
るため sources, filters のインストールが必須となります。
自分が必要となる source, filter をインストールしましょう。以下のリンクから探す
と良いと思います。

https://github.com/topics/ddc-source
https://github.com/topics/ddc-filter

ここでは、比較的一般的な source である ddc-around, ddc-matcher_head,
ddc-sorter_rank を使用するものとします。

https://github.com/Shougo/ddc-around
https://github.com/Shougo/ddc-matcher_head
https://github.com/Shougo/ddc-sorter_rank

ddc.vim の設定は以下のように行います。

```vim
call ddc#custom#patch_global('sources', ['around'])
call ddc#custom#patch_global('sourceOptions', {
      \ '_': {
      \   'matchers': ['matcher_head'],
      \   'sorters': ['sorter_rank']},
      \ })
call ddc#enable()
```

`ddc#custom#patch_global()` によりグローバル設定を変更します。デフォルトでは
source は何も指定されていないため、around source を利用するために `sources` オ
プションを設定する必要があります。

`sourceOptions` により source 固有の設定を行っています。source 名に
`_` を与えることでデフォルトの設定を変更します。ここでは source を設定しても入
力の絞り込みを行う `matchers`、候補の ソートを行う `sorters` をセットしていま
す。

deoplete.nvim と同様に ddc.vim もインストールするだけでは有効になりません。
`ddc#enable()` を呼ぶことにより有効化します。

それでは続いて source を追加してみましょう。今回は ddc-nextword を用います。

https://github.com/Shougo/ddc-nextword

ddc.vim の設定を以下のように変更します。

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

`sources` オプションに source 名のリストをセットしますが、このリストの順番がそ
のまま候補の順番となります。ここでは `['around', 'nextword']` なので、around
source の候補の次に nextword source の候補が必ず現れます。

`sourceOptions` に source 名を指定したので source 固有にオプションの変更を行い
ます。ここでは around source と nextword source の `mark` を変更します。

`mark` とは source を追加したときに、どれがどの source の補完候補か分からなくな
るので、わかりやすく表示するための機能です。deoplete.nvim とは異なり、mark はデ
フォルトではセットされていません。ユーザーが明示的にセットする必要があります。

ちなみに、`sources` や `matchers`, `sorters`, `converters` は名前のチェック機能
が存在しており、インストールしていないものを指定するとエラーになります。

これで最低限の解説は終了です。基本は分かったはずなので、あとは ddc.vim を設定し
ていきながら学んでいきましょう。


## これからのプラグイン開発について

ddc.vim の開発は一通り完了し、自分が日常的に使う上で困らなくなりました。ddc.vim
の今後は機能追加より機能の安定化に注力し、正式版のリリースを目指す予定です。

ddc.vim の次は denops.vim 製ファジーファインダーである ddu.vim の開発を進める予
定となっています。
もうすぐ開発を開始できると思うので、お待ちください。
ddu.vim では ddc.vim の経験をベースにより完成度の高いものを作りたいと考えていま
す。私自身 denite.nvim のパフォーマンスに納得がいっていないので、denite.nvim を
越えるものを作らなければ意味はありません。

https://github.com/Shougo/ddu.vim


## github sponsors について

今回の ddc.vim プラグインの開発は github sponsors の皆さんの支援によって行われ
ました。

https://zenn.dev/shougo/articles/github-sponsors

github のコミット数を見てもらえれば、github sponsors を開始した 6 月 7 月 8 月
とコミット数が劇的に伸びているのを確認できるかと思います。

https://github.com/Shougo

https://github.com/sponsors/Shougo/

github sponsors での支援は確実に自分のプラグインの開発意欲の向上に役立っている
ので、プラグインのユーザーに広く支援をお願いいたします。
