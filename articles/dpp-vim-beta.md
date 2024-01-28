---
title: "プラグインマネージャーの歴史と新世代のプラグインマネージャー dpp.vim"
emoji: "🖤"
type: "tech"
topics: ["vim", "neovim", "denops"]
published: true
---

# 始めに

`ddc.vim`, `ddu.vim` の開発が一通り終了し、次に作成するプラグインについて考えていました。
バイナリ編集プラグイン `ddx.vim` の開発を進めることも考えていたのですが、`dein.vim` の開発から長い時間がたっており新たなプラグインマネージャーも出てきているので、そろそろ作り直すべきではないかと考えました。
プラグインマネージャーの機能がどんどん複雑化する昨今、プラグインマネージャーに必要な機能とは何か考えたときに、「拡張可能なプラグインマネージャーが求められている」と思ったのです。

そして今回作成したプラグインマネージャーは `dpp.vim` です。

https://github.com/Shougo/dpp.vim

`dpp.vim` は作成したばかりで完成度はまだ低く、`dein.vim` ユーザーが乗り換える機能のレベルや安定性にはなっていません。
しかし、その設計思想とインタフェースは十分固まったといえるので今回記事を書くことにしました。

::: message
当時の完成度はまだまだでしたがその後、開発が進んだことで `dpp.vim` はようやくユーザーに推奨できるプラグインマネージャーへと成長しました。

ちなみに、私が以前に作成したプラグインマネージャーである `dein.vim` は既に開発を終了しました。
:::


# プラグインマネージャーの歴史

私は Vim キャリアが長いので、これまでいくつかのプラグインマネージャーを使ってきましたし、開発してきました。
ここではプラグインマネージャーの歴史を振り返ってみることにしましょう。

::: message
私のテキストエディタのキャリアは 15 年以上です。これはもうベテランと呼んでしまってもよいでしょう。
リアルタイムでテキストエディタの進化を経験してきたため、歴史の生き証人となりつつあります。
:::


## プラグインマネージャーを使わなかった時代 (2008 年以前)

Vim プラグインが本格的に作られるようになったのはスクリプト機能が強化された Vim 7.0 がリリースされた 2006 年頃です。
私が Vim を触り始めたのも丁度この時期でした。

その当時、プラグインマネージャーというものは存在しませんでした。
現在では考えられないと思うのですが、`~/.vim` にプラグインを手で放り込むということが行われていたのです。
当時のプラグインは plugin ディレクトリに .vim ファイルが一つしかないシンプルな構成が多く、プラグインの数も少なかったのでそれで済んだのでした。

::: message
当時には `GetLatestVimScripts` というものが存在していました。
これはプラグインマネージャーのようにプラグインのインストールやアップデートを簡単にしてくれるのでプラグインマネージャーの先駆けと言えるでしょう。
ただし、私は `GetLatestVimScripts` を使っていませんでした。
当時は Git が登場したばかりでもちろん github が存在しなかったため、`GetLatestVimScripts` は vim.org にアップロードされているプラグインファイルをダウンロードしてくれる仕組みとなっています。
vim.org はプラグインのハブサイトだったわけです。

https://github.com/vim-scripts/GetLatestVimScripts
:::

::: message
余談ですが当時私が開発したプラグインである `neocomplcache.vim` は Vim プラグインとして破格の複雑さであり、プラグインマネージャーなしでは管理が困難であると思います。
当時のプラグイン水準からすると明らかに機能が Ultimate であり、その名は伊達ではありません。
他に優れた自動補完プラグインが多数登場している現代では使用を推奨しないのですが、提供している機能だけなら現在でも十分に通用します。

https://github.com/Shougo/neocomplcache.vim
:::


## vim-pathogen (2008/10 頃)

https://github.com/tpope/vim-pathogen

2008 年になると `vim-pathogen` というプラグインマネージャーが使われるようになりました。
`vim-pathogen` のプラグイン管理スタイルは従来の管理のように特定のディレクトリに全てを放り込むのではなく、プラグインをサブディレクトリに分けます。
それぞれのサブディレクトリにあるプラグインを全て読み込む仕組みとなっています。
現代でいうと `node_modules` や `pip` の管理方法に近いでしょうか。それぞれのプラグインのディレクトリは独立しており、別のプラグインのファイルが混ざらないので管理がしやすくなっています。

