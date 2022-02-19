---
title: "新世代の UI 作成プラグイン ddu.vim"
emoji: "🖤"
type: "tech"
topics: ["vim", "neovim", "denops"]
published: true
---

# 始めに

Note: この記事は「新世代の自動補完プラグイン ddc.vim」の続編となります。

`ddc.vim` の開発が一通り終了し、ようやく `ddu.vim` (dark deno-powered ui framework)の開発にとりかかることができました。
ここにきて一通りの機能がそろいましたので、広くユーザーに使ってもらうフェーズに進めたいと考えています。
`ddc.vim` のときと同様に、ユーザーからの要望やバグ報告に対応が終わってから正式版の 1.0 をリリースしようと考えています。

https://github.com/Shougo/ddu.vim

私が以前に作成したファジーファインダープラグインである `denite.nvim` は既に開発を終了しました。
私自身は既に `denite.nvim` から `ddu.vim` に移行しています。`ddu.vim` に `denite.nvim` の一部機能はまだ実装されていませんが、自分が使うぶんには十分です。


# ファジーファインダーフレームワーク開発の歴史

私はこれまでいくつかのファジーファインダーを開発してきました。その歴史を振り返ってみることにしましょう。


## unite.vim 2010/07 頃開発

私が一番最初に開発したファジーファインダーは `unite.vim` でした。

https://github.com/Shougo/unite.vim

そもそも `unite.vim` を開発するきっかけは、私が Emacs の `anything.el` (2008 年頃登場) に憧れており Vim でも似たようなことをしようと思っていたためです。

https://github.com/emacs-jp/anything

`anything.el` というフレームワークを利用することで、あらゆる操作を統一したインタフェースで行うという思想は私の理想形でした。

当時は Vim プラグインのファジーファインダーとして `FuzzyFinder`(vim-fuzzyfinder 2007 年登場) や `vim-ku`(2008 年登場) というものがあり私はそれらを使っていました。

https://www.vim.org/scripts/script.php?script_id=1984

https://github.com/kana/vim-ku

しかし、それらは `anything.el` と比較していくつか欠点がありました。

* 候補の表示と選択に補完メニューを使うために自由度が低い

* 開発が止まっている

* 機能が少ない

Note: ちなみに `unite.vim` の各種 API は `vim-ku` の影響を受けています。kind という概念や custom API、define で source を定義する部分などです。vim-ku の提唱した UI フレームワークという基本的な概念を自分が再実装し、拡張したのが unite.vim と考えることができます。

`unite.vim` の特徴は以下の通りです。

* 多数のオプションに対応

* source, filter, kind という概念による拡張性

* normal mode, insert mode 両方に対応

`unite.vim` は拡張に拡張を重ねた結果 2010 年当時に開発されたプラグインとしては完成度が高く、2022 年になっても機能的には見劣りがしません。未だに `unite.vim` を使用している人がいるほどです。

しかし `unite.vim` も万能というわけではなく、以下のような欠点がありました。

* 無理矢理拡張しており、内部構造が巨大になってしまったので今後のメンテナンスや拡張が困難

* パフォーマンスが悪い。数万の候補をまともに処理できない

特に `unite.vim` の保守性の悪さはひどく、作者以外はまともにメンテナンスができない上に作者でさえバグ修正が困難となってしまっています。

`unite.vim` のパフォーマンス問題を軽減させるために、`unite.vim` の一部処理を Lua に対応を行いましたが焼け石に水でした。その後 Vim の Lua インタフェースの互換性が壊れる事件により Lua コードは削除されることになります。

`unite.vim` の欠点を解消させるために、今後私は新たなプラグインを開発していくことになるのでした。

Note: 余談ですが、`unite.vim` の名前の由来として UI と統合(unite)という意味があります。最初から UI 作成するためのプラグインであったのです。


## denite.nvim 2016/02 頃開発

