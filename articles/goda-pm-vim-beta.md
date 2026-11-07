---
title: "goda-pm.vim: goda.vim のためのプラグインマネージャー"
emoji: "📦"
type: "tech"
topics: ["vim", "neovim", "go"]
published: false
---

## 始めに

これまで goda.vim の上に、ddu.vim や ddc.vim など私の作成した denops プラグインを移植してきました。goda-fi.vim、goda-cmp.vim、goda-term.vim、goda-hex.vim、goda-skk.vim といったプラグインです。そして最後に残っていたのがプラグインマネージャー、つまり dpp.vim でした。

https://github.com/Shougo/goda-pm.vim

goda-pm.vim は単に dpp.vim を Go で書き直しただけのものではありません。goda.vim のプラグインを管理するなら goda-pm.vim を使うと楽ができる。この記事ではそれを示したいと思います。

:::message
goda.vim とそのプラグインはまだ開発中です。version 1.0.0 になりリポジトリの分割が終わるまで、パッケージの場所や設定の書き方は変わる可能性があります。
:::

## goda.vim とは

goda.vim は、denops.vim の精神的後継として作っているフレームワークです。プラグインは Go のプログラムで、Vim/Neovim が起動したバイナリと通信します。Deno には依存しません。
単一のサーバーで動作していた denops.vim の機能をそれぞれのプラグイン組み込みにするようなイメージです。

goda.vim のプラグインは、ユーザーが自分の設定（`main.go`）をビルドしたバイナリで動きます。拡張は実行時にロードするのではなく、Go のパッケージとしてバイナリにリンクします。バージョンは `go.mod` で固定され、コンパイラがプログラム全体をチェックします。

この方式は利点も多いですが一つ面倒な点がありました。これは後で説明します。

## goda-pm.vim の特徴

### エディタ起動時には Go バイナリを使わない

基本的な構造は dpp.vim と同じです。設定から一度 state（起動用のスクリプトとプラグインの一覧）を作り、起動時は `goda_pm#min#load_state()` がそれを読み込むだけです。

起動時に Go のバイナリも goda.vim も必要ありません。バイナリが必要なのは、state を作るときと、プラグインのインストールや更新をするときだけです。dpp.vim で denops.vim が state の作成にしか使われなかったのと同じ考え方です。

### 設定は Go で書く

dpp.vim では設定を TypeScript で書きましたが、goda-pm.vim では Go で書きます。

```go
func main() {
	b := pm.New()
	b.AddProtocol(git.New())
	b.AddProtocol(http.New())
	b.AddExt(installer.New())
	b.SetConfig(config)
	b.Main()
}
```

従来のプロトコルと拡張は Go のパッケージで、設定の `main.go` が import してバイナリに組み込みます。dpp.vim のように実行時に拡張を探してロードする仕組みはありません。設定に間違いがあればビルドの時点でわかります。

設定（`*.go` と `go.mod`）がバイナリより新しければ、`goda_pm#make_state()` が先に自動でビルドします。設定を書き換えたら make_state するだけです。

### dpp.vim の TOML がそのまま使える

`ext/toml` は dpp-ext-toml と同じ TOML を読みます。dpp.vim で使っていた TOML ファイルは、ほぼそのまま持ってこられます。私自身の設定も TOML には手を入れていません。dpp.vim 本体と拡張の行を goda-pm.vim と goda.vim に置き換えただけです。
dpp.vim からの移行はとても楽です。

```toml
[[plugins]]
repo = 'Shougo/goda-pm.vim'

[[plugins]]
repo = 'Shougo/goda.vim'
```

遅延起動（`ext/lazy`）とローカルのプラグインロード（`ext/local`）も dpp.vim と同じように動きます。`on_cmd`、`on_ft`、`on_func`、`on_event`、`on_map`、`hook_add`、`hooks_file`、`multiple_hooks` などもそのままです。

### 拡張は最初から揃っている

dpp.vim ではコアからほとんどの機能を分離し、拡張を一つずつインストールする方式にしていました。goda-pm.vim でもコアと拡張は分かれていますが、拡張は同じモジュールに入っています。

| goda-pm.vim | dpp.vim での相当 |
|---|---|
| `ext/protocol/git` | dpp-protocol-git |
| `ext/protocol/http` | dpp-protocol-http |
| `ext/installer` | dpp-ext-installer |
| `ext/toml` | dpp-ext-toml |
| `ext/local` | dpp-ext-local |
| `ext/lazy` | dpp-ext-lazy |
| `ext/fi/pmfi` | ddu-source-dpp（goda-fi.vim のソース） |

