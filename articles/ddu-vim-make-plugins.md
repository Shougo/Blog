---
title: "自動補完プラグイン ddu.vim の拡張方法について"
emoji: "🪄"
type: "tech"
topics: ["vim", "neovim", "denops"]
published: true
---

## 始めに

::: message
この記事は「[自動補完プラグイン ddc.vim の拡張方法について](https://zenn.dev/shougo/articles/ddc-vim-make-plugins)」の続編となります。
:::

最近はプラグイン開発も落ち着いており、細々と `ddx.vim` の開発を進めていたりします。
他になにかするべきではないかと考えていると、「`ddc.vim` や `ddu.vim` のプラグインの作り方記事があるとユーザーが増えるのではないか」という意見をもらいました。
確かに、調べてみても `ddc.vim` や `ddu.vim` の設定について書いてあるものはあっても拡張方法を解説している記事は見当たりません。
ドキュメントには書いているのですが、あれはどちらかというとリファレンスでしょう。
前回は `ddc.vim` の拡張方法について解説したので、今回は `ddu.vim` の拡張方法について解説します。

https://github.com/Shougo/ddu.vim


## ddu.vim プラグインの基本

`ddu.vim` は `denops.vim` を用いて開発されており、TypeScript で拡張することが可能です。
TypeScript で拡張できるということの利点は、型を用いることで保守性が高まることです。
他にもライブラリが豊富なのも Web API を扱うには便利でしょう。
最近は `Deno` から npm のライブラリを呼び出すことができ、よりパワフルな環境となりつつあります。

`unite.vim` は vimrc 上でインラインに source を定義でき、`ddu.vim` よりも source を作るのは簡単でしたが Vim script の複雑なプログラムは保守が大変でした。

`ddu.vim` 用のプラグインは `denops/` 以下の特定の場所にファイルを配置することで自動的にロードされます。
手動でプラグインを登録することはできません。これは、Vim, neovim の Vim script, Lua プラグインと同じです。
最初は手動で登録することもできるようにしていたのですが、それで便利になることがなく手動登録が面倒なだけだったので廃止しました。


## source の作成方法

それではまず、`ddu.vim` の簡単な `source` を作成してみることにしましょう。
プラグインのディレクトリに以下の `denops/@ddu-sources/sample.ts` のファイルを作成します。

```typescript
import {
  BaseSource,
  DduOptions,
  Item,
  SourceOptions,
} from "https://deno.land/x/ddu_vim@v2.2.0/types.ts";
import { Denops, fn } from "https://deno.land/x/ddu_vim@v2.2.0/deps.ts";
import { join } from "https://deno.land/std@0.171.0/path/mod.ts";
import { ActionData } from "https://deno.land/x/ddu_kind_file@v0.3.2/file.ts";

type Params = Record<never, never>;

export class Source extends BaseSource<Params> {
  override kind = "file";

  override gather(args: {
    denops: Denops;
    options: DduOptions;
    sourceOptions: SourceOptions;
    sourceParams: Params;
    input: string;
  }): ReadableStream<Item<ActionData>[]> {
    return new ReadableStream({
      async start(controller) {
        const dir = await fn.getcwd(args.denops) as string;

        const tree = async (root: string) => {
          let items: Item<ActionData>[] = [];
          try {
            for await (const entry of Deno.readDir(root)) {
              const path = join(root, entry.name);

              items.push({
                word: path,
                action: {
                  path: path,
                },
              });
            }
          } catch (e: unknown) {
            console.error(e);
          }

          return items;
        };

        controller.enqueue(
          await tree(dir),
        );

        controller.close();
      },
    });
  }

  override params(): Params {
    return {};
  }
}
```

プラグインの型情報は `ddu_vim` のリポジトリから取得する必要があります。

https://deno.land/x/ddu_vim@v2.2.0

`BaseSource` を継承して Source を作成します。`gather()` はアイテムを返す関数です。ここではカレントディレクトリにあるファイルを返しています。

`gather()` では非同期でアイテムを追加できるようにするため、 `ReadableStream` を用いる必要があります。
`ReadableStream` の `controller` にアイテムを追加することでリアルタイムに反映されます。アイテムの追加が終了したら `controller` を close します。

それぞれのアイテムは `Item` 型となっています。`Item` 型については `:help ddu-item-attributes` を参照してください。
`options` にはユーザーのオプション (`:help ddu-options`)、`sourceOptions` には source 共通のオプション(`:help ddu-source-options`)、`sourceParams` は source 固有のオプションです。
`input` はユーザーの絞り込み文字列が渡されます。

`source` では絞り込みを行わずに、該当する全てのアイテムを返却するようにします。絞り込みを行うのは `filter` の役目だからです。

`source` はアイテムの `kind` を指定することが可能です。ここでは `file` を指定しており、`file` のアクションを実行できるアイテムを生成することが分かります。
`file` `kind` のアクションを実行するために、アイテムの `action` でアクションを実行するための情報をセットする必要があります。
`action` には `ActionData` 型の値をセットする必要があります。`ActionData` の中身は `kind` によって異なるので `kind` のリポジトリから型定義を取得する必要があります。

このプラグインを試すための最小構成は例えば以下のようになります。`~/src/ddu-samples` が今回作成したプラグインのディレクトリです。

```vim
set nocompatible

set rtp+=~/work/denops.vim
set rtp+=~/work/ddu.vim
set rtp+=~/work/ddu-ui-ff
set rtp+=~/work/ddu-kind-file
set rtp+=~/src/ddu-samples

call ddu#custom#patch_global(#{
      \   ui: 'ff',
      \   sources: [#{name: 'sample', params: {}}],
      \   kindOptions: #{
      \     file: #{
      \       defaultAction: 'open',
      \     },
      \   },
      \ })

autocmd FileType ddu-ff call s:ddu_my_settings()
function! s:ddu_my_settings() abort
  nnoremap <buffer> <CR>
        \ <Cmd>call ddu#ui#ff#do_action('itemAction')<CR>
endfunction

call ddu#start({})
```

## filter の作成方法

次に `ddu.vim` の `filter` を作成します。`filter` とは `matchers`, `sorters`, `converters` に指定するもので、それぞれのインタフェースは共通となっています。
プラグインのディレクトリに以下の `denops/@ddu-filters/sample.ts` のファイルを作成します。

```typescript
import {
  BaseFilter,
  DduItem,
  SourceOptions,
} from "https://deno.land/x/ddu_vim@v2.0.0/types.ts";
import { Denops } from "https://deno.land/x/ddu_vim@v2.0.0/deps.ts";

type Params = {
  foot: string;
};

export class Filter extends BaseFilter<Params> {
  // deno-lint-ignore require-await
  override async filter(args: {
    denops: Denops;
    sourceOptions: SourceOptions;
    filterParams: Params,
    input: string;
    items: DduItem[];
  }): Promise<DduItem[]> {
    for (const item of args.items) {
      item.display = item.word + args.filterParams.foot;
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
プラグインの型情報は `ddu_vim` のリポジトリから取得する必要があります。

https://deno.land/x/ddu_vim@v2.2.0

`BaseFilter` を継承して `Filter` を作成します。`filter()` は加工後のアイテムを返す関数です。ここでは `word` に `filterParams.foot` を結合したものを `display` にセットしています。
それぞれの補完候補は `Item` 型となっています。`Item` 型については `:help ddu-item-attributes` を参照してください。
`filterParams` は filter 固有のオプションです。
`input` は絞り込み文字列が渡されます。`items` は全体のアイテムが格納されています。


このプラグインを試すための最小構成は例えば以下のようになります。`~/src/ddu-samples` が今回作成したプラグインのディレクトリです。

```vim
set rtp+=~/work/denops.vim
set rtp+=~/work/ddu.vim
set rtp+=~/work/ddu-ui-ff
set rtp+=~/work/ddu-kind-file
set rtp+=~/src/ddu-samples

call ddu#custom#patch_global(#{
      \   ui: 'ff',
      \   sources: [#{name: 'sample', params: {}}],
      \   sourceOptions : #{
      \     _ : #{
      \       converters: ['sample'],
      \     }
      \   },
      \   kindOptions: #{
      \     file: #{
      \       defaultAction: 'open',
      \     },
      \   },
      \ })