https://github.com/Shougo/denite.nvim

`unite.vim` の後継となる `denite.nvim` (dark-powered neo unite) の開発には `unite.vim` から六年もの歳月がかかりました。
なぜここまで時間がかかったかというと、そのころの私は `unite.vim` を拡張したり `unite.vim` を用いたファイル操作プラグイン `vimfiler.vim` を開発していたためです。

`unite.vim` に代わる次のプラグインは必要でしたが、そのためのブレイクスルーはまだありませんでした。

2014 年に neovim が登場し、neovim がリモートプラグインという新たなプラグインアーキテクチャを提唱しました。
保守性とパフォーマンスの悪い Vim のプラグイン機構に不満を持っていた私は、リモートプラグイン機構を用いて Python で `denite.nvim` を実装することになりました。

当初 `denite.nvim` はユーザーの入力を乗っ取ることで独自のイベントループを実装していました。
これには他のプラグインや設定とコンフリクトしにくい利点があったのですが、同時にユーザーが自由に設定や移動ができなかったりする欠点も抱えていました。
その後独自のイベントループは廃止され、filter window を用いた絞り込みとして機能が整理されることになります。

`denite.nvim` ではデフォルトキーマッピングの廃止というものを行いました。
`unite.vim` にて機能が増える度にキーマッピングも増え、デフォルトキーマッピングが混乱した反省によるものです。
ユーザーによって必要な機能は異なるし、使えるマッピングも違います。デフォルトキーマッピングの定義はただの押し付けではないかと思うに至ったのです。
ユーザーが自分で使うキーマッピングを決めることこそ正しい姿です。

`denite.nvim` はパフォーマンスの向上、保守のしやすさ、Python の豊富なライブラリを利用できることで `unite.vim` の上位互換としての地位を確立することができました。
`unite.vim` のときとは異なり `denite.nvim` は比較的ソースコードの見通しがよいので、現在でも普通にメンテナンスできるほど保守がしやすくなっています。

しかし `denite.nvim` も万能ではありません。`denite.nvim` の欠点は以下です。

* Python に依存している

* Python のインストール環境は様々で、環境により不可解なエラーが起こることがあり解決が困難

* 絞り込み処理が非同期化できておらず、処理のボトルネックになっている

* neovim はリモートプラグイン API のメンテナンスではなく、 Lua API の整備を優先しており将来がない

* Vim8 との互換性が低い、Vim8 環境でのパフォーマンスが悪い

`denite.nvim` 開発後期になると、リモートプラグインに代わり Lua で書かれた neovim 用プラグインがプラグイン界隈を圧巻しはじめます。
それはファジーファインダー界隈でも例外ではありませんでした。

`telescope.nvim` は neovim 利用者に圧倒的速度で広まり、現在ではデファクトスタンダードの地位を確立しています。

https://github.com/nvim-telescope/telescope.nvim

しかし私は Lua で新たなプラグインを作成する気はありませんでした。自分が Lua を選択しなかったのは以下が理由です。

* neovim でしか動作しない

* Lua は立場として高速な Vim script であり、処理が固まる問題を解決ができない。Vim9 script も同様の問題を抱える

* ライブラリが貧弱である

* 型をつけられないので堅牢性に問題がある、巨大なプラグインの保守が困難

`denite.nvim` の行く末に不安を感じつつも、次のブレイクスルーが起こるまで `denite.nvim` の開発は停滞することとなったのでした。


## ddu.vim 2021/12 頃開発

https://github.com/Shougo/ddu.vim

`denite.nvim` の後継プラグインを開発するうえで、次のブレイクスルーとなったのは 2021 年に満を持して登場した `denops.vim` です。

https://github.com/vim-denops/denops.vim

`denops.vim` は以下の特徴を持ちます。

* Vim8/neovim 両方に対応

* スクリプト言語のようにつかえて開発がしやすい

* 速度に優れている