当時 github が出たばかりであったため、`vim-pathogen` は git, github 前提のプラグインマネージャーとまではいえないでしょう。
しかしプラグインが大きくなり数も多くなると手で管理できなくなったので `vim-pathogen` が登場したのは想像に難くありません。
特定ディレクトリにプラグインを `git clone` すれば比較的簡単にプラグインをインストールできるのは魅力といえます。

::: message
手でプラグインを一つずつ `git clone` するのはやってられない、現代の感覚だとそう思うことでしょう。
しかし、当時はテキストエディタにプラグインを 50 個や 100 個インストールするということは考えられなかったので、`git clone` により手でインストールして管理するというその思想はおかしくはないのです。
現代でも `vim-pathogen` で十分という人がいますが、そういう人は間違いなく少数のプラグインしか導入していません。
概ねプラグイン数 10 個から 20 個くらいまでは `vim-pathogen` でも十分管理可能なのです。
:::

`vim-pathogen` は原始的なプラグインマネージャーです。
名前から分かるようにプラグインのロードパスの管理だけを目的としており、それ以外のことはほとんど何もしません。

`vim-pathogen` は Easy ではありませんが Simple であり、理解がしやすく拡張も容易です。私は `vim-pathogen` をベースにしているプラグインマネージャーが幾つもあったことも知っています。
後にこの思想は Vim の `packages` 機能として本体に取り込まれます。


## Vundle.vim (2010/10 頃)

https://github.com/VundleVim/Vundle.vim

2010 年から 2011 年にかけて、一世を風靡したのは `Vundle.vim` というプラグインマネージャーでした。

`Vundle.vim` が広く使われるようになったのは、「プラグインマネージャーにインストール機能を統合した」ためでしょう。
これは明らかに git, github を前提とした仕組みです。面倒なインストール作業を git, github 前提にすることで簡単にしたため、爆発的に流行しました。

`Vundle.vim` によりプラグインインストールが簡単になったのはその時代背景が大きく関係しています。
この頃にはプラグインを大量にインストールすることが増えてきており、数十ものプラグインをインストールすることも普通となりました。
この量を手で `git clone` するのは現実的ではありません。インストールの自動化は必須となりました。
インストールを自動化しておくと、他の環境でも同じプラグインセットを使い始めることが簡単にできるため、そういう意味でも便利な存在でした。

`Vundle.vim` において特徴的な機能は `{name}/{repo}` という記法でプラグインをインストールできるということです。これは github を Vim プラグインの標準リポジトリとしてみなしているわけです。

::: message
当時は github と vim.org の両方でプラグインが配布されているような過渡期であったので、
`Vundle.vim` において `{repo}` は `vim.org` のプラグインを表すという意味がありました。その文化は `vim.org` でのプラグイン配布が下火になってしまったことにより現代ではなくなりました。
:::


## neobundle.vim (2011/09 頃)

https://github.com/Shougo/neobundle.vim

`neobundle.vim` は当時広く使われていた `Vundle.vim` をベースに私が最初に開発したプラグインマネージャーです。

`neobundle.vim` は遅延起動にこだわって設計されました。乱暴に表現すると、`Vundle.vim` に遅延起動を追加したのが `neobundle.vim` です。
なぜ `neobundle.vim` はプラグインの遅延起動を導入したのでしょうか。
これは当時プラグインのインストール数が爆発的に増えたことに関係しています。
Vim はプラグインを起動時に全てロードしてしまうのでプラグインが増えると Vim の起動時間が増えてしまうのです。
かといって、ユーザーは起動時間を削減するために使うプラグインを削減したくないでしょう。
そこでプラグインマネージャーに遅延起動という概念が導入されました。

プラグインの遅延起動とは、プラグインをプラグインを使うときにだけロードするように設定することです。
Vim 標準である `autoload` に似た仕組みですが、プラグイン単位でロード管理する違いがあります。
プラグインの遅延起動をうまく設定することで、プラグインを読み込まない Vim と遜色ない起動時間にすることが可能です。
もちろんプラグインの遅延起動には設定の複雑化というトレードオフがあります。遅延起動を正しく設定するにはユーザーの深い知識が必要で簡単に設定できるものではありません。
プラグインのロード時間のチューニング方法の一つと考えてもらえば結構です。

::: message
余談ですが当時はプラグインマネージャーにプラグインの遅延起動という概念はほとんどありませんでした。
唯一 `vim-unbundle` というプラグインマネージャーが FileType 毎にロードする機構を備えていたと記憶しています。

https://github.com/sunaku/vim-unbundle

