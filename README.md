<div align="center">

# From VSCode to Neovim — The Ultimate Guide

*A complete, from-scratch migration guide. Any language, any OS.*

![Neovim](https://img.shields.io/badge/Neovim-0.10+-57A143?style=flat-square&logo=neovim&logoColor=white)
![Lua](https://img.shields.io/badge/config-Lua-2C2D72?style=flat-square&logo=lua&logoColor=white)
![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen?style=flat-square)

**[English](#english)** · **[Русский](#русский)**

</div>

<br>

---

<a id="english"></a>

# English

> A practical guide for anyone leaving VSCode for Neovim and actually sticking with it. Built config, not a forked distro — you'll understand every line you add.

**Two paths to choose from:**

- **A prebuilt Neovim "distro"** (LazyVim / NvChad / AstroNvim) — install, works out of the box, tweak as you go. Fastest start.
- **A config from scratch** — slower to set up, but you understand every line and carry no dead weight. This guide mainly covers this path; section 7 covers the first one.

### Table of Contents

- [0. Before You Start](#0-before-you-start)
- [1. Vim Modes and Motions](#1-vim-modes-and-motions)
- [2. Installation](#2-installation)
- [3. Config Structure](#3-config-structure)
- [4. Core Options](#4-core-options)
- [5. Keymaps](#5-keymaps)
- [6. Essential Plugins](#6-essential-plugins)
  - [Colorscheme](#colorscheme)
  - [UI: Statusline and Keymap Hints](#ui-statusline-and-keymap-hints)
  - [File Explorer](#file-explorer)
  - [Fuzzy Finder](#fuzzy-finder)
  - [Treesitter](#treesitter)
  - [LSP and Auto-install](#lsp-and-auto-install)
  - [Autocompletion](#autocompletion)
  - [AI Autocompletion](#ai-autocompletion-if-you-used-copilot-in-vscode)
  - [Git](#git)
  - [Debugging](#debugging)
  - [Formatting and Linting](#formatting-and-linting)
  - [Optional but Useful](#optional-but-useful)
- [7. Prebuilt Neovim "Distros"](#7-prebuilt-neovim-distros)
- [8. VSCode to Neovim Cheatsheet](#8-vscode-to-neovim-cheatsheet)
- [9. Common Issues](#9-common-issues)
- [10. Migration Plan](#10-migration-plan)
- [Useful Resources](#useful-resources)

---

## 0. Before You Start

Neovim is a modal editor. The real difference from VSCode isn't the plugin set — it's the entire model of interacting with text. Plugins will cover the functionality (autocomplete, LSP, git integration), but until your fingers adapt to modal editing, everything will feel clunky. That's normal, and it passes after 1–3 weeks of regular use.

---

## 1. Vim Modes and Motions

- **Normal** (default) — movement and editing commands.
- **Insert** (`i`, `a`, `o`) — regular text typing.
- **Visual** (`v`, `V`, `Ctrl-v`) — selection: character-wise, line-wise, block-wise.
- **Command** (`:`) — `:w`, `:q`, `:%s/foo/bar/g`.

Core motions:

| Command | Action |
|---|---|
| `h j k l` | left / down / up / right |
| `w` / `b` / `e` | word forward / backward / end of word |
| `0` / `^` / `$` | start of line / first non-blank char / end of line |
| `gg` / `G` | start / end of file |
| `{` / `}` | previous / next paragraph |
| `%` | jump to matching bracket |
| `dd` / `yy` / `p` | delete line / yank (copy) line / paste |
| `ciw` / `caw` | change word / change word with surrounding space |
| `di(` / `ci"` | delete/change contents inside brackets or quotes |
| `u` / `Ctrl-r` | undo / redo |
| `.` | repeat last change |
| `/text` + `n`/`N` | search forward, next/previous match |

Vim's logic is **operator + motion**: `d` (delete) + `w` (word) = `dw`. This composes across almost every operator (`d`, `c`, `y`) and almost every motion/text object. Learn the principle instead of memorizing commands one by one, and progress is fast.

Run `vimtutor` in a terminal once — 25–30 minutes, covers 80% of the initial learning curve.

---

## 2. Installation

**Linux (package managers):**

```bash
# Arch / CachyOS / Manjaro
sudo pacman -S neovim ripgrep fd git unzip base-devel

# Ubuntu / Debian (repo version is often outdated — prefer a PPA
# or the official AppImage/tar.gz from neovim/neovim GitHub releases)
sudo apt install ripgrep fd-find git unzip build-essential

# Fedora
sudo dnf install neovim ripgrep fd-find git unzip
```

**macOS:**

```bash
brew install neovim ripgrep fd git
```

**Windows:**

```powershell
winget install Neovim.Neovim
# or
scoop install neovim ripgrep fd git
```

Also worth having:
- **A Nerd Font** (e.g. JetBrainsMono Nerd Font or FiraCode Nerd Font) — without one, icons in the file explorer and statusline show as boxes. Install it system-wide and select it as your terminal font.
- **Node.js/npm** — several LSP servers and formatters install through npm.
- **A C compiler** (gcc/clang) — needed to build Treesitter parsers.

---

## 3. Config Structure

```
~/.config/nvim/              (Linux/macOS)
%LOCALAPPDATA%\nvim\         (Windows)
├── init.lua
└── lua/
    ├── config/
    │   ├── options.lua
    │   ├── keymaps.lua
    │   └── autocmds.lua
    └── plugins/
        ├── colorscheme.lua
        ├── ui.lua
        ├── telescope.lua
        ├── treesitter.lua
        ├── lsp.lua
        ├── cmp.lua
        ├── git.lua
        ├── dap.lua
        └── editing.lua
```

`init.lua`:

```lua
local lazypath = vim.fn.stdpath("data") .. "/lazy/lazy.nvim"
if not vim.loop.fs_stat(lazypath) then
  vim.fn.system({
    "git", "clone", "--filter=blob:none",
    "https://github.com/folke/lazy.nvim.git",
    "--branch=stable", lazypath,
  })
end
vim.opt.rtp:prepend(lazypath)

require("config.options")
require("config.keymaps")
require("config.autocmds")

require("lazy").setup("plugins", {
  change_detection = { notify = false },
})
```

`lazy.nvim` automatically picks up every file in `lua/plugins/*.lua`. Plugins load lazily — on an event, a command, or a filetype — so startup stays fast even with dozens of plugins.

---

## 4. Core Options

`lua/config/options.lua`:

```lua
local opt = vim.opt

opt.number = true
opt.relativenumber = true
opt.mouse = "a"
opt.clipboard = "unnamedplus"   -- share the system clipboard; on Wayland you may need
                                 -- wl-clipboard, on X11 xclip/xsel; works out of the box
                                 -- on macOS/Windows
opt.ignorecase = true
opt.smartcase = true
opt.wrap = false
opt.scrolloff = 8
opt.signcolumn = "yes"
opt.updatetime = 250

opt.tabstop = 4
opt.shiftwidth = 4
opt.expandtab = true

opt.splitright = true
opt.splitbelow = true

opt.termguicolors = true
opt.undofile = true
```

---

## 5. Keymaps

`lua/config/keymaps.lua`:

```lua
vim.g.mapleader = " "

local map = vim.keymap.set

map({ "n", "i", "v" }, "<C-s>", "<cmd>w<cr><esc>", { desc = "Save" })
map("n", "<leader>q", "<cmd>q<cr>", { desc = "Quit" })

map("n", "<C-h>", "<C-w>h")
map("n", "<C-j>", "<C-w>j")
map("n", "<C-k>", "<C-w>k")
map("n", "<C-l>", "<C-w>l")

map("v", "J", ":m '>+1<cr>gv=gv")
map("v", "K", ":m '<-2<cr>gv=gv")

map("v", "<", "<gv")
map("v", ">", ">gv")

map("i", "jk", "<esc>")
```

---

## 6. Essential Plugins

For each category: a recommendation plus alternatives, in case the default choice isn't your taste.

### Colorscheme

Pick whichever popular one you like: **catppuccin**, **tokyonight**, **gruvbox.nvim**, **everforest**, **nord.nvim**, **kanagawa.nvim**.

```lua
-- lua/plugins/colorscheme.lua
return {
  "catppuccin/nvim",
  name = "catppuccin",
  lazy = false,
  priority = 1000,
  config = function()
    vim.cmd.colorscheme("catppuccin-mocha")
  end,
}
```

### UI: Statusline and Keymap Hints

```lua
-- lua/plugins/ui.lua
return {
  {
    "nvim-lualine/lualine.nvim",
    dependencies = { "nvim-tree/nvim-web-devicons" },
    opts = {},
  },
  {
    "folke/which-key.nvim",
    event = "VeryLazy",
    opts = {},
  },
}
```

### File Explorer

Three solid options — pick one:

- **neo-tree.nvim** — classic sidebar, closest feel to the VSCode Explorer.
- **nvim-tree.lua** — same idea, simpler and lighter.
- **oil.nvim** — the directory opens as a regular text buffer; rename a line, rename the file on disk. Less magic, takes getting used to.

```lua
-- lua/plugins/explorer.lua (neo-tree option — closest to VSCode)
return {
  "nvim-neo-tree/neo-tree.nvim",
  branch = "v3.x",
  dependencies = {
    "nvim-lua/plenary.nvim",
    "nvim-tree/nvim-web-devicons",
    "MunifTanjim/nui.nvim",
  },
  keys = {
    { "<leader>e", "<cmd>Neotree toggle<cr>", desc = "Explorer" },
  },
}
```

### Fuzzy Finder

Replaces Ctrl+P and project-wide search:

```lua
-- lua/plugins/telescope.lua
return {
  "nvim-telescope/telescope.nvim",
  dependencies = { "nvim-lua/plenary.nvim" },
  keys = {
    { "<leader>ff", "<cmd>Telescope find_files<cr>", desc = "Find files" },
    { "<leader>fg", "<cmd>Telescope live_grep<cr>", desc = "Grep across project" },
    { "<leader>fb", "<cmd>Telescope buffers<cr>", desc = "Buffers" },
    { "<leader>fh", "<cmd>Telescope help_tags<cr>", desc = "Help" },
  },
}
```

If you want more speed on very large monorepos, **fzf-lua** is the alternative.

### Treesitter

```lua
-- lua/plugins/treesitter.lua
return {
  "nvim-treesitter/nvim-treesitter",
  build = ":TSUpdate",
  opts = {
    ensure_installed = {
      "c", "cpp", "python", "javascript", "typescript", "tsx",
      "rust", "go", "lua", "bash", "json", "yaml", "markdown", "html", "css",
    },
    highlight = { enable = true },
    indent = { enable = true },
  },
  config = function(_, opts)
    require("nvim-treesitter.configs").setup(opts)
  end,
}
```

Trim `ensure_installed` to the languages you actually use — no reason to compile parsers you won't touch.

### LSP and Auto-install

```lua
-- lua/plugins/lsp.lua
return {
  { "williamboman/mason.nvim", opts = {} },
  {
    "williamboman/mason-lspconfig.nvim",
    dependencies = { "mason.nvim" },
    opts = {
      -- list your servers here, see the table below
      ensure_installed = { "lua_ls" },
    },
  },
  {
    "neovim/nvim-lspconfig",
    dependencies = { "mason-lspconfig.nvim" },
    config = function()
      local lspconfig = require("lspconfig")

      lspconfig.lua_ls.setup({})
      -- add setup({}) calls for the servers you need, see the table below

      vim.keymap.set("n", "gd", vim.lsp.buf.definition, { desc = "Go to definition" })
      vim.keymap.set("n", "gr", vim.lsp.buf.references, { desc = "References" })
      vim.keymap.set("n", "K", vim.lsp.buf.hover, { desc = "Hover docs" })
      vim.keymap.set("n", "<leader>rn", vim.lsp.buf.rename, { desc = "Rename" })
      vim.keymap.set("n", "<leader>ca", vim.lsp.buf.code_action, { desc = "Code action" })
    end,
  },
}
```

LSP servers for popular languages (name for `mason-lspconfig` and `lspconfig.<name>.setup({})`):

| Language | LSP server |
|---|---|
| C / C++ | `clangd` |
| Python | `pyright` (types) + `ruff` (lint/format) |
| JavaScript / TypeScript | `ts_ls` (formerly tsserver) or `vtsls` |
| Rust | `rust_analyzer` (better via the `rustaceanvim` plugin) |
| Go | `gopls` |
| Java | `jdtls` |
| HTML/CSS | `html`, `cssls` |
| Bash | `bashls` |
| Lua | `lua_ls` |
| Markdown | `marksman` |

### Autocompletion

```lua
-- lua/plugins/cmp.lua
return {
  "saghen/blink.cmp",
  dependencies = "rafamadriz/friendly-snippets",
  version = "*",
  opts = {
    keymap = { preset = "default" },
    appearance = { nerd_font_variant = "mono" },
    sources = { default = { "lsp", "path", "snippets", "buffer" } },
  },
}
```

**nvim-cmp** is the older, more established alternative with a larger extension ecosystem.

### AI Autocompletion (if you used Copilot in VSCode)

- **copilot.vim** — the official GitHub Copilot client, minimal, inline suggestions only.
- **avante.nvim** / **codecompanion.nvim** — a chat interface in the spirit of Cursor/Copilot Chat, works with multiple providers (including Claude via an API key).

### Git

```lua
-- lua/plugins/git.lua
return {
  { "lewis6991/gitsigns.nvim", opts = {} },  -- change markers in the gutter, blame, hunk navigation
  {
    "kdheepak/lazygit.nvim",
    keys = { { "<leader>gg", "<cmd>LazyGit<cr>", desc = "LazyGit" } },
  },
}
```

`lazygit` installs separately as a system package (`brew install lazygit`, `pacman -S lazygit`, `scoop install lazygit`) — a git TUI that covers most of what VSCode's Source Control panel does. In-buffer alternatives: **fugitive.vim** or **neogit**.

### Debugging

```lua
-- lua/plugins/dap.lua
return {
  {
    "mfussenegger/nvim-dap",
    dependencies = {
      "rcarriga/nvim-dap-ui",
      "nvim-neotest/nvim-nio",
      "jay-babu/mason-nvim-dap.nvim",
    },
    config = function()
      local dap, dapui = require("dap"), require("dapui")
      dapui.setup()
      dap.listeners.after.event_initialized["dapui_config"] = dapui.open
      dap.listeners.before.event_terminated["dapui_config"] = dapui.close

      vim.keymap.set("n", "<F5>", dap.continue, { desc = "Debug: Continue" })
      vim.keymap.set("n", "<F10>", dap.step_over, { desc = "Debug: Step over" })
      vim.keymap.set("n", "<F11>", dap.step_into, { desc = "Debug: Step into" })
      vim.keymap.set("n", "<leader>b", dap.toggle_breakpoint, { desc = "Toggle breakpoint" })
    end,
  },
  { "jay-babu/mason-nvim-dap.nvim", opts = { ensure_installed = {} } },
}
```

The adapter is language-specific: `codelldb` for C/C++/Rust, `debugpy` for Python, `delve` for Go, `js-debug-adapter` for Node/TS — install any of them through `mason-nvim-dap`'s `ensure_installed`.

### Formatting and Linting

```lua
-- lua/plugins/editing.lua
return {
  {
    "stevearc/conform.nvim",
    opts = {
      formatters_by_ft = {
        cpp = { "clang-format" },
        c = { "clang-format" },
        python = { "ruff_format" },
        javascript = { "prettier" },
        typescript = { "prettier" },
        rust = { "rustfmt" },
        lua = { "stylua" },
      },
      format_on_save = { timeout_ms = 500, lsp_fallback = true },
    },
  },
  { "mfussenegger/nvim-lint" },  -- for linters not covered by an LSP (eslint_d, etc.)

  -- small things to replace VSCode habits
  { "kylechui/nvim-surround", opts = {} },   -- wrap in quotes/brackets: ys, cs, ds
  { "numToStr/Comment.nvim", opts = {} },    -- gcc, like Ctrl+/
  { "windwp/nvim-autopairs", opts = {} },    -- auto-close brackets
}
```

### Optional but Useful

- **toggleterm.nvim** — a floating/side terminal inside Neovim, like VSCode's built-in terminal.
- **persistence.nvim** — save and restore sessions (open files, window layout).
- **neotest** — run tests from inside the editor with inline results.
- **trouble.nvim** — a unified list of LSP errors/warnings across the whole project, like the Problems panel.

---

## 7. Prebuilt Neovim "Distros"

If building from scratch isn't appealing, mature preconfigured setups exist, installed with a single `git clone`:

| Distro | What it's known for |
|---|---|
| **kickstart.nvim** | A single ~500-line file, heavily commented. Less a distro, more a teaching starter config — meant to be read and taken apart. |
| **LazyVim** | Full-featured, well documented, modular (toggle language "extras" with one line). The most popular pick for a "VSCode experience out of the box". |
| **NvChad** | Focused on startup speed and good looks out of the box. |
| **AstroNvim** | Similar spirit to LazyVim, separate community and per-language "packs". |

A reasonable path: try LazyVim for a couple of weeks, see which modules you actually use versus which are dead weight, then gradually port what you liked into your own from-scratch config.

---

## 8. VSCode to Neovim Cheatsheet

| VSCode | Neovim |
|---|---|
| Ctrl+P (quick file open) | `<leader>ff` (telescope) |
| Ctrl+Shift+F (project search) | `<leader>fg` (telescope live_grep) |
| Ctrl+/ | `gcc` (Comment.nvim) |
| F12 / Go to Definition | `gd` |
| Shift+F12 / Find References | `gr` |
| F2 / Rename Symbol | `<leader>rn` |
| Ctrl+. / Quick Fix | `<leader>ca` |
| Source Control panel | `<leader>gg` (lazygit) |
| Problems panel | `trouble.nvim` |
| Ctrl+\` (terminal) | `:terminal` or toggleterm; exit terminal mode with `<C-\><C-n>` |
| Ctrl+D (multi-cursor) | No 1:1 equivalent: `:%s/foo/bar/g`, or `cgn` + `.` to repeat a replacement |
| Explorer sidebar | `<leader>e` (neo-tree/nvim-tree) or `-` (oil.nvim) |
| Zen Mode | `folke/zen-mode.nvim` |

---

## 9. Common Issues

- **Icons show as boxes or garbled glyphs.** No Nerd Font installed/selected in the terminal.
- **Treesitter fails to build.** No C compiler on the system (`build-essential` / `base-devel` / Xcode Command Line Tools / `gcc` via MSYS2 on Windows).
- **`unnamedplus` doesn't sync with the system clipboard.** On Wayland you need `wl-clipboard`, on X11 `xclip` or `xsel`.
- **LSP doesn't start.** Check `:LspInfo` — the server is either not installed via Mason (`:Mason`) or not bound to the right filetype.
- **Colors look washed out or wrong.** The terminal doesn't support true color — check `opt.termguicolors` and that your terminal emulator (kitty, alacritty, wezterm — fine; old xterm — not fine) supports 24-bit color.

---

## 10. Migration Plan

1. **Week 1** — `vimtutor`, edit small files in Neovim, VSCode stays your daily driver.
2. **Week 2** — build (or install) the config from this guide, move one full project over to Neovim.
3. **Week 3** — LSP and debugging set up for your main stack, VSCode gets opened less and less.
4. From here — keep VSCode installed for a month or two "just in case", but you won't actually reach for it.

---

## Useful Resources

- `:help` — Neovim's built-in docs, more thorough and accurate than any third-party guide.
- **kickstart.nvim** (GitHub, nvim-lua/kickstart.nvim) — a readable, ready-made skeleton to start from.
- Communities: r/neovim, the Neovim Discord, the neovim tag on Stack Overflow.

<br>

**[⬆ Back to top](#from-vscode-to-neovim--the-ultimate-guide)**

---

<a id="русский"></a>

# Русский

> Практический гайд для тех, кто хочет пересесть с VSCode на Neovim и не бросить это дело на второй день. Полностью собранный с нуля конфиг, а не форк готового дистрибутива — понимаешь каждую строчку, которую добавляешь.

**Два пути на выбор:**

- **Готовый "дистрибутив" Neovim** (LazyVim / NvChad / AstroNvim) — ставишь, работает из коробки, донастраиваешь по мере надобности. Быстрый старт.
- **Конфиг с нуля** — дольше, зато понимаешь каждую строчку и ничего лишнего. Этот гайд в основном про второй путь, раздел 7 разбирает первый.

### Оглавление

- [0. Что нужно знать заранее](#0-что-нужно-знать-заранее)
- [1. Матчасть: режимы и базовые движения](#1-матчасть-режимы-и-базовые-движения)
- [2. Установка](#2-установка)
- [3. Структура конфига](#3-структура-конфига)
- [4. Базовые опции](#4-базовые-опции)
- [5. Кеймапы](#5-кеймапы)
- [6. Плагины: обязательный набор](#6-плагины-обязательный-набор)
  - [Тема](#тема)
  - [UI: статусбар и подсказки клавиш](#ui-статусбар-и-подсказки-клавиш)
  - [Файловый менеджер](#файловый-менеджер)
  - [Fuzzy finder](#fuzzy-finder-1)
  - [Treesitter](#treesitter-1)
  - [LSP и автоустановка серверов](#lsp-и-автоустановка-серверов)
  - [Автодополнение](#автодополнение)
  - [AI-автодополнение](#ai-автодополнение-если-пользовался-copilot-в-vscode)
  - [Git](#git-1)
  - [Отладка](#отладка)
  - [Форматирование и линтинг](#форматирование-и-линтинг)
  - [Опционально, но полезно](#опционально-но-полезно)
- [7. Готовые "дистрибутивы" Neovim](#7-готовые-дистрибутивы-neovim)
- [8. Таблица соответствий: VSCode и Neovim](#8-таблица-соответствий-vscode-и-neovim)
- [9. Частые проблемы на старте](#9-частые-проблемы-на-старте)
- [10. План перехода без резкого прыжка](#10-план-перехода-без-резкого-прыжка)
- [Полезные ресурсы](#полезные-ресурсы)

---

## 0. Что нужно знать заранее

Neovim — модальный редактор. Основная разница с VSCode не в наборе плагинов, а в самой модели взаимодействия с текстом. Плагины закроют функциональность (автодополнение, LSP, git-интеграцию), но пока пальцы не привыкнут к модальному редактированию, будет ощущение "почему всё так неудобно". Это нормально и проходит за 1–3 недели постоянного использования.

---

## 1. Матчасть: режимы и базовые движения

- **Normal** (по умолчанию) — команды перемещения и редактирования.
- **Insert** (`i`, `a`, `o`) — обычный ввод текста.
- **Visual** (`v`, `V`, `Ctrl-v`) — выделение: посимвольное, построчное, блочное.
- **Command** (`:`) — `:w`, `:q`, `:%s/foo/bar/g`.

Базовый набор движений:

| Команда | Действие |
|---|---|
| `h j k l` | влево / вниз / вверх / вправо |
| `w` / `b` / `e` | слово вперёд / назад / конец слова |
| `0` / `^` / `$` | начало строки / первый непробельный символ / конец строки |
| `gg` / `G` | начало / конец файла |
| `{` / `}` | предыдущий / следующий абзац |
| `%` | к парной скобке |
| `dd` / `yy` / `p` | удалить строку / скопировать строку / вставить |
| `ciw` / `caw` | сменить слово / сменить слово с пробелом |
| `di(` / `ci"` | удалить/сменить содержимое внутри скобок или кавычек |
| `u` / `Ctrl-r` | отмена / повтор |
| `.` | повторить последнее изменение |
| `/text` + `n`/`N` | поиск вперёд, следующее/предыдущее совпадение |

Логика Vim — комбинация **оператор + движение**: `d` (удалить) + `w` (слово) = `dw`. Это работает почти со всеми операторами (`d`, `c`, `y`) и почти всеми движениями/текстовыми объектами. Освоив принцип, а не зубря команды по одной, прогресс идёт быстро.

Пройди `vimtutor` (или `vimtutor ru`) в терминале один раз — 25–30 минут, закрывает 80% первичного барьера.

---

## 2. Установка

**Linux (пакетные менеджеры):**

```bash
# Arch / CachyOS / Manjaro
sudo pacman -S neovim ripgrep fd git unzip base-devel

# Ubuntu / Debian (в репозиториях часто старая версия — лучше через apt из PPA
# или официальный AppImage/tar.gz с GitHub-релизов neovim/neovim)
sudo apt install ripgrep fd-find git unzip build-essential

# Fedora
sudo dnf install neovim ripgrep fd-find git unzip
```

**macOS:**

```bash
brew install neovim ripgrep fd git
```

**Windows:**

```powershell
winget install Neovim.Neovim
# или
scoop install neovim ripgrep fd git
```

Дополнительно пригодятся:
- **Nerd Font** (например JetBrainsMono Nerd Font или FiraCode Nerd Font) — без него иконки в файловом менеджере и статусбаре будут квадратиками. Ставится в систему и выбирается как шрифт терминала.
- **Node.js/npm** — часть LSP-серверов и форматтеров ставится через npm.
- **Компилятор C** (gcc/clang) — нужен для сборки Treesitter-парсеров.

---

## 3. Структура конфига

```
~/.config/nvim/              (Linux/macOS)
%LOCALAPPDATA%\nvim\         (Windows)
├── init.lua
└── lua/
    ├── config/
    │   ├── options.lua
    │   ├── keymaps.lua
    │   └── autocmds.lua
    └── plugins/
        ├── colorscheme.lua
        ├── ui.lua
        ├── telescope.lua
        ├── treesitter.lua
        ├── lsp.lua
        ├── cmp.lua
        ├── git.lua
        ├── dap.lua
        └── editing.lua
```

`init.lua`:

```lua
local lazypath = vim.fn.stdpath("data") .. "/lazy/lazy.nvim"
if not vim.loop.fs_stat(lazypath) then
  vim.fn.system({
    "git", "clone", "--filter=blob:none",
    "https://github.com/folke/lazy.nvim.git",
    "--branch=stable", lazypath,
  })
end
vim.opt.rtp:prepend(lazypath)

require("config.options")
require("config.keymaps")
require("config.autocmds")

require("lazy").setup("plugins", {
  change_detection = { notify = false },
})
```

`lazy.nvim` подхватывает каждый файл из `lua/plugins/*.lua` — плагины грузятся лениво, по событию/команде/типу файла, конфиг стартует быстро даже с полусотней плагинов.

---

## 4. Базовые опции

`lua/config/options.lua`:

```lua
local opt = vim.opt

opt.number = true
opt.relativenumber = true
opt.mouse = "a"
opt.clipboard = "unnamedplus"   -- общий буфер с системой; на Wayland иногда нужен wl-clipboard,
                                 -- на X11 — xclip/xsel, на macOS/Windows работает из коробки
opt.ignorecase = true
opt.smartcase = true
opt.wrap = false
opt.scrolloff = 8
opt.signcolumn = "yes"
opt.updatetime = 250

opt.tabstop = 4
opt.shiftwidth = 4
opt.expandtab = true

opt.splitright = true
opt.splitbelow = true

opt.termguicolors = true
opt.undofile = true
```

---

## 5. Кеймапы

`lua/config/keymaps.lua`:

```lua
vim.g.mapleader = " "

local map = vim.keymap.set

map({ "n", "i", "v" }, "<C-s>", "<cmd>w<cr><esc>", { desc = "Save" })
map("n", "<leader>q", "<cmd>q<cr>", { desc = "Quit" })

map("n", "<C-h>", "<C-w>h")
map("n", "<C-j>", "<C-w>j")
map("n", "<C-k>", "<C-w>k")
map("n", "<C-l>", "<C-w>l")

map("v", "J", ":m '>+1<cr>gv=gv")
map("v", "K", ":m '<-2<cr>gv=gv")

map("v", "<", "<gv")
map("v", ">", ">gv")

map("i", "jk", "<esc>")
```

---

## 6. Плагины: обязательный набор

Для каждой категории — рекомендация и альтернативы, если стандартный выбор не нравится.

### Тема

Рекомендация — любая из популярных, дело вкуса: **catppuccin**, **tokyonight**, **gruvbox.nvim**, **everforest**, **nord.nvim**, **kanagawa.nvim**.

```lua
-- lua/plugins/colorscheme.lua
return {
  "catppuccin/nvim",
  name = "catppuccin",
  lazy = false,
  priority = 1000,
  config = function()
    vim.cmd.colorscheme("catppuccin-mocha")
  end,
}
```

### UI: статусбар и подсказки клавиш

```lua
-- lua/plugins/ui.lua
return {
  {
    "nvim-lualine/lualine.nvim",
    dependencies = { "nvim-tree/nvim-web-devicons" },
    opts = {},
  },
  {
    "folke/which-key.nvim",
    event = "VeryLazy",
    opts = {},
  },
}
```

### Файловый менеджер

Три рабочих варианта — выбери один:

- **neo-tree.nvim** — классическая боковая панель, максимально похоже на VSCode Explorer.
- **nvim-tree.lua** — то же самое, но проще и легче.
- **oil.nvim** — директория открывается как обычный текстовый буфер; переименовал строку — переименовал файл на диске. Меньше магии, непривычно на старте.

```lua
-- lua/plugins/explorer.lua (вариант neo-tree — самый VSCode-подобный)
return {
  "nvim-neo-tree/neo-tree.nvim",
  branch = "v3.x",
  dependencies = {
    "nvim-lua/plenary.nvim",
    "nvim-tree/nvim-web-devicons",
    "MunifTanjim/nui.nvim",
  },
  keys = {
    { "<leader>e", "<cmd>Neotree toggle<cr>", desc = "Explorer" },
  },
}
```

### Fuzzy finder

Замена Ctrl+P и глобального поиска:

```lua
-- lua/plugins/telescope.lua
return {
  "nvim-telescope/telescope.nvim",
  dependencies = { "nvim-lua/plenary.nvim" },
  keys = {
    { "<leader>ff", "<cmd>Telescope find_files<cr>", desc = "Find files" },
    { "<leader>fg", "<cmd>Telescope live_grep<cr>", desc = "Grep across project" },
    { "<leader>fb", "<cmd>Telescope buffers<cr>", desc = "Buffers" },
    { "<leader>fh", "<cmd>Telescope help_tags<cr>", desc = "Help" },
  },
}
```

Альтернатива, если хочется скорости на очень больших монорепах — **fzf-lua**.

### Treesitter

```lua
-- lua/plugins/treesitter.lua
return {
  "nvim-treesitter/nvim-treesitter",
  build = ":TSUpdate",
  opts = {
    ensure_installed = {
      "c", "cpp", "python", "javascript", "typescript", "tsx",
      "rust", "go", "lua", "bash", "json", "yaml", "markdown", "html", "css",
    },
    highlight = { enable = true },
    indent = { enable = true },
  },
  config = function(_, opts)
    require("nvim-treesitter.configs").setup(opts)
  end,
}
```

Список `ensure_installed` подгони под свои языки — незачем компилировать парсеры, которыми не пользуешься.

### LSP и автоустановка серверов

```lua
-- lua/plugins/lsp.lua
return {
  { "williamboman/mason.nvim", opts = {} },
  {
    "williamboman/mason-lspconfig.nvim",
    dependencies = { "mason.nvim" },
    opts = {
      -- список серверов под свой стек, см. таблицу ниже
      ensure_installed = { "lua_ls" },
    },
  },
  {
    "neovim/nvim-lspconfig",
    dependencies = { "mason-lspconfig.nvim" },
    config = function()
      local lspconfig = require("lspconfig")

      lspconfig.lua_ls.setup({})
      -- добавляй нужные setup({}) по таблице ниже

      vim.keymap.set("n", "gd", vim.lsp.buf.definition, { desc = "Go to definition" })
      vim.keymap.set("n", "gr", vim.lsp.buf.references, { desc = "References" })
      vim.keymap.set("n", "K", vim.lsp.buf.hover, { desc = "Hover docs" })
      vim.keymap.set("n", "<leader>rn", vim.lsp.buf.rename, { desc = "Rename" })
      vim.keymap.set("n", "<leader>ca", vim.lsp.buf.code_action, { desc = "Code action" })
    end,
  },
}
```

Таблица LSP-серверов под популярные языки (имя для `mason-lspconfig` и `lspconfig.<имя>.setup({})`):

| Язык | LSP-сервер |
|---|---|
| C / C++ | `clangd` |
| Python | `pyright` (типы) + `ruff` (линт/формат) |
| JavaScript / TypeScript | `ts_ls` (бывш. tsserver) или `vtsls` |
| Rust | `rust_analyzer` (лучше через плагин `rustaceanvim`) |
| Go | `gopls` |
| Java | `jdtls` |
| HTML/CSS | `html`, `cssls` |
| Bash | `bashls` |
| Lua | `lua_ls` |
| Markdown | `marksman` |

### Автодополнение

```lua
-- lua/plugins/cmp.lua
return {
  "saghen/blink.cmp",
  dependencies = "rafamadriz/friendly-snippets",
  version = "*",
  opts = {
    keymap = { preset = "default" },
    appearance = { nerd_font_variant = "mono" },
    sources = { default = { "lsp", "path", "snippets", "buffer" } },
  },
}
```

Альтернатива с более долгой историей и большим числом расширений — **nvim-cmp**.

### AI-автодополнение (если пользовался Copilot в VSCode)

- **copilot.vim** — официальный клиент GitHub Copilot, минимальный, только inline-предложения.
- **avante.nvim** / **codecompanion.nvim** — чат-интерфейс в духе Cursor/Copilot Chat, работает с разными провайдерами (включая Claude через API-ключ).

### Git

```lua
-- lua/plugins/git.lua
return {
  { "lewis6991/gitsigns.nvim", opts = {} },  -- значки изменений на полях, blame, hunk-навигация
  {
    "kdheepak/lazygit.nvim",
    keys = { { "<leader>gg", "<cmd>LazyGit<cr>", desc = "LazyGit" } },
  },
}
```

`lazygit` ставится отдельно как системный пакет (`brew install lazygit`, `pacman -S lazygit`, `scoop install lazygit`) — TUI для git, закрывает панель Source Control из VSCode почти полностью. Альтернативы прямо в буфере — **fugitive.vim** или **neogit**.

### Отладка

```lua
-- lua/plugins/dap.lua
return {
  {
    "mfussenegger/nvim-dap",
    dependencies = {
      "rcarriga/nvim-dap-ui",
      "nvim-neotest/nvim-nio",
      "jay-babu/mason-nvim-dap.nvim",
    },
    config = function()
      local dap, dapui = require("dap"), require("dapui")
      dapui.setup()
      dap.listeners.after.event_initialized["dapui_config"] = dapui.open
      dap.listeners.before.event_terminated["dapui_config"] = dapui.close

      vim.keymap.set("n", "<F5>", dap.continue, { desc = "Debug: Continue" })
      vim.keymap.set("n", "<F10>", dap.step_over, { desc = "Debug: Step over" })
      vim.keymap.set("n", "<F11>", dap.step_into, { desc = "Debug: Step into" })
      vim.keymap.set("n", "<leader>b", dap.toggle_breakpoint, { desc = "Toggle breakpoint" })
    end,
  },
  { "jay-babu/mason-nvim-dap.nvim", opts = { ensure_installed = {} } },
}
```

Адаптер под язык добавляется отдельно: `codelldb` для C/C++/Rust, `debugpy` для Python, `delve` для Go, `js-debug-adapter` для Node/TS — всё ставится через `mason-nvim-dap` в `ensure_installed`.

### Форматирование и линтинг

```lua
-- lua/plugins/editing.lua
return {
  {
    "stevearc/conform.nvim",
    opts = {
      formatters_by_ft = {
        cpp = { "clang-format" },
        c = { "clang-format" },
        python = { "ruff_format" },
        javascript = { "prettier" },
        typescript = { "prettier" },
        rust = { "rustfmt" },
        lua = { "stylua" },
      },
      format_on_save = { timeout_ms = 500, lsp_fallback = true },
    },
  },
  { "mfussenegger/nvim-lint" },  -- для линтеров, которые не встроены в LSP (eslint_d и т.п.)

  -- мелочи из VSCode-привычек
  { "kylechui/nvim-surround", opts = {} },   -- обёртка в кавычки/скобки: ys, cs, ds
  { "numToStr/Comment.nvim", opts = {} },    -- gcc — как Ctrl+/
  { "windwp/nvim-autopairs", opts = {} },    -- автозакрытие скобок
}
```

### Опционально, но полезно

- **toggleterm.nvim** — плавающий/боковой терминал внутри Neovim, аналог встроенного терминала VSCode.
- **persistence.nvim** — сохранение и восстановление сессии (открытые файлы, layout окон).
- **neotest** — запуск тестов прямо из редактора с результатами инлайн.
- **trouble.nvim** — единый список ошибок/предупреждений LSP по всему проекту, аналог панели Problems.

---

## 7. Готовые "дистрибутивы" Neovim

Если собирать с нуля не хочется — есть зрелые преднастроенные конфиги, устанавливаются одной командой git clone:

| Дистрибутив | Особенность |
|---|---|
| **kickstart.nvim** | Один файл ~500 строк с подробными комментариями. Не столько дистрибутив, сколько обучающий стартовый конфиг — задуман, чтобы его читали и растаскивали под себя. |
| **LazyVim** | Полноценный, хорошо задокументированный, модульный (включаешь "extras" под нужный язык одной строкой). Самый популярный вариант для тех, кто хочет "VSCode experience из коробки". |
| **NvChad** | Акцент на скорость запуска и внешний вид из коробки, у автора приятная тема/UI по умолчанию. |
| **AstroNvim** | Похож на LazyVim по духу, отдельное коммьюнити и набор "packs" под языки. |

Логичный путь: попробовать LazyVim пару недель, посмотреть какие модули реально используются, а какие — балласт, и по желанию постепенно вынести понравившееся в свой конфиг с нуля.

---

## 8. Таблица соответствий: VSCode и Neovim

| VSCode | Neovim |
|---|---|
| Ctrl+P (быстрый поиск файла) | `<leader>ff` (telescope) |
| Ctrl+Shift+F (поиск по проекту) | `<leader>fg` (telescope live_grep) |
| Ctrl+/ | `gcc` (Comment.nvim) |
| F12 / Go to Definition | `gd` |
| Shift+F12 / Find References | `gr` |
| F2 / Rename Symbol | `<leader>rn` |
| Ctrl+. / Quick Fix | `<leader>ca` |
| Панель Source Control | `<leader>gg` (lazygit) |
| Панель Problems | `trouble.nvim` |
| Ctrl+` (терминал) | `:terminal` или toggleterm; выход из терминального режима — `<C-\><C-n>` |
| Ctrl+D (multi-cursor) | Аналога нет 1:1: `:%s/foo/bar/g`, либо `cgn` + `.` для последовательной замены |
| Explorer слева | `<leader>e` (neo-tree/nvim-tree) или `-` (oil.nvim) |
| Zen Mode | `folke/zen-mode.nvim` |

---

## 9. Частые проблемы на старте

- **Иконки — квадратики или кракозябры.** Не установлен/не выбран Nerd Font в терминале.
- **Treesitter не собирается.** Нет компилятора C в системе (`build-essential` / `base-devel` / Xcode Command Line Tools / `gcc` через MSYS2 на Windows).
- **`unnamedplus` не работает с системным буфером.** На Wayland нужен `wl-clipboard`, на X11 — `xclip` или `xsel`.
- **LSP не запускается.** Проверить `:LspInfo` — сервер либо не установлен через Mason (`:Mason`), либо не привязан к нужному filetype.
- **Цвета выглядят блёкло/неправильно.** Терминал не поддерживает true color — проверить `opt.termguicolors` и что сам эмулятор терминала (kitty, alacritty, wezterm — ок; старый xterm — не ок) поддерживает 24-битный цвет.

---

## 10. План перехода без резкого прыжка

1. **Неделя 1** — `vimtutor`, редактирование мелких файлов в Neovim, VSCode остаётся основным.
2. **Неделя 2** — собранный по этому гайду (или установленный) конфиг, один проект целиком переезжает в Neovim.
3. **Неделя 3** — LSP и отладка настроены под основной стек, VSCode открывается всё реже.
4. Дальше — VSCode можно оставить установленным ещё месяц-два "на всякий", но по факту не открывается.

---

## Полезные ресурсы

- `:help` — встроенная документация Neovim, полнее и точнее любого стороннего гайда.
- **kickstart.nvim** (GitHub, nvim-lua/kickstart.nvim) — если нужен готовый читаемый каркас.
- Сообщества: r/neovim, канал Neovim в Discord, тег neovim на Habr/Stack Overflow.

<br>

**[⬆ Наверх](#русский)**

