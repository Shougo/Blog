---
title: "自動補完プラグイン ddc.vim + pum.vim"
emoji: "🪄"
type: "tech"
topics: ["vim", "neovim", "denops"]
published: true
---

## 始めに

[前回の記事](https://zenn.dev/shougo/articles/ddc-vim-beta)から約二ヶ月が経過しました。`ddc.vim` の開発は順調に進んでおり、ようやく仕様が安定化してきています。
正式リリースも近いです。
今回は最近私が実装を行っている `pum.vim` という新プラグインと `ddc.vim` との連携について解説します。

https://github.com/Shougo/ddc.vim
https://github.com/Shougo/pum.vim


## pum.vim について

これはもともと `nvim-cmp` が実現していたアイディアになります。

https://github.com/hrsh7th/nvim-cmp/pull/224

`pum.vim` はネイティブで用意されている補完機能を使用せずに、自前で Vim の popup window 機能や floating window 機能を用いて補完を行うプラグインです。

Emacs でいうと `popup.el` に相当します。

https://github.com/auto-complete/popup-el

`ddc.vim` + `pum.vim` と `nvim-cmp` の独自補完ウインドウの大きな違いとしては、nvim-cmp は neovim 専用なのに対して `ddc.vim` + `pum.vim` だと Vim でも動作するということです。

`nvim-cmp` の独自補完ウインドウは専用なので他から使うことはできません。
`pum.vim` は `ddc.vim` 専用の仕組みではなく独立しているため、他のプラグインで使用することが可能です。


## pum.vim による自作補完ウインドウ機能のメリット

自作補完ウインドウを実装すると何がよいのかメリットについて説明します。


### パフォーマンスの改善、チラつきの低減

これは `nvim-cmp` で顕著のようですが、プラグイン側で細かい制御をするのでプラグイン側でパフォーマンスの改善を行える余地があります。
自前でウインドウを描画するので、ネイティブ補完のようにチラつきを制御するために細かい制御をする必要がありません。


### 自由なマッピング

ネイティブの補完ウインドウはマッピングがソースコードに組込まれており、完全に決まっています。一応ユーザーが `pumvisible()` を用いて条件分岐すれば変更できるのですが、一部マッピングできないキーが存在します。

`pum.vim` ではマッピングはユーザーが定義するため、完全に自由なマッピングにすることが可能です。


### 豊富な拡張性

ネイティブの補完ウインドウは本体が C 言語側で実装しているので制限が多いのですが、`pum.vim` はプラグインで制御するので機能追加がやりやすく拡張の余地があります。
プラグインとの連携も比較的簡単です。


### カスタムハイライト

ネイティブの補完ウインドウは色付けに対応していませんが、独自に補完ウインドウを描画するとこの問題はありません。

`pum.vim` にはデフォルトで abbr, kind, menu に色付けを行うことができます。プラグイン側で独自に色を付けることも可能です。
もちろん、この機能は補完ウインドウのパフォーマンスに影響があるので注意が必要です。

現在 `ddc-fuzzy` がカスタムハイライトによるマッチング位置の強調表示に対応しています。

https://github.com/tani/ddc-fuzzy

![ddc highlight](/images/ddc-highlight.png)


### コマンドライン補完

ネイティブの補完ウインドウはもちろん挿入モードの補完にしか対応していません。
ただし Vim にはコマンドラインで自由な補完を行うパッチがあったり、neovim は補完をポップアップウインドウで表示する設定 `set wildoptions+=pum` に対応しています。

`wilder.nvim` を用いるとポップアップウインドウでコマンドライン補完を行うことが可能です。
`wilder.nvim` は外部の補完プラグインを呼んでいるのではなく補完 source を組込みで持っているようです。

https://github.com/gelguy/wilder.nvim

`pum.vim` はコマンドラインでの補完に対応しており、以下のように `ddc.vim` での補完をコマンドラインで行うことが可能です。

![command line completion](https://user-images.githubusercontent.com/41495/135990228-a0c95c07-7a50-4294-8897-a1a59662b3b5.png)

応用例としては、`neco-vim` を用いてコマンド引数を補完する、`ddc-cmdline-history` を用いてコマンドラインヒストリから補完する、バッファーの内容から補完するといったことがあります。

https://github.com/Shougo/neco-vim

https://github.com/Shougo/ddc-cmdline-history

Note: `nvim-cmp` もコマンドライン補完対応機能を開発中です。

https://github.com/hrsh7th/nvim-cmp/pull/362


### 端末補完(実験的機能)

`pum.vim` は挿入モードやコマンドラインだけでなく、端末モードからでさえ補完を行うことが可能です。夢が広がるのではないでしょうか。

![terminal completion](https://user-images.githubusercontent.com/41495/136798285-cd809598-a1d1-42f4-92ff-c4cac0eb372c.png)


手元だと一応動作はしていますが、Vim/neovim の terminal API の制限により挙動は安定していません。あくまで実験的機能となります。


## pum.vim による自作補完ウインドウ機能のデメリット

`pum.vim` による自作補完ウインドウも万能ではありません。以下のようなデメリットがあります。


### 実装が複雑化

`pum.vim` による補完ウインドウは Vim や neovim の実装と完全に独立しているので、全て自前で機能を実装する必要があります。実装の複雑化は避けられません。


### 依存関係が増える

`pum.vim` は `ddc.vim` と独立しているので、当然別個でインストールしなければいけません。依存が増えるのを嫌う人もいるかと思います。


### 既存の設定やプラグインとの互換性問題

既存の設定やプラグインはネイティブの補完ウインドウを前提に作られているので、コンフリクトする可能性が高いです。
マクロや `.` によるリピートとも相性がよくありません。

`nvim-cmp` は `feedkeys()` による黒魔術で無理矢理対応しているようです。


### マッピングが独自

既存の補完ウインドウのマッピングは使えません。`pum.vim` 独自のマッピングを使用する必要があります。
`pumvisible()` も使えないので、`pum#pumvisible()` を使う必要があります。


## ddc.vim + pum.vim の連携方法

`ddc.vim` において `pum.vim` をどのように有効化するかについて説明します。
ここではそれぞれのプラグインと依存関係はすでにインストール済み、ロード済みとして話を続けます。

`pum.vim` との連携はデフォルトで無効になっています。以下の設定を行うことで有効化することが可能です。

```vim
call ddc#custom#patch_global('completionMenu', 'pum.vim')
```

`pum.vim` はデフォルトマッピングを定義しないので、以下のようにマッピングを定義する必要もあります。

```vim
inoremap <silent><expr> <TAB>
      \ pum#visible() ? '<Cmd>call pum#map#insert_relative(+1)<CR>' :
      \ (col('.') <= 1 <Bar><Bar> getline('.')[col('.') - 2] =~# '\s') ?
      \ '<TAB>' : ddc#manual_complete()
inoremap <S-Tab> <Cmd>call pum#map#insert_relative(-1)<CR>
inoremap <C-n>   <Cmd>call pum#map#select_relative(+1)<CR>
inoremap <C-p>   <Cmd>call pum#map#select_relative(-1)<CR>
inoremap <C-y>   <Cmd>call pum#map#confirm()<CR>
inoremap <C-e>   <Cmd>call pum#map#cancel()<CR>
```

コマンドライン補完を有効化するには、以下のように設定する必要があります。`:` を入力したときに `ddc.vim` の設定を切り替えるようにします。

```vim
call ddc#custom#patch_global('autoCompleteEvents', [
    \ 'InsertEnter', 'TextChangedI', 'TextChangedP',
    \ 'CmdlineEnter', 'CmdlineChanged',
    \ ])

nnoremap :       <Cmd>call CommandlinePre()<CR>:

function! CommandlinePre() abort
  " Note: It disables default command line completion!
  cnoremap <expr> <Tab>
  \ pum#visible() ? '<Cmd>call pum#map#insert_relative(+1)<CR>' :
  \ ddc#manual_complete()
  cnoremap <S-Tab> <Cmd>call pum#map#insert_relative(-1)<CR>
  cnoremap <C-y>   <Cmd>call pum#map#confirm()<CR>
  cnoremap <C-e>   <Cmd>call pum#map#cancel()<CR>

  " Overwrite sources
  let s:prev_buffer_config = ddc#custom#get_buffer()
  call ddc#custom#patch_buffer('sources',
          \ ['cmdline', 'cmdline-history', 'around'])

  autocmd User DDCCmdlineLeave ++once call CommandlinePost()

  " Enable command line completion
  call ddc#enable_cmdline_completion()
  call ddc#enable()
endfunction
function! CommandlinePost() abort
  " Restore sources
  call ddc#custom#set_buffer(s:prev_buffer_config)
  cunmap <Tab>
endfunction
```

`ddc#enable_cmdline_completion()` はコマンドライン補完を有効化する設定で、コマンドライン補完を使用するには必ず呼び出す必要があります。これはコマンドライン補完の設定をコマンドライン外で常に有効化するとパフォーマンスに影響があるためです。

以下の source はコマンドライン補完で使うのに便利です。

https://github.com/Shougo/ddc-cmdline
https://github.com/Shougo/ddc-cmdline-history
https://github.com/Shougo/ddc-around


## GitHub sponsors について

今回の `ddc.vim` プラグインの開発と `pum.vim` プラグインの開発は GitHub sponsors の皆さんの支援によって行われました。

https://zenn.dev/shougo/articles/github-sponsors

GitHubのコミット数を見てもらえれば、GitHub sponsorsを開始した6月以降にコミット数が劇的に伸びているのを確認できるかと思います。

https://github.com/Shougo

https://github.com/sponsors/Shougo/

最近、テキストエディタのプラグインを開発するため、開発用ノートパソコンとして Thinkpad T14 Gen2 を購入しました。
これは 6 年半ぶりのマシン更新となります。効率的な開発には効率的なノートパソコンが必要不可欠です。
これも github sponsors があったから決断できたことです。github sponsors の皆さんには本当に感謝しています。