* 型があるのでコードが堅牢になる

* ライブラリが豊富

`denops.vim` は私の求めていた要件をほとんど満たしていました。

`denops.vim` を用いて私が作成した自動補完プラグイン `ddc.vim` が成功し、ようやく本命である `ddu.vim` の開発が開始されることになったのです。

https://github.com/Shougo/ddc.vim

`ddc.vim` の開発で得られたノウハウを `ddu.vim` では活用し、より実装を洗練させました。`ddu.vim` はいままでにはない完成度のプラグインとなったのです。


# ddu.vim の特徴

`ddu.vim` は従来のファジーファインダーを過去にするほどのインパクトを秘めています。その特徴をこれから説明していきます。


## Vim8/neovim 両対応、環境依存が少なくインストールが容易

`ddu.vim` は `denops.vim` を用いているので `ddc.vim` のように Vim8/neovim 両方に対応し環境依存も少ないです。
特に Windows 環境なんかでは嬉しい要素ではないでしょうか。

Note: ただし Deno がうまく動作しない特殊環境では使いにくい可能性があります。それでも Python よりは環境構築が容易です。


## ui, source, filter, kind という概念

`ddu.vim` には ui, source, filter, kind という概念が存在します。`unite.vim`, `denite.nvim` にも同様の概念がありますが、一度ここで整理しましょう。

ui とは `ddu.vim` の view を制御する拡張プラグインのことです。

source とは `ddu.vim` の item を生成する拡張プラグインのことです。`anything.el` では情報源と呼ばれています。`telescope.nvim` では picker となっています。

filter とは matcher, sorter, converter をまとめた概念で、 item を絞り込みする、ソートする、加工する機能をもった拡張プラグインのことです。

kind とは item に対して実行するアクションを定義するものです。

例えば、`ddu-source-file_rec` source と `ddu-source-file` source は両者とも file kind の item を生成するので、使用するには file kind のインストールが必要です。
両者の item に対して行えるアクションは基本的に共通となりますが、source 毎にアクションを定義することが可能です。


## UI プラグインによる拡張

`ddu.vim` 独自の概念として UI プラグインというものがあります。
`telescope.nvim` にも theme というものがあり、見た目を変更することができるのでそれとはどう違うのだと思う人もいるでしょう。

`telescope.nvim` の theme とは一部挙動や画面レイアウトを入れ替えるもので、画面設定のプリセットみたいなものでしょう。
theme を入れ替えたところで、見た目が変わるだけでありプラグインの機能そのものが変化するわけではありません。

`ddu.vim` の UI プラグインとは `ddu.vim` の view を「丸ごと」定義するためのものです。
`ddu.vim` は view を自由に入れ替えることができ、`ddu.vim` を内部で用いた派生プラグインを自由に作ることができます。
もちろん、同じフレームワークなのでそれぞれ設定や source には互換性があります。

https://github.com/Shougo/ddu-ui-ff

https://github.com/Shougo/ddu-ui-filer

`unite.vim` では `vimfiler.vim` を `unite.vim` ベースで実装していたり、`denite.nvim` では view のコードを本体と分離して開発しやすくなっていました。
自分が理想としていたものをようやく `ddu.vim` で実装することができたのです。

そもそも、なぜ世の中にファジーファインダープラグインが溢れているのでしょうか。ファジーファインダープラグインの乱立といったら、自動補完やスニペット、ファイル操作プラグインとは比較になりません。
これは UI の好みが人それぞれだからだと思います。

`denite.nvim` はオプションで UI をカスタム可能にすることで問題に一部対処していますが、根本的な解決とはなりませんでした。
ファジーファインダーが乱立すると何が問題なのでしょうか。それは設定や派生プラグインに互換性がなくことです。
UI が違うファジーファインダーに対して、毎回車輪の再発明がなされることになります。これは無駄ではないでしょうか。

