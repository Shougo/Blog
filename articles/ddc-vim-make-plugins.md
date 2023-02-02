---
title: "自動補完プラグイン ddc.vim の拡張方法について"
emoji: "🪄"
type: "tech"
topics: ["vim", "neovim", "denops"]
published: true
---

## 始めに

最近はプラグイン開発も落ち着いており、細々と `ddx.vim` の開発を進めていたりします。
他になにかするべきではないかと考えていると、「`ddc.vim` や `ddu.vim` のプラグインの作り方記事があるとユーザーが増えるのではないか」という意見をもらいました。
確かに、調べてみても `ddc.vim` や `ddu.vim` の設定について書いてあるものはあっても拡張方法を解説している記事は見当たりません。
ドキュメントには書いているのですが、あれはどちらかというとリファレンスでしょう。
作者が直接記事を書くことも悪くないと思ったので、まずは `ddc.vim` の拡張方法について解説することにします。

https://github.com/Shougo/ddc.vim


## ddc.vim プラグインの基本

`ddc.vim` は `denops.vim` を用いて開発されており、TypeScript で拡張することが可能です。
TypeScript で拡張できるということの利点は、型を用いることで保守性が高まることです。
他にもライブラリが豊富なのも Web API を扱うには便利でしょう。
最近は `Deno` から npm のライブラリを呼び出すことができ、よりパワフルな環境となりつつあります。

`unite.vim` は vimrc 上でインラインに source を定義でき、`ddc.vim` よりも source を作るのは簡単でしたが Vim script の複雑なプログラムは保守が大変でした。

`ddc.vim` 用のプラグインは `denops/` 以下の特定の場所にファイルを配置することで自動的にロードされます。
手動でプラグインを登録することはできません。これは、Vim, neovim の Vim script, Lua プラグインと同じです。
最初は手動で登録することもできるようにしていたのですが、それで便利になることがなく手動登録が面倒なだけだったので廃止しました。


## source の作成方法

それではまず、`ddc.vim` の簡単な `source` を作成してみることにしましょう。
プラグインのディレクトリに以下の `denops/@ddc-sources/sample.ts` のファイルを作成します。

```typescript
import {
  BaseSource,
  DdcOptions,
  Item,
  SourceOptions,
} from "https://deno.land/x/ddc_vim@v3.4.0/types.ts";
import {
  Denops,
} from "https://deno.land/x/ddc_vim@v3.4.0/deps.ts";

type Params = Record<never, never>;

export class Source extends BaseSource<Params> {
  override async gather(args: {
    denops: Denops;
    options: DdcOptions;
    sourceOptions: SourceOptions;
    sourceParams: Params;
    completeStr: string;
  }): Promise<Item[]> {
    return [
        { word: "foo"}, { word: "bar"}, { word: "baz"},
    ];
  }

  override params(): Params {
    return {};
  }
}
```

プラグインの型情報は `ddc_vim` のリポジトリから取得する必要があります。

https://deno.land/x/ddc_vim@v3.4.0

`BaseSource` を継承して Source を作成します。`gather()` は補完候補を返す関数です。ここでは簡単化のために単純な単語のみを返しています。
それぞれの補完候補は `Item` 型となっています。`Item` 型については `:help ddc-item-attributes` を参照してください。
`options` にはユーザーのオプション (`:help ddc-options`)、`sourceOptions` には source 共通のオプション(`:help ddc-source-options`)、`sourceParams` は source 固有のオプションです。
`completeStr` は補完文字列が渡されます。

この `source` では補完開始位置を明示的に指定していません。その場合は `ddc-option-keywordPattern` が使用されます。
`source` が独自に補完開始文字位置を指定する場合は `getCompletePosition()` を定義する必要があります。

`source` では絞り込みを行わずに、該当する全ての補完候補を返却するようにします。絞り込みを行うのは `filter` の役目だからです。

このプラグインを試すための最小構成は例えば以下のようになります。`~/src/ddc-samples` が今回作成したプラグインのディレクトリです。

```vim
set nocompatible

set runtimepath+=~/work/denops.vim
set runtimepath+=~/work/ddc.vim
set runtimepath+=~/work/ddc-ui-native
set runtimepath+=~/work/ddc-matcher_head
set runtimepath+=~/work/ddc-sorter_rank

set runtimepath+=~/src/ddc-samples

set completeopt=menu,menuone,noselect

call ddc#custom#patch_global('sources', ['sample'])
call ddc#custom#patch_global('ui', 'native')

call ddc#custom#patch_global('sourceOptions', #{
      \   _: #{
      \     matchers: ['matcher_head'],
      \     sorters: ['sorter_rank']
      \   },
      \ })

call ddc#enable()
```

## filter の作成方法

次に `ddc.vim` の `filter` を作成します。`filter` とは `matchers`, `sorters`, `converters` に指定するもので、それぞれのインタフェースは共通となっています。
プラグインのディレクトリに以下の `denops/@ddc-filters/sample.ts` のファイルを作成します。

プラグインの型情報は `ddc_vim` のリポジトリから取得する必要があります。

https://deno.land/x/ddc_vim@v3.4.0

`BaseFilter` を継承して `Filter` を作成します。`filter()` は加工後の候補を返す関数です。ここでは `word` に `filterParams.foot` を結合したものを abbr にセットしています。
それぞれの補完候補は `Item` 型となっています。`Item` 型については `:help ddc-item-attributes` を参照してください。
`filterParams` は filter 固有のオプションです。
`completeStr` は補完文字列が渡されます。

```typescript
import {
  BaseFilter,
  Item,
} from "https://deno.land/x/ddc_vim@v3.4.0/types.ts";

type Params = {
  foot: string;
};

export class Filter extends BaseFilter<Params> {
  override filter(args: {
    filterParams: Params,
    completeStr: string,
    items: Item[],
  }): Promise<Item[]> {
    for (const item of args.items) {
      item.abbr = item.word + args.filterParams.foot;
    }

    return Promise.resolve(args.items);
  }

  override params(): Params {
    return {
      foot: "hoge",
    };
  }
}
```

このプラグインを試すための最小構成は例えば以下のようになります。`~/src/ddc-samples` が今回作成したプラグインのディレクトリです。

```vim
set nocompatible

set runtimepath+=~/work/denops.vim
set runtimepath+=~/work/ddc.vim
set runtimepath+=~/work/ddc-ui-native
set runtimepath+=~/work/ddc-matcher_head
set runtimepath+=~/work/ddc-sorter_rank

set runtimepath+=~/src/ddc-samples

set completeopt=menu,menuone,noselect

call ddc#custom#patch_global('sources', ['sample'])
call ddc#custom#patch_global('ui', 'native')

call ddc#custom#patch_global('sourceOptions', #{
      \   _: #{
      \     matchers: ['matcher_head'],
      \     sorters: ['sorter_rank'],
      \     converters: ['sample']
      \   },
      \ })

call ddc#enable()
```

## UI の作成方法

`ddc.vim` は補完 UI も拡張可能です。UI を自分で作成することで、補完候補を自由に表示できます。
しかし source, filter と比較すると需要はあまりないと思うので、今回は解説を省略します。


## GitHub sponsors について

`ddc.vim` プラグインの開発は GitHub sponsors の皆さんの支援によって行われました。
今後も Github sponsors による支援を受けてテキストエディタ活動を続けていきたいと思っています。

https://zenn.dev/shougo/articles/github-sponsors
