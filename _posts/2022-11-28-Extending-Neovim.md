---
layout:     post
title:      Extending Neovim - LSPs, Linters, Formatters, Treesitter, and the DAP
date:       2022-11-28 13:20
type:       post
draft:      true
---

## Abstract

This article aims to describe the relationship between the various different plugins and
technologies Neovim leverages.

This article aims to describe how to effectively extend Neovim in a way that is easy to
maintain.

First we'll clarify which technologies we'll be using, then how we use them, then how we
can diagnose issues, and finally, how we can learn and improve.

## Understanding the Technologies

### LSP - the Language Server Protocol

The Langauge Server Protocol was introduced to improve editor performance.

NeoVIM added LSP support in whenever and describes it as follows:
> LSP facilitates features like go-to-definition, find-references, hover, completion, rename, format, refactor, etc., using semantic whole-project analysis (unlike ctags).

For each filetype opened an LSP client will connect to an LSP server and depending on
the server, a number of features become available, the most useful of which are probably:
* completion
* linting
* formatting
* hover-signatures

Somewhat confusingly, not all servers support all features and so sometimes it's
necessary to fall-back to executing a program to perform some feature (i.e: linting, or
formatting) for you.

In practice this means you may need to separately configure your LSP client, formatter(s), and a linter for every filetype that you wish to have these features for. This can be complicated to get right and in the fast-paced world of neovim means that your configuration can break often as things change so rapidly.

So, there is a solution - a community maintained set of configurations that handle most
of this for most stuff.

### Linters

Linters check code for common problems.

### Formatters

Formatters format code to conform to a specific style.

### Treesitter

Treesitter builds an internal graph representation of your code which can be used by
plugins authors to write plugins and for better than normal syntax highlighting.

### Dap - the Debugger Adapter Protocol

> nvim-dap is a Debug Adapter Protocol client implementation for Neovim. nvim-dap allows you to:
> * Launch an application to debug
> * Attach to running applications and debug them
> * Set breakpoints and step through code
> * Inspect the state of the application

## My Approach 

LunarVim is described as "An IDE layer for Neovim with sane defaults. Completely free and community driven.". LunarVIM adds a good set of default plugins to NeoVIM with configurations that will suit most people, and more importantly to me, it comes with all the essentials pre-configured but also allows customisation (enabling/disabling/configuration) and extension using additional plugins.

The default plugin list can be found [here](https://www.lunarvim.org/docs/plugins/core-plugins-list), along with a list of extra plugins [here](https://www.lunarvim.org/docs/plugins/extra-plugins).

Default vim settings: https://github.com/LunarVim/LunarVim/blob/master/lua/lvim/config/settings.lua

### Neovim Plugins which Solve Problems

* nvim-lspconfig - configs to connect the built-in lsp client to lsp servers
* nvim-lsp-installer - originally used to install lsp /servers/, now replaced by Mason.
* Mason - a plugin which can be used to install and manage LSP servers, DAP servers, linters, and formatters
* mason-lspconfig - This bridges the gap between nvim-lspconfig and mason - registering
* mason-tool-installer - 
  LSP configs with neovim so the LSP client can connect to the servers
* null-ls - allow hooking things into the LSP client - this is used to, for example,
  hook programmes that are not LSP servers into the LSP client such as formatters, linters, etc. that are not LSP servers themselves.

mason + null-ls https://github.com/jose-elias-alvarez/null-ls.nvim/blob/main/doc/BUILTINS.md


## What to do when editing a new file type

Langauge servers (LSPs)

```
# Show available language servers
LspInstall <filetype>

# Inspect which formatters and linters are attached to the buffer
LspInfo
# -or-
LvimInfo
```

Ensure treesitter parser to ensure highlighting works
```
TSInstall <filetype>
# Show installed and available parsers
TSInstallInfo
```

Optionally configure formatter and linter outside of LSP
```lua
-- set a formatter, this will override the language server formatting capabilities (if it exists)
local formatters = require "lvim.lsp.null-ls.formatters"
formatters.setup {
  { command = "black", filetypes = { "python" } },
  { command = "isort", filetypes = { "python" } },
  { command = "shfmt", filetypes = { "sh" } },
  { command = "terraform_fmt", filtypes = { "terraform" } },
  {
    command = "prettier",
    -- extra_args = { "--print-with", "100" },
    filetypes = { "typescript", "typescriptreact" },
  },
}

--
-- Linting
--

local linters = require "lvim.lsp.null-ls.linters"
linters.setup {
  { command = "flake8", filetypes = { "python" } },
  { command = "shellcheck", extra_args = { "--severity", "warning" }, },
  { command = "codespell", filetypes = { "javascript", "python" },
  },
}
```

## CheatSheet

### Movement

```
# Left, down, up, right
h
j
k
l
```

```
# Previous/next paragraph
{
}
```

```
# Next/previous block
[
]
```

```
# Top/bottom of file
gg
G
```


### Comment Management

```
# Toggle comments
<leader>-/
```

### Search and Replace

```
/<pattern>
```

```
# Disable highlight after search
<leader>-h
```

```
# press * on a word (or visual selection)
*
# double // represents selected string
:%s//replace/gc
```

```
# Delete "whatever" from every open buffer
bufdo exe ":%g/whatever/d" | w
```

### Diagnostis

```
# Open diagnostics (Trouble plugin)
# Switch buffers with ctrl-j/k
<leader>-t
```

```
# Toggle inline diagnostics
<leader>--
```

```
# Next/previous diagnostics
]d
[d
```

### Introspection

Show hint
```
<shift>-k
```

Goto definition
```
gd
```

### File Management

```
# Open file explorer
<leader>-e
```

```
# Prev/next buffer
<shift>-h
<shift>-l
```

```
# Close buffer
<leader>-c
```

```
# Fuzzy switch between buffers
<leader>-f 
```

### Completion

#### Functions, etc.

#### Snippets