つまり `neobundle.vim` は本格的に遅延起動を採用したプラグインマネージャーの先駆けでした。
遅延起動を実現するために Vim 本体に機能を追加したりもしました。
その後、プラグインマネージャーではプラグインの遅延起動が当然のようになっていくのは皆さんが記憶にある通りです。
正しい歴史の流れを知ることはとても大事です。
:::


## vim-plug (2013/09 頃)

https://github.com/junegunn/vim-plug

`Vundle.vim` の開発が停滞していたころ、2013 年 9 月に現れたのは `vim-plug` でした。
`vim-plug` が独自に行ったこととして大きいのはプラグインの大量並列インストール(プラグインインストールの高速化)、インストール UI の改良でしょう。
この変更がなぜ行われたのか想像してみるに、`Vundle.vim` の時代よりも更にプラグインのインストールが増えてきたということがあると思います。
もう当時はプラグインを 100 個 200 個導入しているという状態は普通となっていました。
プラグインを沢山インストールしていると、当然プラグインのアップデート時間はかかるわけです。インストーラーの並列化は必須であったでしょう。

`vim-plug` の面白い思想としてはプラグインマネージャーをシンプル化し、1 ファイルで配布することでインストールを容易にしようというものが挙げられます。
1 ファイルで配布されている場合、次のようにファイルをダウンロードするコマンド一つでプラグインマネージャーを導入することができるわけです。

```sh
$ curl -fLo ~/.vim/autoload/plug.vim --create-dirs https://raw.githubusercontent.com/junegunn/vim-plug/master/plug.vim
```

`vim-plug` が開発されてから既に 10 年ほど経ち開発が落ちついていますが、今だにファンが多いプラグインマネージャーです。

::: message
余談ですが、`vim-plug` は `A minimalist Vim plugin manager.` を謳っています。
私の目から見ると `vim-plug` が標準で提供している機能は十分に多く、全く `minimalist` には見えないのですが、これもプラグインマネージャーが複雑化したということなのかもしれません。
:::


## dein.vim (2015/12 頃)

https://github.com/Shougo/dein.vim

`neobundle.vim` を長年開発した結果、機能と実装が複雑化し開発の継続が困難になってきました。一度機能の整理をしようと思いフルスクラッチで開発したのが `dein.vim` となります。
`dein.vim` の特徴としては設定をユーザーに委ねること、UI はシンプルであること、ユーザーが設定をチューニングできるようにプラグインマネージャーが余計なことをしないことです。

`neobundle.vim` では遅延起動によりパフォーマンスをチューニングするとができましたが、`dein.vim` ではより細かいプラグインマネージャーの挙動をチューニングできるようにしています。
`dein.vim` は `neobundle.vim` よりも更に機能が多いのでちゃんと設定するのは難しいです。適切な設定さえできればプラグインマネージャー最速クラスの速度を叩き出すことが可能です。

私は現在でも `dein.vim` を利用していますし、機能面での不満はないです。


## Vim packages 機能(2016/09 頃正式リリース)

`:help packages`

Vim 8.0 の目玉機能の一つとして、Vim にパッケージ管理機能(`packages`)が追加されました。

ちなみに、`packages` 機能はプラグインパスを管理するだけでありプラグインをインストールする方法は存在していません。
プラグインインストールは本体に入れるには複雑であり、プラグインをインストールする標準リポジトリも存在しないためです。

::: message
Emacs の標準パッケージ機能(`package.el`)はインストーラーも内蔵しているのですが、こちらはプラグインインストールのためのリポジトリ(elpa)も自前で用意したという違いがあります。
プラグインインストールの実現は皆さんが思うほどに簡単ではないのです。
:::

要するに `vim-pathogen` 相当の機能が Vim 本体に組込まれたわけです。本体はあくまでバックエンドを提供し、プラグインマネージャーのフロントエンドは自由に選択できるようになったとも言えます。

`packages` 機能はプラグインの読み込みについて癖があります。これは本体で実装することを考えると仕方のない部分です。
例えば vimrc の読み込みのあとに `'runtimepath'` が設定されるため、vimrc 内で autoload 関数を読み込めません。

https://zenn.dev/kuu/articles/vim-pack-attention

::: message
余談ですが dein.vim はあえて Vim の packages 機能を使わずに実装しています。
これは packages によるプラグイン読み込みに癖があることも理由の一つとなります。
:::


## minpac (2017/02 頃)

https://github.com/k-takata/minpac

Vim 本体への `packages` 機能の実装により、`packages` を利用した幾つものプラグインマネージャーが開発されていきます。