autocmd FileType ddu-ff call s:ddu_my_settings()
function! s:ddu_my_settings() abort
  nnoremap <buffer> <CR>
        \ <Cmd>call ddu#ui#ff#do_action('itemAction')<CR>
endfunction

call ddu#start({})
```

## kind の作成方法

`ddu.vim` では独自の kind を追加することが可能です。アクションは kind に紐付いており、独自の kind を追加することで独自のアクションを実行することができます。
プラグインのディレクトリに以下の `denops/@ddu-kinds/sample.ts` のファイルを作成します。

```typescript
import {
  ActionArguments,
  ActionFlags,
  BaseKind,
  DduItem,
} from "https://deno.land/x/ddu_vim@v2.2.0/types.ts";
import { Denops, fn, vars } from "https://deno.land/x/ddu_vim@v2.2.0/deps.ts";


export type ActionData = {
  path: string;
};

type Params = Record<never, never>;

export class Kind extends BaseKind<Params> {
  override actions: Record<
    string,
    (args: ActionArguments<Params>) => Promise<ActionFlags>
  > = {
    echo: async (args: { denops: Denops; items: DduItem[] }) => {
      for (const item of args.items) {
        console.log(item.action.path);
      }
      return Promise.resolve(ActionFlags.None);
    },
  };

