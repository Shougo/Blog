---
title: "nvim-treesitter トラブルシューティング"
emoji: "🖤"
type: "tech"
topics: ["neovim", "treesitter"]
published: true
---

# 始めに

neovim では treesitter(`nvim-treesitter`) 機能が組込まれており、それを用いてシンタックスハイライトができますがトラブルが多いです。
`nvim-treesitter` を使っているときのトラブルとその対処方法についてまとめておきます。
不定期更新です。


# nvim-treesitter のクエリエラーについて

neovim や `nvim-treesitter` をアップデートしたときにクエリのエラーが発生することはないでしょうか。

これは基本的に treesitter のパーサーが古くクエリと不整合を起こしたときにエラーが出ます。
以下のスクリプトを実行して現在使用しているパーサーのバージョンを確認することができます。

```lua
local utils = require("nvim-treesitter.utils")
local configs = require("nvim-treesitter.configs")

local function get_installed_revision(lang)
    local lang_file = utils.join_path(configs.get_parser_info_dir(), lang .. ".revision")
    if vim.fn.filereadable(lang_file) == 1 then
        return vim.fn.readfile(lang_file)[1]
    end
end

vim.print {
    markdown = get_installed_revision("markdown"),
    markdown_inline = get_installed_revision("markdown_inline"),
    latex = get_installed_revision("latex"),
}
```

`nvim-treesitter` で本来使用しないといけないパーサーのバージョンは以下に書かれています。

https://github.com/nvim-treesitter/nvim-treesitter/blob/master/lockfile.json

さて、`:TSUpdate` によってパーサーはバージョンアップしているはずなのになぜこの問題が起こるのでしょうか。
それは一部パーサーのバージョンアップにビルドが必要ですが、それがビルドできていないからです。
実は `:TSUpdate` の実行時にエラーが発生していますが、ユーザーにはそれが分からないので無視できてしまいます。

例えば以下のようなエラーです。

```
tree-sitter CLI not found: `tree-sitter` is not executable!
tree-sitter CLI is needed because `latex` is marked that it needs to be generated from the grammar definitions to be compatible with nvim!
```

それをビルドするためには tree-sitter CLI を以下からビルドしてインストールする必要があるのです。

https://github.com/tree-sitter/tree-sitter


# 特定の言語で treesitter ハイライトを無効にする

treesitter によるハイライトは便利なのですが、特定の言語で treesitter のハイライトが気にいらないときに強制的に無効化したいときもあると思います。
treesitter によるハイライトは `nvim-treesitter` のハイライトオプションで無効化することも可能なのですが、最近は neovim 本体が強制的に treesitter ハイライトを有効化していることがありその場合に効果がありません。

確実に無効化するには以下のような設定をするとよいです。

```vim
function s:config_treesitter()
  lua << END
  vim.treesitter.start = (function(wrapped)
    return function(bufnr, lang)
      local ft = vim.fn.getbufvar(bufnr or vim.fn.bufnr(''), '&filetype')
      local check = (
        ft == 'help' or lang == 'vimdoc' or lang == 'diff'
        or lang == 'gitcommit' or lang == 'swift' or lang == 'latex'
      )
      if check then
        return
      end

      wrapped(bufnr, lang)
    end
  end)(vim.treesitter.start)
END
endfunction
autocmd Syntax * ++once call s:config_treesitter()
```