`minpac` は `packages` 機能を用いてシンプルに実装されたプラグインマネージャーです。
Vim の `packages` 機能を使いやすくしたフロントエンドともいえます。
`packages` 機能が好きならよい選択肢だと思います。


## packer.nvim (2017/11 頃)

https://github.com/wbthomason/packer.nvim

`packer.nvim` は `vim-plug` を Lua で再実装することを目指したプラグインマネージャーです。
Lua で書かれていることから当然 neovim 専用となります。

`packer.nvim` は Lua で書かれた neovim プラグインと相性がよいのが特徴です。neovim の事実上標準プラグインマネージャーとして愛用者が多くいました。
ただし `minpac` と同じく `packer.nvim` の実装には Vim packages 機能を用いており、プラグインの読み込みに独特の癖があるため注意が必要です。

::: message
`packer.nvim` は 2023 年 8 月に開発終了がアナウンスされました。精神的後継プラグインは `https://github.com/lewis6991/pckr.nvim` となるようです。
:::


## vim-jetpack (2022/02 頃)

https://github.com/tani/vim-jetpack

`vim-jetpack` の特徴は `vim-plug` と同じく 1 ファイルのみでインストール可能であること、簡単に使えることにあります。
プラグインマネージャーとしては珍しく、`'runtimepath'` のマージ機能に対応しています。

::: message
`'runtimepath'` のマージ機能とは、遅延起動しないプラグインのインストール箇所をプラグインマネージャーを使っていなかったときのように一箇所にまとめることをいいます。
`'runtimepath'` が短かくできるので多少プラグイン読み込みの高速化を見込めます。`'runtimepath'` のマージ機能は元々 `dein.vim` が実装していた機能です。
:::

`vim-jetpack` は `dein.vim` と比較するとチューニングできるところが少ない代わりに初心者には分かりやすいと思います。


## lazy.nvim (2022/11 頃)

https://github.com/folke/lazy.nvim

最近登場した新進気鋭のプラグインマネージャーです。開発が停滞している `packer.nvim` の後継として彗星のように現れました。
その特徴は UI のスタイリッシュさと起動時間のチューニングにあると思います。名前の通り neovim 専用なのは Vim ユーザーにとっては残念といえるでしょうか。

`lazy.nvim` はある意味でプラグインマネージャーの一つの到達点であり、今現在最も開発が盛んなプラグインマネージャーといってもよいでしょう。


## dpp.vim (2023/09 頃)

https://github.com/Shougo/dpp.vim

そして私が今回開発したのが `dpp.vim` です。
`dpp.vim` はプラグインマネージャーとしては最後発にあたり、他のプラグインマネージャーにはない思想で設計されています。
その特徴的な機能については次のセクションにて詳しく説明します。

私は `dpp.vim` を作るにあたって、ユーザーがコードを書けば何でもできるということをコンセプトにしました。
既存のプラグインマネージャーはなんでもやってくれる代わりに、ユーザーにコードを書かせないようになっています。
これはいけません。プラグインマネージャーはユーザーの要望を叶えてしかるべきです。ユーザーは自分の要望のためならコードも書くでしょう。
コードを書かない人は `dpp.vim` のユーザーではありません。
プラグインマネージャーを使うのはテキストエディタの素人ではなく、コードを書けるプログラマーなのですから。

::: message
私はプラグインを作成するにあたり思想を重要視しています。
その思想によってプラグインはなにをやるべきか、なにをやらないべきかが明確になるのです。
思想に合わないプラグインを使い続けるのは苦痛です。ユーザーは自分の思想にあったプラグインを使用するべきでしょう。
:::

::: message
`dpp.vim `の完成度は現状不十分であり、他のプラグインマネージャーに機能は劣ってしまいます。
しかし `dpp.vim` の速度には自信があります。実装は最適化されており、無駄な処理をしないためあの `dein.vim` をも越えるポテンシャルがあるのです。
:::

# dpp.vim の特徴

`dpp.vim` の特徴は以下の通りです。

* ほとんど全ての機能をコアから分離する

* プラグインのインストール機能は拡張機能として提供

* TypeScript と Vim script のハイブリッド実装

* 変数の廃止、オプションの動的変更は不可

* Deno に依存する代わりに環境依存は最小限


## ほとんど全ての機能をコアから分離する

私が `dpp.vim` で目指したのは機能のモジュール化と分離です。

`dpp.vim` は `dein.vim` の備えるほとんどの機能を拡張機能として実装しています。
例えば以下のようなものが挙げられます。

* プラグインのインストール(`dpp-ext-installer`)

https://github.com/Shougo/dpp-ext-installer

