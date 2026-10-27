---
title: "Go で動く新世代の補完プラグイン goda-cmp.vim"
emoji: "🖤"
type: "tech"
topics: ["vim", "neovim", "go"]
published: false
---

## 始めに

新世代の補完プラグインである goda-cmp.vim の主要な機能が一通りそろいました。私自身も ddc.vim から goda-cmp.vim に移行し、普段の編集で使っています。そろそろ広く使ってもらえる品質になったと考えたので、この記事で紹介します。

まだ開発初期の段階で、API は予告なく変わる可能性があります。皆さんに使ってもらい、バグ報告や要望をもとに仕様を固めて、version 1.0 を目指す予定となっています。

## goda-cmp.vim とは

goda-cmp.vim は、Vim/Neovim 両対応の自動補完プラグインです。ddc.vim の精神的な後継にあたります。

- 補完の本体は [goda.vim](https://github.com/Shougo/goda.vim) の上で動きます
- source、filter、UI は本体から分離した拡張です
- 設定は **Go のコード** で書きます
- ユーザーが自分でプラグインをビルドします

goda.vim は Deno に依存しない、Go 製の denops.vim のようなフレームワークです。プラグインは Go のプログラムで、エディタがそれを起動して通信します。Vim とは JSON channel、Neovim とは msgpack-rpc で話すので、同じバイナリが Vim でも Neovim でも動きます。

```
Neovim ── msgpack-rpc over stdio ─┐
                                  ▼
Vim ───── JSON channel over stdio ─► plugin binary ── プラグインと拡張
                                     (バイナリ毎に      (Go のパッケージとして
                                      1 プロセス)        ビルド時にリンク)
```

## なぜ goda-cmp.vim を作ったのか

実は自分自身、ddc.vim にそこまで大きな不満があったわけではありません。TypeScript による開発体験はなかなかよく、これまでのプラグインよりも明らかに仕様が安定していたし、このままでいく予定でした。

ただ、Deno 社による Deno の開発終了が致命的でした。

https://blog.cloudflare.com/deno-joins-cloudflare/

Deno の今後は 1 年間サポートが続けられ、コミュニティによる開発が続くものと見られますが開発体制の不安定化は避けられません。
私の開発環境の大部分が Deno で書かれたプラグインによって構成されており、サポート終了を待っていると手遅れになりそうでした。

::: message
同時に denops.vim 原作者からも denops.vim の開発終了が宣告されましたが、こちらについては私のほうでメンテナンス作業を引き継ぐことにしました。
しかしもう denops.vim に新機能を追加したりは難しいでしょう。あくまで本格的な移行までの時間稼ぎとなります。
:::

Deno の代わりを探すにあたって、私の中では Go しか候補がないなと考えられました。
Go は実行のためにコンパイルが必要ですが、スクリプト言語のように体験がよく Deno からの移行もスムーズにできると考えられるからです。

なぜ Lua で実装しなかったのかについては、これはもう私のポリシーによるものです。ここで Lua を利用したら他の Lua 製プラグインとの差別化は難しくなりますし、Go なら一つのバイナリで Vim と Neovim の両方に対応することができます。
私の求める世界は Lua によるプラグイン開発ではないと思ったのです。

Go で ddc.vim の後継プラグインをフルスクラッチ開発するにあたって、今回 AI (ClaudeCode)の力を利用することにしました。ddc.vim が持つ大量の機能を手で移植していたら終わらないと考えられるからです。
とはいえ、自分は AI 前提の開発にはしない予定です。あくまで AI はテキストエディタのための道具であり、テキストエディタこそがメインだと考えています。

ddc.vim は denops.vim の上で動いていて、実行には Deno が必要でした。goda-cmp.vim は、ビルドに Go が必要なだけです。できあがるのは cgo を使わない純粋な Go のバイナリが1つで、実行時には何も要りません。
実行時依存は嫌われますが、ビルド時の依存なら我慢できる人も多いのではないでしょうか。特に Windows 環境でのパフォーマンス向上が期待できます。
goda-cmp.vim は依存を go.mod で固定した単一バイナリなので動的にキャッシュするなどといった挙動はなく、Nix でパッケージ化しやすいはずです。

また、ddc.vim では拡張を実行時に動的に読み込んでいましたが、goda-cmp.vim では拡張は単なる Go のパッケージです。ユーザーが import してビルド時にリンクするので、次の利点があります。

- 各プラグインは Deno の起動時間を待つ必要がありません
- goda.vim のプラグインはそれぞれ別プロセスなので、1つが落ちても他のプラグインには影響しません。goda-cmp.vim の中でも、panic やタイムアウトを起こした source はその回だけ無視されます
- 拡張のバージョンは、設定の `go.mod` で固定されます
- 設定と拡張を含むプログラム全体を、コンパイラが検査します
- 存在しない source 名の指定やオプションの型の誤りは、起動前に分かります
- 使用していない拡張機能はリンクされません

## goda-cmp.vim の特徴

### Vim/Neovim 両対応

goda-cmp.vim では同じバイナリと同じ設定が Vim と Neovim の両方で動きます。必要なのは Vim 9.1 以降(`+job +channel +timers`)または Neovim です。

### 設定は自分の Go プログラム

goda-cmp.vim の設定は、自分で持つ小さな Go のモジュールです。dotfiles に置いておけば、どこでも同じ設定を再現できます。

```go
package main

import (
	"github.com/Shougo/goda-cmp.vim/cmp"
	"github.com/Shougo/goda-cmp.vim/ext/filter/matcherhead"
	"github.com/Shougo/goda-cmp.vim/ext/filter/sorterrank"
	"github.com/Shougo/goda-cmp.vim/ext/source/around"
	"github.com/Shougo/goda-cmp.vim/ext/ui/pum"
)

func main() {
	b := cmp.New()

	b.AddUI(pum.New())
	b.AddSource(around.New(around.Params{}))
	b.AddFilter(matcherhead.New(matcherhead.Params{}))
	b.AddFilter(sorterrank.New())

	b.PatchGlobal(cmp.Config{
		UI:      "pum",
		Sources: []string{"around"},
		SourceOptions: map[string]cmp.SourceOption{
			"_": {
				Matchers: []string{"matcherhead"},
				Sorters:  []string{"sorterrank"},
			},
			"around": {Mark: "A"},
		},
	})

	b.Main()
}
```

上記は現状の goda-cmp.vim の設定ファイルの例となりますが、ddc.vim を使っていた人なら見覚えがあると思います。`sources`、`sourceOptions`、`sourceParams`、`filterParams`、`patch_global`、`patch_filetype`、`patch_buffer` といった設定項目と考え方は、ddc.vim のものをそのまま引き継いでいます。

設定が Go のコードなので、gopls による補完や型検査がそのまま効きます。設定を書くときにも補完が使えるのです。

動的な設定も Go で書けます。

- `DynamicConfig`: リクエスト毎に設定を変える
- `DynamicSources`: source を切り替える
- `DynamicUI`: UI を切り替える

ddc.ts で書いていたような「候補が1つなら inline、それ以外は pum」といった切り替えも、普通の Go の関数です。

```go
DynamicUI: func(_ context.Context, _ sdk.Host, a cmp.DynamicUIArgs) (string, error) {
	if a.Context.Mode != "t" && len(a.Items) == 1 {
		return "inline", nil
	}
	return "pum", nil
},
```

もちろん、Vim script からの設定もできます。`goda_cmp#patch_global()`、`goda_cmp#patch_filetype()`、`goda_cmp#patch_buffer()` で、実行時に設定を上書きできます。コマンドラインに入るときだけ設定を変える、といった ddc.vim でおなじみの使い方もそのままです。

### source、filter、UI の本体からの分離

ddc.vim と同じく、goda-cmp.vim 本体は補完の仕組みだけを提供します。何をどう補完するかは、ユーザーが拡張を組み合わせて決めます。

ddc.vim と違うのは、拡張を「プラグインとしてインストールする」必要がない点です。拡張は Go のパッケージなので、使うパッケージを import するだけです。プラグインマネージャに登録するのは、goda.vim、goda-cmp.vim、それに Vim 側の部品を持つ UI(pum.vim など)だけです。

### コマンドラインと端末での補完

UI が対応していれば、挿入モードに加えて次の場面でも補完できます。

- **コマンドライン:** `goda_cmp#enable_cmdline()` で有効にします。`input()` と式レジスタ(`<C-r>=`)は、何もしなくても補完されます。
- **端末:** `goda_cmp#enable_terminal()` で有効にします。zsh、fish、xonsh 自身の補完を裏で動かす shellnative source と組み合わせると、Vim の端末の中でシェルの補完候補を pum.vim で選べます。

### LSP の補完

lsp source は、言語サーバーを自分で起動して通信します。Neovim の LSP クライアントに依存しないので、Vim でも同じように動きます。

```go
b.AddSource(lsp.New(lsp.Params{
	Name:        "lsp",
	Command:     []string{"gopls"},
	Filetypes:   []string{"go"},
	RootMarkers: []string{"go.work", "go.mod", ".git"},
}))
```

上記のように、言語サーバーごとに source として登録します。1つの source が1つの LSP サーバーを起動します。


### プレビュー

pum.vim の `previewer` オプションに `goda_cmp#get_previewer` を指定すると、選択中の候補のプレビューを表示できます。vim source や cmdline source では、候補のヘルプを表示できます。

### 同梱している拡張

まだ開発中であるため、拡張機能のリポジトリ分離はされていません。version 1.0 までは、ddc.vim の拡張から移植したものを `ext/` に同梱しています。名前は元の拡張から `_` を除いたものです(`matcher_head` は `matcherhead`)。パラメータも元の拡張のものをそのまま使えます。

| 種類 | 拡張 |
|---|---|
| UI | native、pum、inline、none |
| source | around、buffer、file、register、line、mocword、input、cmdline、cmdlinehistory、shellnative、shell、shellhistory、vim、lsp |
| filter | matcherhead、matcherprefix、matcherlength、sorterrank、sorterhead、sorterlength、sortertext、sortercmdlinehistory、sortershellhistory、converterremoveoverlap、convertertruncateabbr |

## ddc.vim からの移行

私自身の ddc.ts を Go に翻訳した設定が、[examples/ddc](https://github.com/Shougo/goda-cmp.vim/tree/main/examples/ddc) にあります。Vim script 側のマッピングなども一緒に置いてあるので、移行するときの参考にしてください。

ddc.vim との主な違いは次のとおりです。

- **拡張の名前:** `_` を除いた名前になります(`cmdline_history` は `cmdlinehistory`)。
- **`context_filetype`:** `ddc#enable()` の引数ではなく、`contextFiletype` オプションで指定します。
- **拡張の登録:** `ddc#register()` のような実行時の登録はありません。拡張は設定の `main.go` で import します。
- **`ddc#custom#set_context_*()`:** Go の `DynamicConfig` で置き換えます。

## goda-cmp.vim の使用方法

Go 1.26 以降と、Vim 9.1 以降または Neovim が必要です。

まず、プラグインマネージャで次のプラグインをインストールします。

- `Shougo/goda.vim`
- `Shougo/goda-cmp.vim`
- pum UI を使う場合は `Shougo/pum.vim`

次に、自分の設定モジュールを作ります。

```sh
mkdir -p ~/.vim/goda-cmp && cd ~/.vim/goda-cmp   # Neovim なら ~/.config/nvim/goda-cmp
cp /path/to/goda-cmp.vim/examples/config/{main.go,config.go} .
go mod init goda-cmp-config
go get github.com/Shougo/goda-cmp.vim@latest
go mod tidy
```

vimrc に次の行を追加します。

```vim
autocmd VimEnter * call goda_cmp#enable()
```

最後に、Vim から設定をビルドして起動します。

```vim
:call goda_cmp#build()
:call goda_cmp#enable()
```

挿入モードで文字を打つと、バッファ中の単語が補完されます。何も出ない場合は、`:echo goda#log()` でプラグインの出力を確認してください。

拡張を更新するときは、設定モジュールで `go get -u ./...` を実行し、`goda_cmp#build()` でビルドし直します。
Go が必要になるのはビルドのときだけなので実行時には必要がありません。

詳しくは `:help goda_cmp` を参照してください。

## これからについて

ddc.vim の機能はほぼ揃ったので、当面は機能追加よりも仕様の確定と安定化に力を入れます。version 1.0 では、リポジトリを次の単位に分ける予定です。

- 本体
- UI(UI 毎にリポジトリ分離)
- 拡張(source と filter をまとめて1つ)

goda-cmp.vim では ddc.vim と違って、Go の拡張は import する単位で選べるので、source 毎にリポジトリを分ける必要はありません。リポジトリを増やしすぎると管理が大変になるので、大きくなりすぎたものだけを個別に分離する方針です。

## GitHub Sponsors について

goda.vim, goda-cmp.vim の開発は、GitHub Sponsors の支援を受けて行っています。よろしければ支援をお願いします。

https://zenn.dev/shougo/articles/github-sponsors
