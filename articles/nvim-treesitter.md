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

これは基本的に treesitter のパーサー(クエリ)が古いときにエラーが出ます。
以下のスクリプトを実行して現在使用しているクエリのバージョンを確認することができます。

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

本来使用しないといけないクエリのバージョンは以下に書かれています。

https://github.com/nvim-treesitter/nvim-treesitter/blob/master/lockfile.json

さて、`:TSUpdate` によってクエリはバージョンアップしているはずなのになぜこの問題が起こるのでしょうか。
それは一部クエリのバージョンアップにビルドが必要ですが、それがビルドできていないからです。

```
tree-sitter CLI not found: `tree-sitter` is not executable!
tree-sitter CLI is needed because `latex` is marked that it needs to be generated from the grammar definitions to be compatible with nvim!
```

それをビルドするためには tree-sitter CLI を以下からビルドしてインストールする必要があるのです。

https://github.com/tree-sitter/tree-sitter