* toml サポート(`dpp-ext-toml`)

https://github.com/Shougo/dpp-ext-toml

* ローカルプラグインの読み込み(`dpp-ext-local`)

https://github.com/Shougo/dpp-ext-local

* git サポート(`dpp-protocol-git`)

https://github.com/Shougo/dpp-protocol-git

`dpp.vim` はデフォルトでは何も動作しないことを目指します。`vim-plug`, `vim-jetpack` の「簡単にインストールでき、すぐに使い始められるプラグインマネージャー」とは真逆に位置する思想となります。
`dpp.vim` の思想は easy ではなく究極の simple なのです。easy なプラグインマネージャーが必要な人には既に沢山の選択肢があるので問題にはなりません。

もちろん、この決断によってプラグインマネージャーのインストールは少し大変になるかもしれません。
しかし、プラグインマネージャーのインストールはそんなに頻繁に行うものでしょうか。インストール手順が複雑ならスクリプト化すればよいのではないでしょうか。
`dpp.vim` はテキストエディタの素人が使うことを意図していないし、プラグインマネージャーのインストールは簡単にする必要はありません。

プラグインマネージャーの歴史を見れば分かる通り、それは複雑化の歴史です。
この理由は分かっています。ユーザーにとって必要な機能が異なるので、それらの要望を全て叶えようとするとユーザーが増加するにあたって無限に機能が増えて複雑化していくのです。
そして最終的には開発が放棄され、次のプラグインマネージャーが開発されるという歴史を繰り返しています。
もちろん、プラグインマネージャーの一部にはシンプルさを売りにしたものも存在しています。しかしユーザーによって必要な機能が異なるので、機能を単純に削ぎ落とすことには問題があります。

`dpp.vim` ではコア機能とそうでない機能が分離されていることにより、ユーザーは自分に必要な機能のみをインストールすればよくなります。ユーザーは足りない機能を自分で作ってもよいし、自由自在です。
もちろん、プラグインマネージャーをインストールする前に考えることは増えています。しかしこれこそが楽しいのではないでしょうか。あなたは自由を手にいれたのです。

`dpp.vim` では `dpp.vim` と拡張機能のみで既存のプラグインマネージャーの機能を再現することが可能です。

例えば、

* `dpp.vim` + `dpp-ext-local`: `pathogen.vim` 相当

* `dpp.vim` + `dpp-ext-installer`: `Vundle.vim` 相当

* `dpp.vim` + `dpp-ext-installer` + `dpp-ext-lazy`: `vim-plug` 相当

* `dpp.vim` + `dpp-ext-installer` + `dpp-ext-lazy` + `dpp-ext-local` + `dpp-ext-toml`: `dein.vim` 相当

拡張機能をインストールするだけでシンプルなプラグインマネージャーとしても、複雑なプラグインマネージャーとしても振る舞うことのできる `dpp.vim` はある意味で最強のプラグインマネージャーと呼べるでしょう。

::: message
余談となりますが、巷でよく言われる minimal とは本当の意味での minimal といえるのでしょうか。
「作者にとっての minimal」を真の minimal と勘違いしている例は非常に多くあると考えています。
作者にとっての minimal は他の人にとっても本当に minimal でしょうか。そうではないから、巷には自称 minimal のプラグインマネージャーが溢れているのです。
私の考える minimal とは「作者であっても設定を十分にしなければ使いものにならないほどコアの機能が少なく minimal」という意味です。
`dpp.vim` のコア部分には本当に最小の機能しかなく共通で使用されるインタフェースを定義しているだけです。
これに各々のユーザーが好きな拡張機能をインストールすることで「誰にとっても最小構成の minimal」を実現することが可能です。
:::

みなさんは既存のプラグインマネージャーに不満があったりしませんか。
自分に不必要な機能がプラグインマネージャーに含まれていることが気になりませんか。
一部の挙動を修正するためにプラグインマネージャーを fork するなどしていませんか。
おめでとうございます。その問題は `dpp.vim` で解決できます。`dpp.vim` では既存の機能や拡張機能に不満があればあなた自身で拡張を作ってしまえばよいのです。

「ユーザーはコードを書かないというこれまでの常識」から解き放たれ、プラグインを設定するという自由を手に入れましょう。