拡張をバラバラにインストールしなくてよいので、最初の導入はかなり楽になったと思います。http プロトコルは Go でダウンロードと展開を行うので、curl も unzip も tar も要りません。

:::message
リポジトリの分割をするときに、これらの拡張も別のリポジトリに分ける予定ですが、管理を楽にするために dpp.vim ほどの分割はしない予定です。
:::

## goda.vim プラグインの更新が楽になる

ここからが本題です。

### これまでの問題

goda.vim のプラグインはその仕様上、Vim script の部分と Go の部分に分かれています。

- Vim script はプラグインマネージャーがインストールしたものが使われる
- Go の部分は、設定モジュールの `go.mod` が指すバージョンがバイナリに入る

プラグインを更新すると Vim script 部分は新しくなります。ところが、バイナリの Go の部分は `go.mod` のバージョンのままです。両者が食い違うと、goda.vim のバージョンチェックがバイナリを拒否します。

そのため、プラグインを更新するたびにこういう作業が必要でした。

```sh
cd ~/.vim/goda-fi
go get -u github.com/Shougo/goda-fi.vim
go mod tidy
```

そのうえで `goda_fi#build()` を実行します。これをプラグインの数だけ繰り返します。正直に言って面倒です。
一応回避策はあって、アップデートのときには `build` を使ってコマンドを実行して更新し、hook_post_update で `goda_fi#build()` を自動実行すればよいのですがあまりに手間です。

### goda-pm.vim を使うと

goda-pm.vim は state を作るとき、インストール済みのプラグインのうち Go モジュールであるもののディレクトリを `g:goda#workspace` に設定します。

goda.vim の `goda#build()`（各プラグインの `goda_xxx#build()` はこれを使います）は、`g:goda#workspace` があれば go.work を書き出してビルドします。設定が require しているプラグインのモジュールは、`go.mod` のバージョンではなく、インストールされたディレクトリのソースが使われます。

つまり、バイナリの Go の部分は常にインストールされている Vim script と一致します。プラグインを更新したら、やることはビルドだけです。`go get -u` は要りません。

```vim
:call goda_fi#build()
```

しかも、TOML に hook_post_update として書いておけば更新のたびに自動でビルドされます。

```toml
[[plugins]]
repo = 'Shougo/goda-fi.vim'
hook_post_update = 'call goda_fi#build()'
```

goda-pm.vim 自身のバイナリも同じように書けます。インストーラーが動いている最中に `goda_pm#build()` が呼ばれた場合、ビルドは処理が終わるまで待ちます。

```toml
[[plugins]]
repo = 'Shougo/goda-pm.vim'
hook_post_update = 'call goda_pm#build()'
```

バイナリがバージョンチェックで拒否されたときのメッセージも変わります。`g:goda#workspace` があれば、`go get -u` ではなく `call goda_fi#build()` のようにビルドの関数を案内します。

### ローカルで開発している goda.vim プラグインも恩恵を受ける

例えば `ext/local` でローカルにある開発中のプラグインを使っている場合、そのディレクトリも `g:goda#workspace` に入ります。開発中のプラグインを書き換えて `goda_xxx#build()` すれば、その変更がそのままバイナリに入ります。

go.work に入るのは、設定が require しているモジュールだけです。関係のないプラグインの `go.mod` が壊れていても、ビルドには影響しません。

## goda-pm.vim の使用方法

dpp.vim のときと同じく、概要だけ説明します。詳しくは `:help goda-pm` を見てください。

### Vim script の設定

```vim
" goda-pm.vim must be in 'runtimepath'.
set runtimepath^=~/.cache/goda-pm/repos/github.com/Shougo/goda-pm.vim

const s:goda_pm_base = '~/.cache/goda-pm'->expand()
let $BASE_DIR = '<script>:h'->expand()

" The configuration (main.go) lives with your dotfiles
let g:goda_pm#config_dir = '$BASE_DIR/goda-pm'->expand()

if s:goda_pm_base->goda_pm#min#load_state()
  " NOTE: goda.vim is needed to make the state
  set runtimepath^=~/.cache/goda-pm/repos/github.com/Shougo/goda.vim
  autocmd VimEnter * ++once call goda_pm#make_state(s:goda_pm_base)
endif

autocmd User GodaPm:makeStatePost
      \ echomsg 'goda_pm#make_state() is done'
```