例えば `fzf-preview` は fzf を無理矢理拡張して独自の UI を実現しています。

https://github.com/yuki-ycino/fzf-preview.vim

`ddu.vim` だとこの問題は解決しています。
私が作成した `ddu-ui-ff` は `denite.nvim` ライクで標準的なファジーファインダーの UI ですが、UI が気に入らなければ自分で好きな UI をフルスクラッチすればよいのです。
`ddu-ui-filer` では `ddu.vim` をベースにしたファイル操作のプラグインを実装する予定です。これも `ddu.vim` の大きな可能性といえます。


## TypeScript による source 定義

`ddu.vim` は TypeScript で処理を記述することが可能で、型チェックの恩恵を受けることが可能です。
これは堅牢性につながります。
TypeScript はライブラリも豊富なので欲しい機能を簡単に実装できるのも優れています。
ライブラリの依存関係は自動的に解決され、ユーザーに明示的なインストール処理を要求されません。

Note: その代わり初回起動時にはライブラリをインターネットからダウンロードするオーバーヘッドがあります。インターネット接続環境でない場合には向きませんが、そのような環境にそもそもリッチな編集環境が必要なのだろうかと考えるべきでしょう


## 非同期処理

`denite.nvim` ではリモートプラグインとマルチプロセス処理により非同期処理を実現していました。
しかし、それでも絞り込み処理など一部の処理が非同期にできずパフォーマンスに問題を抱えていました。

`ddu.vim` は TypeScript を用いることで簡単に非同期処理が実装でき、ありとあらゆる処理が非同期で実行されるので「一部の処理が非同期にできない」ことはありません。


## 改善されたパフォーマンス

`ddu.vim` の起動速度は Deno の起動時間があるので、それを含めて考える必要がありますがワーストケースでも `denite.nvim` と同等です。
`ddu.vim` や `denops.vim` を遅延起動させる場合には初回起動時間に注意が必要です。
とはいえ、`denite.nvim` は初回の Python のロードが遅く、`unite.vim` も大量の Vim script を起動時にロードする必要があり起動が軽快とはいえませんでした。
`ddu.vim` は他のプラグインにより Deno が既に起動している状態ならば圧倒的な速度で起動します。

`ddu.vim` は `denite.nvim` よりも候補の取得速度が高速です。しかし `denite.nvim` も候補取得は非同期化されておりパフォーマンスもチューニングしているので特別遅いわけではないです。
両者の比較結果は以下のようになります。

```
denite.nvim file/rec(scantree.py) 10.86 秒  40 万ファイル
denite.nvim file/rec(rg) 17.72 秒  64 万ファイル
ddu.vim file_rec(ネィティブ) 15.85 秒 98 万ファイル
```

ざっくりと比較すると、`ddu.vim` の候補取得は `denite.nvim` の 1.5 倍高速です。しかも `denite.nvim` より安定しているので `denite.nvim` を使う理由がもはやありません。

Note: ただし、`ddu.vim` でも候補の取得中に強制終了させるとクラッシュするので注意が必要

`ddu.vim` は TypeScript を用いることで大幅な高速化に成功しましたが、さすがにネイティブコードを用いた各種ファジーファインダーと比較するとパフォーマンスではやや劣ります。
しかしネイティブコードを用いるデメリットとして、どうしても柔軟性が下がります。クライアントにはビルド環境が必要ですし、プラグインに拡張機能を同梱することも非常にやりづらいでしょう。
`ddu.vim` の利点はスクリプト言語の柔軟性がありつつ、パフォーマンスに優れていることにあるかと思います。
ネイティブコードを用いるとファジーファインダーの抱える全ての問題が解決というほど甘くはないのです。

Note: 例えば `linearf` プラグインはネイティブコードを用いており動作が `ddu.vim` よりも高速ですが、更新の度に全ての拡張プラグインを一度ロードして一つにしてビルドしなければいけません。遅延起動したプラグインとは壊滅的に相性が悪いです。
`ddu.vim` では拡張プラグインを必要なときのみ動的にロードするので遅延起動も問題なく対応できます。