::: message
`dpp.vim` の拡張可能なプラグインマネージャーという思想は前例がありません。これはなぜでしょうか。
一つに、設計が難しいことが挙げられます。「プラグインをインストールする」という一見簡単な作業についても実装にあたり汎用的なインタフェースを考えなくてはいけません。
他には「モノリポジトリで全て一つになっていたほうが実装が楽」、「コードを管理しやすいし実装を再利用しやすい」、「インタフェースも変更しやすい」からです。
現代のプロジェクト開発でモノリポジトリが重宝されているのを見ると想像は付くのではないでしょうか。
:::

::: message
余談となりますが、`ddc.vim`, `ddu.vim`, `dpp.vim` では拡張機能にプラグインが名前を直接設定することはできず、名前はファイル名から自動的にセットされます。
これはなぜかというと、名前を拡張機能側でセットするようにするとユーザーが拡張機能のソースコードを確認しなければ正確な名前を知ることができないからです。
「拡張機能の設定を絶対にしなければならないプラグイン」において、これは避けなければならないことです。
もちろんドキュメントに名前が書いてある可能性はありますが、私はドキュメントが用意されることとその正確性を信用していないのです。
:::


## プラグインのインストール機能は拡張機能として提供

`dpp.vim` の機能分離の一環として、インストール機能を分離することにしました。
現代ではプラグインマネージャーにインストール機能は必須のように考えられています。
しかし太古のプラグインマネージャーである `pathogen.vim` や Vim のパッケージ管理機能がそうであるように、プラグインのインストール機能というのはプラグインマネージャーにとって必須ではありません。
更に、プラグインのインストール機能というのは実装が複雑になりがちです。

`dpp.vim` ではインストール機能を `dpp-ext-installer` として分離することにより、コアの機能をシンプルにします。

例えば、`dein.vim` では以下のようにプラグインをインストールしていました。

`call dein#install()`

同じことを `dpp.vim` では以下のように行います。

`call dpp#async_ext_action('installer', 'install')`

`dein.vim` の場合は専用の API を用意してプラグインのインストール作業を行います。

`dpp.vim` の場合は専用の API というものはなく、あくまでも汎用的な拡張機能の呼び出し(`dpp#async_ext_action()`)により全ての処理を実装しています。
`installer` 拡張機能(`dpp-ext-installer`) の `install` アクションを呼び出すことでプラグインのインストール作業を行います。
私はこちらのほうが設計が美しいと思いました。あなたはどう思うでしょうか。


## TypeScript と Vim script のハイブリッド実装

プラグインマネージャーの開発に用いる言語の選択はとても難しいものでした。
最初から実装に Lua や Vim 9 script を使うことは選択肢にありませんでした。
Lua だと neovim 専用になってしまうし、既存のプラグインマネージャーが既に存在しています。
Vim 9 script は neovim で動作しない上に未完成、今後の開発にも不安が残ります。

そこで白羽の矢をたてたのが `denops.vim` を用いた TypeScript によるプラグインマネージャーの実装でした。TypeScript によるプラグイン開発は `ddc.vim`, `ddu.vim` でも行っており実績も十分です。
TypeScript を用いると型情報により堅牢なプラグインが書けるし、機能のモジュール化やその読み込みも簡単です。
しかし一つの課題がありました。`denops.vim` は遅延起動するので Vim 起動時には使えません。
プラグインマネージャーはその性質上どうしても Vim 起動時に動作する必要があります。`denops.vim` のロード時間も課題となりました。

私はそこで発想の転換をすることにしました。
「TypeScript でプラグインマネージャーの機能全てを記述するのではなく、起動時に必要な最小限の機能を Vim script で実装する、複雑な処理のみを TypeScript 化する」と。

`dein.vim` ではそもそも state ファイルという概念が存在しています。
これは起動速度の高速化のために起動時キャッシュを生成し、次回の起動のときはそちらを読み込むことで限界まで高速化するという手法です。
これを応用し、「`dpp.vim` では起動時キャッシュを生成する部分を TypeScript 化する」という設計にまとまりました。
起動時やプラグイン読み込みで必要な部分は Vim script が残りますが、それでも随分と Vim script の量を減らせる計算です。

この決断により、ユーザーの設定もキャッシュ生成部分(TypeScript)とキャッシュ(Vim script)読み込み部分で分かれることとなりました。
Vim script を書く量が減るのはユーザーにも開発者にも歓迎される部分でしょう。

::: message
完全に TypeScript で構築されたプラグインマネージャーとして `dvpm` というものがあります。
https://github.com/yukimemi/dvpm
`dpp.vim` と `dvpm` の違いとして、`dvpm` は TypeScript で全ての設定を行うのに対し、`dpp.vim` は TypeScript と Vim script のハイブリッド構成であること、
`dpp.vim` はプラグインマネージャーとしての機能を分離していることが挙げられます。
:::