dpp.vim との違いは主に二つです。

- `DenopsReady` を待つ必要がありません。goda.vim は最初の呼び出しで読み込まれます。
- `make_state()` に設定ファイルのパスを渡しません。設定はプラグインのバイナリにそもそも組込まれているからです。

goda-pm.vim と goda.vim は、最初に git clone しておきます。

```sh
git clone https://github.com/Shougo/goda-pm.vim ~/.cache/goda-pm/repos/github.com/Shougo/goda-pm.vim
git clone https://github.com/Shougo/goda.vim ~/.cache/goda-pm/repos/github.com/Shougo/goda.vim
```

### Go の設定

`g:goda_pm#config_dir` に Go のモジュールを作ります。私の設定から抜粋します。

```go
func config(ctx context.Context, c *pm.ConfigArgs) (*pm.ConfigReturn, error) {
	c.Options.InlineVimrcs = []string{"$BASE_DIR/options.rc.vim"}
	c.Options.Protocols = []string{"git", "http"}
	c.Options.ExtParams["installer"] = pm.Params{
		"checkDiff":     true,
		"minCommitDays": 1,
	}

	var plugins []pm.Plugin
	for _, f := range []struct {
		path    string
		options pm.Plugin
	}{
		{"$BASE_DIR/dpp.toml", pm.Plugin{"lazy": false}},
		{"$BASE_DIR/lazy.toml", pm.Plugin{"lazy": true}},
		{"$BASE_DIR/goda.toml", pm.Plugin{"lazy": true, "depends": "goda.vim"}},
	} {
		t, err := toml.Load(ctx, c.Host, f.path, f.options)
		if err != nil {
			return nil, err
		}
		plugins = append(plugins, t.Plugins...)
	}

	r, err := lazy.MakeState(ctx, c.Host, plugins)
	if err != nil {
		return nil, err
	}
	return &pm.ConfigReturn{
		Plugins:    r.Plugins,
		StateLines: r.StateLines,
	}, nil
}
```

dpp.vim の TypeScript の設定と比べると、`getExt()` で拡張を取り出して `action.callback()` を呼ぶ部分がただの関数呼び出しになりました。かなり見通しがよくなったと思います。完全な例は goda-pm.vim の `examples/config` にあります。私の dpp.ts をそのまま Go に書き直したものです。

設定ファイルにおいてモジュールの準備はこれだけです。

```sh
cd $BASE_DIR/goda-pm
go mod init goda-pm-config
go get github.com/Shougo/goda-pm.vim@latest
go mod tidy
```

### 最初の起動

1. 最初の起動では、`make_state()` がバイナリをビルドしてから state を作ります。
2. Vim を再起動し、プラグインをインストールします。

```vim
:call goda_pm#async_ext_action('installer', 'install')
```

`goda_pm#sync_ext_action()` もあります。インストールや更新のような長い処理でもタイムアウトしないので、スクリプトから更新するときに便利です。

## dpp.vim からの移行

関数や変数の名前は、基本的に置き換えるだけです。

| dpp.vim | goda-pm.vim |
|---|---|
| `dpp#min#load_state()` | `goda_pm#min#load_state()` |
| `dpp#make_state(base, config)` | `goda_pm#make_state(base)` |
| `dpp#async_ext_action()` | `goda_pm#async_ext_action()` |
| `g:dpp` | `g:goda_pm` |
| `Dpp:makeStatePost` | `GodaPm:makeStatePost` |
| `require('dpp')` | `require('goda_pm')` |

dpp.vim から意図的に外した機能もあります。

- packspec はサポートしません。残念ながらまだ流行っていないので。

互換性を保つため、denops.vim のプラグインはそのまま使えます。遅延起動された denops プラグインは denops.vim に登録されますし、インストーラーには `denoCache` もあります。

## 今後について

goda.vim のプラグインが安定化したら、そのときはリポジトリを分割して各プラグインを v1.0.0 にする予定です。各プラグインのドキュメントに goda-pm.vim 向けの設定例を載せるのはその後になります。

## GitHub sponsors について

goda.vim とそのプラグインの開発は GitHub Sponsors の支援で続けています。denops.vim の上に作ったプラグインを一通り Go に移すのは、かなりのエネルギーが必要な作業でした。
これから先はプラグインの完成度をより高めていきたいです。

goda.vim や goda-pm.vim が便利だと思ったら、ぜひ支援をお願いします。

https://github.com/sponsors/Shougo