  override params(): Params {
    return {};
  }
}
```
`BaseKind` を継承して `Kind` を作成します。`actions` には `kind` のアイテムのアクションを定義します。ここでは `echo` アクションを定義しています。
`ActionData` にはアイテムに含まれるアクションの固有情報を定義します。
`items` はアクションの実行対象となるアイテムが格納されています。アイテムが複数選択されていた場合に対応するためにリストとなっています。
アクションは `ActionFlags` を返却します。それぞれの `ActionFlags` の意味は以下です。

* `ActionFlags.None`: アクションの実行後、UI を閉じる。
* `ActionFlags.RefreshItems`: アクションの実行後、UI のアイテムを再取得する。
* `ActionFlags.Redraw`: アクションの実行後、UI を再描画する。
* `ActionFlags.Persist`: アクションの実行後、UI を閉じない。

ちなみに、`kind` のデフォルトアクションはユーザーが指定するので `kind` 側で指定できません。


この `kind` を `source` 側で使用するには以下のように `source` の `kind` として指定する必要があります。

```typescript
export class Source extends BaseSource<Params> {
  override kind = "sample";
  ...
}

```

このプラグインを試すための最小構成は例えば以下のようになります。`~/src/ddu-samples` が今回作成したプラグインのディレクトリです。

```vim
set rtp+=~/work/denops.vim
set rtp+=~/work/ddu.vim
set rtp+=~/work/ddu-ui-ff
set rtp+=~/work/ddu-kind-file
set rtp+=~/src/ddu-samples

call ddu#custom#patch_global(#{
      \   ui: 'ff',
      \   sources: [#{name: 'sample', params: {}}],
      \   sourceOptions : #{
      \     _ : #{
      \       converters: ['sample'],
      \     }
      \   },
      \   kindOptions: #{
      \     sample: #{
      \       defaultAction: 'echo',
      \     },
      \   },
      \ })

autocmd FileType ddu-ff call s:ddu_my_settings()
function! s:ddu_my_settings() abort
  nnoremap <buffer> <CR>
        \ <Cmd>call ddu#ui#ff#do_action('itemAction')<CR>
endfunction

call ddu#start({})
```


## UI の作成方法

`ddu.vim` は UI も拡張可能です。UI を自分で作成することで、アイテムを自由に表示できます。
しかし `source`, `filter`, `kind` と比較すると仕様が複雑である上に需要はあまりないと思うので、今回は解説を省略します。


## columns の作成方法

`ddu.vim` は表示カラムの拡張もサポートしていますが、こちらは `ddu-ui-filer` といった一部の UI の表示でしか使わないものなので解説を省略します。


## GitHub sponsors について

`ddu.vim` プラグインの開発は GitHub sponsors の皆さんの支援によって行われました。
今後も Github sponsors による支援を受けてテキストエディタ活動を続けていきたいと思っています。

https://zenn.dev/shougo/articles/github-sponsors