## 変数の廃止、オプションの動的変更は不可

`dpp.vim` では `ddc.vim` や `ddu.vim` と同じく設定変数を廃止しました。
設定変数を廃止することによりインタフェースは固定化され、暗黙的な設定はできなくなります。
更に、設定した値はキャッシュされて固定されるので動的に変更することはできません。これはユーザーに不便を強いる仕様ではありますが、一種の割り切りとなっています。


## Deno に依存する代わりに環境依存は最小限

`dein.vim` では外部依存がほぼ無かったのにかかわらず、`dpp.vim` では Deno(`denops.vim`) に依存してしまっています。
これはマイナス面ばかりかというと、そうでもありません。

`dein.vim` には環境に依存しないという代わりにインストーラーのパフォーマンスが悪かったり、環境に依存したコードがありました。
`dpp.vim` は `Deno` の機能をそのまま使えるので環境に依存したコードがほとんどありません。
Windows でも Mac でも Linux でも Vim でも neovim でも使用することが可能なのです。


# dpp.vim の使用方法

`dpp.vim` は親切なプラグインマネージャーではありません。
詳しい使用方法はドキュメントを参照しながら試行錯誤してください、としたいのですが簡単に説明をしておきます。

`dpp.vim` には Vim script による設定部分と TypeScript による設定部分があります。
ユーザーは両者が何をしているのか理解する必要があります。


## Vim script 設定部分について

```vim
const s:dpp_base = '~/.cache/dpp/'

" Set dpp source path (required)
const s:dpp_src = '~/.cache/dpp/repos/github.com/Shougo/dpp.vim'
const s:denops_src = '~/.cache/dpp/repos/github.com/vim-denops/denops.vim'

" Set dpp runtime path (required)
execute 'set runtimepath^=' .. s:dpp_src

if dpp#min#load_state(s:dpp_base)
  " NOTE: dpp#make_state() requires denops.vim
  execute 'set runtimepath^=' .. s:denops_src
  autocmd User DenopsReady
  \ call dpp#make_state(s:dpp_base, '{TypeScript config file path}')
endif
```

`dpp.vim` と `denops.vim` は既にインストールされているとします。
まず、`dpp.vim` の関数を読み込めるように `dpp.vim` に 'runtimepath' を通します。これはプラグインマネージャーを使うなら常識であると言えるでしょう。

`dpp#min#load_state()` がポイントです。これは `dpp.vim` の state を読み込む関数となります。
`dpp#min#load_state()` は state file が不正な場合や存在しない場合に失敗して if のブロックの中が実行されます。

if のブロック内では state file を作りなおすために `dpp#make_state()` を実行します。
`dpp#make_state()` は `denops.vim` に依存しているので、'runtimepath' に `denops.vim` を追加しておかなければなりません。
`dpp#make_state()` は `denops.vim` がロードされていなければ実行できないので、`denops.vim` のロード完了後(`DenopsReady`)に自動実行されるようにします。

::: message
`dpp#make_state()` は `denops.vim` に依存していますが、他の機能は `denops.vim` に依存していないのは `dpp.vim` の大きなポイントです。
あくまで `denops.vim` は state file の生成に利用しているだけだからです。
これにより、`dpp.vim` は `denops.vim` に依存しておりながら `denops.vim` のロードを最低限にすることが可能です。
:::

`dpp#min#load_state()`, `dpp#make_state()` の引数に渡している `s:dpp_base` はプラグインのインストール先のベースとなるパスを表します。
`dein.vim` を知っているなら理解がしやすいでしょう。

`dpp#make_state()` における `{TypeScript config file path}` は次で説明するファイルへのパスのことです。
これはユーザーの TypeScript 設定ファイルを示します。


## TypeScript 設定部分について

Vim script だけでは state file の作成と更新しかできないので、`dpp.vim` では TypeScript による設定が肝となります。
`dein.vim` とは異なり、Vim script による設定には対応していません。

TypeScript による設定ファイルは Config ファイルと呼びます。
これは `ddc.vim` や `ddu.vim` における `load_config()` に相当するものと覚えておくと分かりやすいでしょう。
TypeScript による設定は例えば以下のようなものです。

