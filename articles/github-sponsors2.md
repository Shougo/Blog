---
title: "github sponsors を有効にしてからと、これまでのプラグイン開発について"
emoji: "🖤"
type: "tech"
topics: ["vim", "neovim", "denops", "github"]
published: true
---

## はじめに

::: message
この記事は「[github sponsors を有効にしたことと、これからのプラグイン開発について](https://zenn.dev/shougo/articles/github-sponsors)」の続編となります。
:::

私が github sponsors を有効にしてから一年と半年も経ち、プラグイン開発についても落ち着いているので、そろそろ私の開発が github sponsors の支援を受けたことによってどうなったのかについて語りたいと思います。

https://github.com/sponsors/Shougo/


## これまでのプラグイン開発について

前回、私が github sponsors を始めるにあたって、新たなプラグインを開発するということを高らかに宣言しました。現在それらの目標はほぼ達成できたものと考えています。

私が最近開発したプラグインは以下の通りです。

* [ddc.vim](https://github.com/Shougo/ddc.vim)

* [ddu.vim](https://github.com/Shougo/ddu.vim)

* [pum.vim](https://github.com/Shougo/pum.vim)

* [ddu-ui-filer](https://github.com/Shougo/ddu-ui-filer)

* [ddu-ui-ff](https://github.com/Shougo/ddu-ui-ff)

もちろん私が開発したのはこれだけではなく、各種プラグインの source, 関連プラグイン、既存プラグインのサポートといった作業も行っていました。


以下のプラグインはまだ構想段階です。

* [ddx.vim](https://github.com/Shougo/ddx.vim)

前回の記事を見てもらえば分かる通り、私は昨年 github sponsors を始める前はほとんど影も形もなかったプラグインを完成させました。
それらは日々成長し、既存のプラグインと比較しても完成度は勝るとも劣らないものになっています。

正直いうと、自分でもプラグインを完成させられるかは分かりませんでした。
プラグインを一つ新しく作るというのは想像以上に労力がかかります。せっかく作るからには、これまでのプラグインを越えないといけないというプレッシャーもあります。
更に今回は「プラグインのインタフェースを統一する、全ての機能は拡張可能で分離する」という高い目標もありました。
いくら私がこれまでの経験があるとはいっても、皆さんからの支援がなければ完成できていたかどうか分かりません。
これまで支援してくれた皆さまに感謝したいと思います。

私が今回のプラグイン開発で心掛けたのは、「ユーザーに正しく設定をさせる、ユーザーのやりたいことを邪魔しない」という一貫とした思想です。
最近の傾向として、「簡単に使える」や「設定が最小限で済む」プラグインが持て囃される傾向にあると思っています。
しかし、それは本当に正しいのでしょうか。設定が楽になるようにユーザーに「おもてなし」をするのはよいのですが、
その結果いざ挙動を変更しようとすると「おもてなし」が邪魔をして困難が待ち受けています。

私が作成しているプラグインは正直いって難しいです。思想やインタフェースは一貫していますが、それを理解できなければ訳が分からないことでしょう。
私はテキストエディタを使う上で頭を使う、勉強することはそもそも必要だと思っています。
私は Vim/neovim を長い期間考え、勉強してきました。テキストエディタとは簡単に習得できるものではなく、明らかに勉強が必要なソフトウェアなのです。
皆さんも例外なく長い時間を掛けてテキストエディタを学んできたのではないでしょうか。なぜプラグインは時間をかけて学ぼうとしないのでしょうか。
そこには矛盾があると思っています。テキストエディタに掛ける時間と労力に手を抜くのはやめましょう。

私はこれまで長い間プラグイン開発をして得た結論として、ユーザーに「おもてなし」をするのはやめました。
ユーザーは十分に賢いので、自分の頭で考えてプラグインを設定できるはずです。
これにより、ユーザーがやりたいことの 90% は実現できるようになったと考えています。


## その他のテキストエディタ活動について

私にとってのテキストエディタ活動はプラグインの開発だけではありません。
プラグインを開発するには本体のサポートも必要不可欠です。
プラグインに必要な機能が本体側になかった場合は諦めるのではなく、積極的に追加していくようにしています。

今回はテキストエディタ本体に以下の改善を行いました。

* `set cmdheight=0` の実装

* `TextChangedT` autocmd

* コマンドライン補完実現のための各種機能追加

* レジスタを上書きしない貼り付けマッピング

主にプラグインの改善のためにどうしても必要であった機能が多いですが、昔から要望されていた機能もあります。

`set cmdheight=0` については語ると長くなってしまいます。詳しくは以下を参照してください。

https://zenn.dev/shougo/articles/set-cmdheight-0

`TextChangedT` autocmd は端末上での自動補完を実現させるために追加しました。現在は `ddc.vim` + `pum.vim` で活用しています。

`nvim-cmp` が自前のポップアップでコマンドライン補完を実現していたのにインスパイアされ、 `ddc.vim` + `pum.vim` でも同じ機能を追加することにしました。
その際に本体に機能が不足していたので、必要な機能を自分で追加しました。これでかなり扱いやすくなったと思っています。

「レジスタを上書きしない貼り付けマッピング」は昔から要望されていた機能です。自分でも欲しいと思っていたので自分で実装しました。

ちなみに私が機能を追加する場合は、ほとんどの場合 Vim/neovim 双方に機能追加します。


## これからやってみたいことについて

これまでの自分の活動はメインとしてプラグイン開発を行い、サブのタスクとしてテキストエディタ本体への活動をしていました。
プラグイン開発に一息ついてきた今こそ、テキストエディタ本体の改善に向けて作業するべきではないかと思っています。

テキストエディタ本体(Vim, neovim)の開発、課題の解決といった作業は誰かがやらなければいけません。
他人に頼るだけではだめなのです。人が作業するのを待つのではなく、まず自分でやる。これは昔から私のモットーとしています。


## ddx.vim について

ddx.vim は私が [vinarise.vim](https://github.com/Shougo/vinarise.vim) の後継として開発しているバイナリ編集のためのプラグインです。
なぜバイナリ編集のためのプラグインを作るのかというと、自分の趣味というのもありますが全てのテキストはバイナリファイルでもあるので、全てはバイナリファイルに通じると考えているからです。バイナリエディタはテキストエディタのその先の究極の高みに存在しています。
更にバイナリファイル編集のソフトウェアはとてもニッチでクロスプラットフォームでまともに動くものがほとんどなく、日本語への対応も絶望的です。
未だに古いバイナリエディタが使われている昨今の状況から、そこには強い需要があるのではないかと私は考えました。バイナリ編集のためにテキストエディタは Vim を選ぶ、将来的にはそうなってくれると嬉しいです。

https://github.com/Shougo/ddx.vim

vinarise.vim は私が Vim でバイナリ編集を行いたいと思い作成したプラグインで、大容量ファイルの編集や日本語編集機能、プラグインによる拡張を実現しています。
当時としてはなかなかよい機能を備えていたと思っています。vinarise.vim はさすがに現代では古いプラグインとなっていますが、代替となるプラグインも特にないので一部の人にはまだ使われているようです。

ddx.vim では既存の ddc.vim, ddu.vim と同様に [denops.vim](https://github.com/vim-denops/denops.vim) を用いて実装しています。

ddx.vim では以下の機能を実現する予定です。

* UI の拡張

* 外部プラグインによるファイル解析のサポート

* 新規ファイル作成のサポート

* バイトの挿入削除のサポート

来年度は ddx.vim の開発に力を入れる予定です。ご期待ください。


## github sponsors 制度によって何が変わったか

github sponsors を始めて変わったのはテキストエディタ活動へのモチベーション、緊張感です。
寄付とはいえ、人からお金を貰っているわけですから活動に手を抜くわけにはいきません。
今までよりも定期的に活動をするようになりました。


## github sponsors 制度の課題について

私が実際に github sponsors をやってみて思ったのは github sponsor を増やしていくことの困難さです。もちろん最初のうちは支援者が順調に増えていて有り難いと思ったのですが、その後はほとんど増えなくなってしまいました。

私は「活動を継続すれば github sponsor は増えるだろう」、「新しいプラグインを開発すれば github sponsor が増えるだろう」と考えていたのですが、その認識は甘そうです。
これはおそらく、「github sponsors になってくれるのは自分の知り合いである」ことだと思っています。私の知り合いの多くが既に github sponsor になってくれているので、そこで頭打ちになってしまうのでしょう。

自分の OSS 活動を安定させるためにも、github sponsor を増やしていくことは不可欠です。これまではプラグイン開発をなによりも優先していましたが、定期的に記事を執筆するなどして、その成果を発表してく必要があるなと実感をします。


## OSS 開発でお金を得ることには夢があるのだろうか

最近、以下のブログ記事を見ました。

https://sosukesuzuki.dev/advent/2022/14/

この方は OSS 開発で 500 万円近くを得ており、これのみで生活するのはさすがに不安定すぎるのですが、それでも副収入として十分に夢のある金額です。
フルタイムで仕事をしてみれば分かりますが、年収を 100 万円上げるのもすごく苦労しなければならないためです。
この方の場合、JavaScript の Prettier という比較的メジャーでユーザーが直接使うソフトウェアなので寄付が集まりやすかったのだと思います。

以下の [azu](https://github.com/azu) 氏は github sponsors で年に 160 万円という金額を得ています。注目するべきは、この方は 3 年以上前から github sponsors を利用しており、スポンサーが 100 人を越えています。企業からも支援を受けているためこのような金額になっています。

https://efcl.info/2022/12/22/github-sponsors-report/

これらの成功事例を見ると、github sponsors を使えば OSS 開発者は簡単に大金を得ることができるように見えるかもしれません。しかし、実感としてこんなにうまくいくことはありません。人から寄付を受けるのは簡単ではないからです。
個人が開発者に寄付をするということには高いハードルがありますし、一人が払える金額も多くありません。安定した収益にするにはどうしても支援者の人数が必要ですが、知名度が相当高くないと支援者を多くするのは困難です。
つまり個人が寄付だけで OSS で支援することには限界があるので、企業は積極的に OSS を支援する必要があると思います。人を一人雇うことのコストを考えたら、OSS に払うコストというのはとても低いです。github sponsors で高収入を得ている人は例外なく企業からも支援を受けているように思います。


## 副業でお金を稼ぐということ

私が実際に github sponsors をやってみて思うのは、副業でお金を得るのは大変であるということです。世の中にはうまい話はありません。副業で稼ぐために、裏では様々な努力をしなければならないです。その人が楽に稼いでいるように見えるのは、努力を人に見せてないだけです。

人には本業が別にあるわけですから、副業で稼ぐというのはプライベートの時間を副業に突っ込むということです。自分の余暇の時間を売るということに等しいです。
あなたはどれだけの時間を売れますでしょうか。

更に副業はあくまでサブの収入源であり、それのみで生活はできません。本業が安定して稼げているからこそ生活は成りたっています。
副業がいくら稼げるようになっても本業の収入を断ってしまうと危険です。
副業とは奥さんのパートタイム収入みたいなものと考えると分かりやすいでしょうか。自分が副業で収入を得てからというもの、いっそうのこと本業の重要性というのも分かるようになりました。

副業というのが分かりにくければ、投資で考えでみましょう。
もしあなたが投資で 10% 儲けが出たとします。これは相当に大きなお金ですが、元手の 1000 万円全部投資に使っても利益は 100 万円にしかなりません。
しかも利益は一度使うとなくなってしまいますし、ずっとこの利益が出る保証はありません。これのみで生活するのは危険すぎることが分かってもらえますでしょうか。
普通にフルタイムで働いた場合はリスクも少なく、元手もいらず安定して何百万円も稼ぐことができるのです。なんと簡単なことでしょう。
庶民はまず本業で稼ぐことを頑張りましょう。本業が安定して、更に収入を増やしたいときのみ他の収入源を考えるべきです。