https://github.com/octaltree/linearf


## ui, source, filter, kind の分離

`ddu.vim` では ui, source, filter, kind の全ての機能が分離されており、ユーザーが好きな機能を選択することができます。
もちろんこれには欠点があり、`ddu.vim` プラグイン本体をインストールするだけだと使い始めることはできません。
ユーザーは各種プラグインの機能を理解して、必要なものを個別にインストールする必要があるのです。

`ddu.vim` とは異なり、本体にほとんど全てを同梱しているファジーファインダーは多くあります。
そもそも私が開発した `unite.vim`, `denite.nvim` がそうですし、`fzf-preview` や `telescope.nvim` もそうです。
インストールするプラグインが少なくて済むのでユーザーとしては便利ですが、本体が肥大化するのが問題になっています。`telescope.nvim` ではビルトイン機能を分離する話も出てきているほどです。
とはいえ、後から分離するとユーザーの反発も大きいでしょうし容易ではありません。`ddu.vim` のように最初から分離するべきであったと言えるでしょう。

https://github.com/nvim-telescope/telescope.nvim/issues/1228

機能を分離することにはメンテナンス上の理由があります。
本体と各種機能が密結合することがなくなり、責任が明確になり issues や Pull requests が汚れることを防いでくれます。
これは長いメンテナンスを考えるととても重要です。プラグインは一度作って終わりではありません。


## 設定項目や設定方法の整理

`ddu.vim` の設定には変数は一切存在せず、すべての設定は以下の custom API を用いて行うことになります。

* グローバル設定 (`ddu#custom#patch_global()`)
* ローカル設定 (`ddu#custom#patch_local()`)

グローバル設定がデフォルトの設定の上書きで、ローカル設定は特定の name が設定された ddu セッション固有の設定となります。
これは `denite.nvim` でいう `denite#custom#option()` にて buffer_name を指定したときに相当します。

更に `ddc.vim`, `ddu.vim` では options, params という独自の概念があります。

options とは、それぞれの source, filter, ui, kind に共通した汎用的な設定です。

params とは、個別の source, filter, ui, kind 特有の設定です。

options は本体により既定値が決まっており、拡張プラグインがその値を変更することはできません。
デフォルト値を上書きできるのはユーザーのみとなります。これは拡張プラグインが値を上書きすることで、ユーザーが意図しない挙動をしないようにするための措置です。
`ddu.vim` では、拡張プラグイン側が何か推奨する設定をユーザーにしてほしいと思った場合、ドキュメントにてサンプルを提示することが推奨されます。
ユーザーがその指示に従うかどうかは自由です。ユーザーが選択できることこそが価値なのです。

params については拡張プラグインが自由に初期値を設定することができ、ユーザーがそれを上書きすることができます。
params の初期値についてはドキュメントに記述することが推奨されます。


## コマンドの分離

`ddu.vim` では `denite.nvim` とは異なり、コマンドも `ddu-commands.vim` という別プラグインに分離しました。
これはなぜかというと、コマンドはシンタックスシュガーというべきもので誰もが必要なものではないし、必要ならばユーザーが自由に作るべきものだと考えているからです。

https://github.com/Shougo/ddu-commands.vim


# 廃止した機能について

`ddu.vim` では `ddc.vim` とは異なり意図的に廃止した機能は少ないです。


## クイックマッチ機能

使用頻度が低い割に実装が大変なため。


## source の is_interactive フラグ

source の `is_interactive` フラグは source の候補取得が入力に依存することを表すものです。
`ddu.vim` では入力に応じて候補を refresh する場合、ユーザーが明示的に `volatile` オプションを用いるので不要です。


## source の is_async フラグ