```typescript
import {
  BaseConfig,
  ContextBuilder,
  Dpp,
  Plugin,
} from "https://deno.land/x/dpp_vim@v0.0.2/types.ts";
import { Denops, fn } from "https://deno.land/x/dpp_vim@v0.0.2/deps.ts";

type Toml = {
  hooks_file?: string;
  ftplugins?: Record<string, string>;
  plugins: Plugin[];
};

type LazyMakeStateResult = {
  plugins: Plugin[];
  stateLines: string[];
};

export class Config extends BaseConfig {
  override async config(args: {
    denops: Denops;
    contextBuilder: ContextBuilder;
    basePath: string;
    dpp: Dpp;
  }): Promise<{
    plugins: Plugin[];
    stateLines: string[];
  }> {
    args.contextBuilder.setGlobal({
      protocols: ["git"],
    });

    const [context, options] = await args.contextBuilder.get(args.denops);

    // Load toml plugins
    const tomls: Toml[] = [];
    const toml = await args.dpp.extAction(
      args.denops,
      context,
      options,
      "toml",
      "load",
      {
        path: "$BASE_DIR/dein.toml",
        options: {
          lazy: false,
        },
      },
    ) as Toml | undefined;
    if (toml) {
      tomls.push();
    }

    // Merge toml results
    const recordPlugins: Record<string, Plugin> = {};
    const ftplugins: Record<string, string> = {};
    const hooksFiles: string[] = [];
    for (const toml of tomls) {
      for (const plugin of toml.plugins) {
        recordPlugins[plugin.name] = plugin;
      }

      if (toml.ftplugins) {
        for (const filetype of Object.keys(toml.ftplugins)) {
          if (ftplugins[filetype]) {
            ftplugins[filetype] += `\n${toml.ftplugins[filetype]}`;
          } else {
            ftplugins[filetype] = toml.ftplugins[filetype];
          }
        }
      }

      if (toml.hooks_file) {
        hooksFiles.push(toml.hooks_file);
      }
    }

    const localPlugins = await args.dpp.extAction(
      args.denops,
      context,
      options,
      "local",
      "local",
      {
        directory: "~/work",
        options: {
          frozen: true,
          merged: false,
        },
      },
    ) as Plugin[] | undefined;

    if (localPlugins) {
      // Merge localPlugins
      for (const plugin of localPlugins) {
        if (plugin.name in recordPlugins) {
          recordPlugins[plugin.name] = Object.assign(
            recordPlugins[plugin.name],
            plugin,
          );
        } else {
          recordPlugins[plugin.name] = plugin;
        }
      }
    }

    const lazyResult = await args.dpp.extAction(
      args.denops,
      context,
      options,
      "lazy",
      "makeState",
      {
        plugins: Object.values(recordPlugins),
      },
    ) as LazyMakeStateResult | undefined;

    return {
      ftplugins,
      hooksFiles,
      plugins: lazyResult?.plugins ?? [],
      stateLines: lazyResult?.stateLines ?? [],
    };
  }
}
```

TypeScript 部分では使用する protocol(protocols オプション) の設定をしたり、
拡張機能からプラグインリストの読み込みを行ったりします。

上記では `dpp-ext-toml` の `load` action を使って toml からプラグインリストを取得しています。
これは `dein.vim` における `dein#load_toml()` と同等です。

更に、`dpp-ext-local` の `load` action を用いてローカルディレクトリにあるプラグインリストを取得することができます。

`dpp-ext-toml`, `dpp-ext-local` どちらもプラグインリストを生成するので、両者をマージしなければいけません。
実は `dein.vim` では内部で自動的にマージを行っていましたが、`dpp.vim` ではユーザーがコードを書いて自らの手でマージする必要があります。
自分でコードを書くことにより高度な柔軟性が得られ、暗黙的な処理がなくなるのですから、これはよいトレードオフだと思います。

`dpp-ext-lazy` による遅延起動を使うには state file に記述を追加する必要があるので、`makeState` action を明示的に呼び出さなければいけません。
`dpp.vim` は親切ではないので、自動的に処理が追加されません。あなたが設定した通りに動作します。

Config ファイルでは `plugins` によるプラグインリストと `stateLines` による初期化設定を与える必要があります。


# GitHub sponsors について

今回の `dpp.vim` プラグインの開発は GitHub sponsors の皆さんの支援によって行われました。
何年もかけて開発してきたプラグインを作り直すのはとてもエネルギーの必要な作業です。
皆さんからの支援がなければ、到底実現できなかっただろうと思います。

https://zenn.dev/shougo/articles/github-sponsors

https://github.com/sponsors/Shougo/

GitHub sponsors での支援は確実に自分のプラグインの開発意欲の向上に役立っているので、プラグインのユーザーに広く支援をお願いします。
