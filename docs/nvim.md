# Neovim

* [Popular Neovim Configurations](https://dotfyle.com/neovim/configurations/top)
* [Christian Chiarulli's neovim config](https://github.com/ChristianChiarulli/nvim/)


## Tips-And-Tricks

* [Is it possible to disable lsp formatting temporarily?](https://www.reddit.com/r/neovim/comments/oo8jcu/is_it_possible_to_disable_lsp_formatting/)
   You can avoid all autocommands with `:noa w`, or ignore some (or all) for some time with `:set eventignore=BufWritePre`.


## LazyVim

LazyVim is a collection of plugins and is built on top of [lazy.vim](https://github.com/folke/lazy.nvim).
LazyVim is a Neovim setup powered by [💤 lazy.nvim](https://github.com/folke/lazy.nvim) to make it easy to customize and extend your config.

* [LazyVim](https://www.lazyvim.org/)
* [GitHub](https://github.com/LazyVim/LazyVim): Neovim config for the lazy
* [GitHub - lazy.vim](https://github.com/folke/lazy.nvim): 💤 A modern plugin manager for Neovim


## TreeSitter

TreeSitter in `nvim` is used for better syntax highlighting.

Tree-sitter is a parser generator tool and an incremental parsing library.
It can build a concrete syntax tree for a source file and efficiently update the syntax tree as the source file is edited.
Tree-sitter aims to be:
* General enough to parse any programming language
* Fast enough to parse on every keystroke in a text editor
* Robust enough to provide useful results even in the presence of syntax errors
* Dependency-free so that the runtime library (which is written in pure C) can be embedded in any application

* [tree-sitter](https://github.com/tree-sitter/tree-sitter): An incremental parsing system for programming tools
* [Documentation](https://tree-sitter.github.io/tree-sitter/)
* [nvim-treesitter](https://github.com/nvim-treesitter/nvim-treesitter): Nvim Treesitter configurations and abstraction layer


### nvim-treesitter-textobjects

[github](https://github.com/nvim-treesitter/nvim-treesitter-textobjects):
Syntax aware text-objects, select, move, swap, and peek support.


### nvim-treesitter-context

[github](https://github.com/nvim-treesitter/nvim-treesitter-context):
Show the function you are in at the top of the buffer.

A Vim plugin that shows the context of the currently visible buffer contents.
It's supposed to work on a wide range of file types, but is probably most useful when looking at source code files.
In most programming languages this context will show you which function you're looking at, and within that function which loops or conditions are surrounding the visible code.




## Telescope

### telescope.nvim

[github](https://github.com/nvim-telescope/telescope.nvim):
Find, Filter, Preview, Pick. All lua, all the time.

### telescope-fzf-native.nvim

[github](https://github.com/nvim-telescope/telescope-fzf-native.nvim):
FZF sorter for telescope written in c.
`fzf-native` is a `c` port of `fzf`.
It only covers the algorithm and implements few functions to support calculating the score.


## which-key.nvim

[github](https://github.com/folke/which-key.nvim):
💥 Create key bindings that stick. WhichKey is a lua plugin for Neovim 0.5 that displays a popup with possible keybindings of the command you started typing.


## gitsigns.nvim

[github](https://github.com/lewis6991/gitsigns.nvim):
Git integration for buffers.
Super fast git decorations implemented purely in Lua.

Features
* Signs for added, removed, and changed lines
* Asynchronous using luv
* Navigation between hunks
* Stage hunks (with undo)
* Preview diffs of hunks (with word diff)
* Customizable (signs, highlights, mappings, etc)
* Status bar integration
* Git blame a specific line using virtual text.
* Hunk text object
* Automatically follow files moved in the index.
* Live intra-line word diff
* Ability to display deleted/changed lines via virtual lines.
* Support for yadm
* Support for detached working trees.


## vim-illuminate

[github](https://github.com/RRethy/vim-illuminate):
illuminate.vim - (Neo)Vim plugin for automatically highlighting other uses of the word under the cursor using either LSP, Tree-sitter, or regex matching.


## trouble.Nvim

[github](https://github.com/folke/trouble.nvim):
🚦 A pretty diagnostics, references, telescope results, quickfix and location list to help you solve all the trouble your code is causing.


## Language Sever Protocol (LSP)


## mason.nvim

[github](https://github.com/williamboman/mason.nvim):
Portable package manager for Neovim that runs everywhere Neovim runs.
Easily install and manage LSP servers, DAP servers, linters, and formatters.

`:h mason-introduction`

`mason.nvim` is a Neovim plugin that allows you to easily manage external editor tooling such as LSP servers, DAP servers, linters, and formatters through a single interface.
It runs everywhere Neovim runs (across Linux, macOS, Windows, etc.), with only a small set of [external requirements](https://github.com/williamboman/mason.nvim#requirements) needed.

Packages are installed in Neovim's data directory (`:h standard-path`) by default.
Executables are linked to a single `bin/` directory, which `mason.nvim` will add to Neovim's PATH during setup, allowing seamless access from Neovim builtins (shell, terminal, etc.) as well as other 3rd party plugins.

For a list of all available packages, see https://mason-registry.dev/registry/list.


## noice.nvim

[github](https://github.com/folke/noice.nvim):
💥 Highly experimental plugin that completely replaces the UI for messages, cmdline and the popupmenu.


## mini.indentscopre

[github](https://github.com/echasnovski/mini.indentscope):
Neovim Lua plugin to visualize and operate on indent scope. Part of 'mini.nvim' library.


## lualine.nvim

[github](https://github.com/nvim-lualine/lualine.nvim):
A blazing fast and easy to configure neovim statusline plugin written in pure lua.


## LuaSnip

[github](https://github.com/L3MON4D3/LuaSnip):
Snippet Engine for Neovim written in Lua.


## nvim-cmp

[github](https://github.com/hrsh7th/nvim-cmp):
A completion plugin for neovim coded in Lua.
A completion engine plugin for neovim written in Lua. Completion sources are installed from external repositories and "sourced".


## cmp-nvim-lsp

[github](https://github.com/hrsh7th/cmp-nvim-lsp):
nvim-cmp source for neovim builtin LSP client.

Language servers provide different completion results depending on the capabilities of the client.
Neovim's default omnifunc has basic support for serving completion candidates.
`nvim-cmp` supports more types of completion candidates, so users must override the capabilities sent to the server such that it can provide these candidates during a completion request.
These capabilities are provided via the helper function `require('cmp_nvim_lsp').default_capabilities`

As these candidates are sent on each request, adding these capabilities will break the built-in omnifunc support for neovim's language server client.
`nvim-cmp` provides manually triggered completion that can replace omnifunc.
See `:help cmp-faq` for more details.


## cmp-buffer

[github](https://github.com/hrsh7th/cmp-buffer):
nvim-cmp source for buffer words.


## cmp-path

[github](https://github.com/hrsh7th/cmp-path):
nvim-cmp source for filesystem paths.


## cmp_luasnip

[github](https://github.com/saadparwaiz1/cmp_luasnip):
luasnip completion source for nvim-cmp


## mini.pairs

[github](https://github.com/echasnovski/mini.pairs):
Neovim Lua plugin to automatically manage character pairs. Part of 'mini.nvim' library.


## mini.surround

[github](https://github.com/echasnovski/mini.surround):
Neovim Lua plugin with fast and feature-rich surround actions. Part of 'mini.nvim' library.


## mini.ai

[github](https://github.com/echasnovski/mini.ai):
Neovim Lua plugin to extend and create `a`/`i` textobjects. Part of 'mini.nvim' library.


## mini.comment

[github](https://github.com/echasnovski/mini.comment):
Neovim Lua plugin for fast and familiar per-line commenting. Part of 'mini.nvim' library.


## nvim-notify

[github](https://github.com/rcarriga/nvim-notify):
A fancy, configurable, notification manager for NeoVim.