source の `is_async` フラグは source の候補取得が非同期であるかを表すものです。
ddu.vim では `ReadableStream` を用いて非同期に候補を更新するので不要となりました。

https://developer.mozilla.org/ja/docs/Web/API/ReadableStream


# 自分では実装しない機能について

自分が必要な機能は `ddu.vim` に一通り実装していますが、諸事情により自分では実装しない機能について説明します。ユーザーは独自に実装できます。


## メニュー機能(:Denite menu)

`denite.nvim` にはメニュー機能があります。
しかし自分が使用していないので `ddu.vim` では実装しないことにしました。
ただし `ddu.vim` には実行中の source を切り替える機能が存在するので、ユーザーがメニューを独自に実装することが可能です。


# ddu.vim の使用方法

詳しい使用方法はドキュメントを参照してください、としたいのですが簡単に説明をしておきます。

前述の通り、`ddu.vim` の導入には Deno と `denops.vim` が必須となります。先に導入をしておいてください。

- <https://deno.land/>
- <https://github.com/vim-denops/denops.vim>

ここではプラグインと依存関係はすでにインストール済み、ロード済みとして話を続けます。

必須の依存関係のほかにも、`ddu.vim` ではデフォルトの ui, source, filter, kind を廃止しているため、それらのインストールが必須となります。
自分が必要となるものをインストールしましょう。以下のリンクから探すと良いです。

https://github.com/topics/ddu-ui
https://github.com/topics/ddu-source
https://github.com/topics/ddu-filter
https://github.com/topics/ddu-kind

ここでは、比較的一般的なものである `ddu-ui-ff`, `ddu-source-file_rec`, `ddu-filter-matcher_substring`, `ddu-kind-file` を使用するものとします。

https://github.com/Shougo/ddu-ui-ff
https://github.com/Shougo/ddu-source-file_rec
https://github.com/Shougo/ddu-filter-matcher_substring
https://github.com/Shougo/ddu-kind-file

Note: ちなみに、`ddu-source-file_rec` は `ddu-kind-file` に依存しているので `ddu-source-file_rec` を利用するなら `ddu-kind-file` のインストールは必須です。

`ddu.vim` の設定は以下のように行います。

```vim
call ddu#custom#patch_global({
    \   'ui': 'ff',
    \   'sources': [{'name': 'file_rec', 'params': {}}],
    \   'sourceOptions': {
    \     '_': {
    \       'matchers': ['matcher_substring'],
    \     },
    \   },
    \   'kindOptions': {
    \     'file': {
    \       'defaultAction': 'open',
    \     },
    \   }
    \ })

call ddu#start({})
```

`ddu#custom#patch_global()` によりグローバル設定を変更します。

`ui` オプションに `ff` を指定することにより、`ddu-ui-ff` を読み込むという意味になります。

デフォルトでは source は何も指定されていないため、`file_rec` source を利用するために `sources` オプションを設定しています。
ちなみに、`ui`, `sources` `matchers`, `sorters`, `converters` は名前のチェック機能が存在しており、インストールしていないものを指定するとエラーになります。

`sourceOptions` により source 固有の設定をします。source 名に `_` を与えることでデフォルトの設定を変更します。ここでは入力の絞り込みを行う `matchers` として `matcher_substring` をセットしています。

`ddu.vim` 特有の設定として、kind のデフォルトアクションをユーザーが指定する必要があります。ここでは選択したファイルを開く `open` アクションを指定しています。

`call ddu#start({})` を実行すると、デフォルトの設定で source を起動します。この場合は `file_rec` source が起動して以下のような表示になるはずです。

![ddu.vim](/images/ddu.png)

`ddu-ui-ff` にはデフォルトキーマッピングが存在しないので、`ddu-ui-ff` のウインドウが表示されてもこれだけでは操作ができません。
以下のようにキーマッピングを設定する必要があります。これは `denite.nvim` でも同様の仕様のため、 `denite.nvim` の設定に慣れている人ならば理解しやすいでしょう。

