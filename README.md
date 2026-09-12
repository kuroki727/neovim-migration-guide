# Ultimate-гайд: полный переход с VSCode на Neovim

Универсальная версия — без привязки к конкретному языку или дистрибутиву. Подходит фронтендеру на TypeScript, питонисту, рустоводу и системщику на C++ в равной степени. Где есть развилки — даны альтернативы, выбирай под себя.

Два пути на выбор:

- **Готовый "дистрибутив" Neovim** (LazyVim / NvChad / AstroNvim) — ставишь, работает из коробки, донастраиваешь по мере надобности. Быстрый старт.
- **Конфиг с нуля** — дольше, зато понимаешь каждую строчку и ничего лишнего. Этот гайд в основном про второй путь, но раздел 8 разбирает первый.

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

### UI: статусбар, иконки, подсказки клавиш

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

### Fuzzy finder (замена Ctrl+P и глобального поиска)

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

### LSP + автоустановка серверов

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

## 8. Таблица соответствия VSCode → Neovim

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