```vim
autocmd FileType ddu-ff call s:ddu_my_settings()
function! s:ddu_my_settings() abort
  nnoremap <buffer><silent> <CR>
        \ <Cmd>call ddu#ui#ff#do_action('itemAction')<CR>
  nnoremap <buffer><silent> <Space>
        \ <Cmd>call ddu#ui#ff#do_action('toggleSelectItem')<CR>
  nnoremap <buffer><silent> i
        \ <Cmd>call ddu#ui#ff#do_action('openFilterWindow')<CR>
  nnoremap <buffer><silent> q
        \ <Cmd>call ddu#ui#ff#do_action('quit')<CR>
endfunction

autocmd FileType ddu-ff-filter call s:ddu_filter_my_settings()
function! s:ddu_filter_my_settings() abort
  inoremap <buffer><silent> <CR>
  \ <Esc><Cmd>close<CR>
  nnoremap <buffer><silent> <CR>
  \ <Cmd>close<CR>
  nnoremap <buffer><silent> q
  \ <Cmd>close<CR>
endfunction
```

`ddu-ui-ff` は filetype `ddu-ff` のバッファーを生成するので、それを利用してキーマッピングの設定を行います。

`ddu#ui#ff#do_action()` は UI 固有のアクションを実行するための機能です。`denite.nvim` でいうと `denite#do_map()` に相当します。

`itemAction` は選択した item またはカーソル上の item の item アクションを実行する機能です。`denite.nvim` でいうと `do_action` に相当します。
このアクションは引数に item アクション名をとりますが、省略するとデフォルトアクションとなります。

`ddu-ui-ff` において、絞り込みを行うには `openFilterWindow` アクションで filter window を開かなければいけません。
これは `denite.nvim` と同じ仕様となります。filter window を開くと以下のようになります。

![ddu-filter.vim](/images/ddu-filter.png)

filter window のキーマッピングは `FileType ddu-ff-filter` autocmd で設定します。
`denite.nvim` とは異なり、`ddu-ui-ff` の filter window にはデフォルトで何もマッピングがされていません。

`ddu.vim` において特に注意しないといけないことは、UI の設定は全て `uiOptions` や `uiParams` に記述するということです。
ユーザーはそれが何の設定であるか把握しておく必要があります。
例えば、neovim の floating window 機能で `ddu-ui-ff` のウインドウを表示したい場合以下のように設定をします。

```vim
call ddu#custom#patch_global({
    \   'uiParams': {
    \     'ff': {
    \       'split': 'floating',
    \     }
    \   },
    \ })
```

`split` は `ddu-ui-ff` ウインドウの分割設定を変更する `ddu-ui-ff` 固有の設定なので `uiParams` の `ff` をキーに設定しています。

これで最低限の解説は終了です。基本は分かったはずですので、あとは `ddu.vim` を設定していきながら学んでいきましょう。


# これからのプラグイン開発について

`ddu.vim` の開発は一通り完了し、自分が日常的に使う上で困らなくなりました。`ddu.vim` の今後は機能追加より機能の安定化に注力し、正式版のリリースを目指します。
その後は `defx.nvim` の後継プラグインを `ddu.vim` の UI プラグインとして開発する予定です。

https://github.com/Shougo/ddu-ui-filer


# GitHub sponsors について

今回の `ddu.vim` プラグインの開発は GitHub sponsors の皆さんの支援によって行われました。
何年もかけて開発してきたプラグインを作り直すのはとてもエネルギーの必要な作業です。
皆さんからの支援がなければ、到底実現できなかっただろうと思います。

https://zenn.dev/shougo/articles/github-sponsors

https://github.com/sponsors/Shougo/

GitHub sponsors での支援は確実に自分のプラグインの開発意欲の向上に役立っているので、プラグインのユーザーに広く支援をお願いします。
