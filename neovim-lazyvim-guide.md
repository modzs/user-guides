# NeoVim and LazyVim Guide for Beginners

A complete guide to getting started with NeoVim and LazyVim, from installation to everyday workflows.

**Platform Support:** This guide covers macOS, Ubuntu/Debian Linux, and Arch/Omarchy Linux. The core features are identical across all platforms - only installation methods differ.

**Version note:** Everything in this guide was checked against **NeoVim 0.12.5** and **LazyVim 16.0.0**, plus the official LazyVim documentation at <https://lazyvim.org>. LazyVim moves, so if `:checkhealth` on your machine disagrees with this page, your machine is right.

## Table of Contents
1. [What are NeoVim and LazyVim?](#what-are-neovim-and-lazyvim)
2. [Before You Install](#before-you-install)
3. [Installing NeoVim](#installing-neovim)
4. [Installing LazyVim](#installing-lazyvim)
5. [Your First Time Opening NeoVim](#your-first-time-opening-neovim)
6. [Understanding Modes](#understanding-modes)
7. [Navigation](#navigation)
8. [Basic Editing](#basic-editing)
9. [Common Tasks](#common-tasks)
10. [LazyVim Essentials](#lazyvim-essentials)
11. [Settings and Configuration](#settings-and-configuration)
12. [File Operations](#file-operations)
13. [Search and Replace](#search-and-replace)
14. [Windows and Splits](#windows-and-splits)
15. [Keeping LazyVim Updated](#keeping-lazyvim-updated)
16. [Essential Keybindings Reference](#essential-keybindings-reference)
17. [Tips for Getting Better](#tips-for-getting-better)
18. [Troubleshooting](#troubleshooting)
19. [Next Steps](#next-steps)

---

## What are NeoVim and LazyVim?

**NeoVim** is a modern text editor that's a fork and enhancement of Vim. It's powerful, fast, and works entirely from the keyboard - no need to reach for the mouse.

**LazyVim** is a starter configuration for NeoVim that comes pre-configured with useful plugins and settings. It makes NeoVim easier to use for beginners while being customizable for advanced users.

Think of it this way: NeoVim is the engine, LazyVim is the car with all the features already installed and set up nicely.

---

## Before You Install

Read this page before you run a single command. Two of the four things below are the reason most beginners get stuck on day one, and both are trivial to avoid if you know about them in advance.

### LazyVim needs a recent NeoVim

**LazyVim requires NeoVim 0.11.2 or newer**, built with LuaJIT. Older versions do not merely misbehave - LazyVim refuses to start and hands you an error that does not obviously say "your editor is too old".

Check what you have:

```bash
nvim --version
```

The first three lines tell you everything:

```
NVIM v0.12.5
Build type: RelWithDebInfo
LuaJIT 2.1.1787165859
```

Version `0.12.5`, and `LuaJIT` on the third line. That machine is fine.

### ⚠️ Your distribution's package is probably too old

This is the trap. `sudo apt-get install neovim` is the obvious first thing to try, and on every current Debian and Ubuntu release it installs a NeoVim that LazyVim will not run:

| Release | NeoVim in the default repos | Enough for LazyVim? |
|---------|-----------------------------|---------------------|
| Debian 12 (bookworm) | 0.7.2 | No |
| Debian 13 (trixie) | 0.10.4 | No |
| Ubuntu 22.04 (jammy) | 0.6.1 | No |
| Ubuntu 24.04 (noble) | 0.9.5 | No |
| Ubuntu 25.04 (plucky) | 0.9.5 | No |
| Ubuntu 25.10 (questing) | 0.10.4 | No |

Arch and Omarchy are fine - `pacman` tracks upstream closely. Homebrew is fine. On Debian and Ubuntu, use the official binary instead, which the next section walks through.

### You need a Nerd Font

LazyVim's interface uses icon glyphs that live outside normal fonts. Without a **Nerd Font (v3.0 or later)**, the UI fills with boxes, question marks, and blank rectangles. Everything still *works*, but it looks broken, and this is the single most common "did I install it wrong?" question from new LazyVim users.

You install a Nerd Font once, then set it as your terminal's font. It is a terminal setting, not a NeoVim setting.

1. Download one from <https://www.nerdfonts.com/font-downloads>. `JetBrainsMono Nerd Font`, `FiraCode Nerd Font`, and `Hack Nerd Font` are all good picks.
2. Install it the way you install any font:
   - **macOS:** open the `.ttf` files and click Install Font, or `brew install --cask font-jetbrains-mono-nerd-font`
   - **Linux:** copy the `.ttf` files into `~/.local/share/fonts/`, then run `fc-cache -fv`
   - **Windows:** select the `.ttf` files, right-click, Install
3. **Change your terminal's font setting to it.** This step is the one people skip. Installing the font is not enough; your terminal has to be told to use it.

Omarchy users: the default terminal is already configured with a Nerd Font, so there is nothing to do.

### Other tools LazyVim expects

None of these stop LazyVim from starting, but each one silently disables a feature until you install it.

| Tool | What breaks without it |
|------|------------------------|
| `git` (2.19+) | Plugin installation |
| `curl` | The completion engine |
| `ripgrep` | Project-wide text search (`<leader>/`) |
| `fd` | Fast file finding |
| `fzf` (0.25.1+) | Fuzzy pickers |
| A C compiler | Syntax highlighting via treesitter |
| `tree-sitter-cli` (0.26.1+) | The same thing: LazyVim builds treesitter parsers with it |
| `lazygit` | The built-in git interface (optional) |

Install them all in one go:

```bash
# macOS
brew install git curl ripgrep fd fzf lazygit tree-sitter

# Ubuntu/Debian
sudo apt-get update
sudo apt-get install git curl ripgrep fd-find fzf build-essential

# Arch/Omarchy
sudo pacman -S git curl ripgrep fd fzf lazygit base-devel tree-sitter-cli
```

On Debian and Ubuntu the `fd` command is installed as `fdfind`. If you want the usual name:

```bash
mkdir -p ~/.local/bin && ln -s "$(which fdfind)" ~/.local/bin/fd
```

`tree-sitter-cli` is missing from that `apt-get` line on purpose. LazyVim 16 tracks nvim-treesitter's `main` branch, which needs `tree-sitter-cli` 0.26.1 or newer, and every current Debian and Ubuntu release packages something older: 0.20.8 on Ubuntu 24.04 and 25.04, 0.22.6 on Ubuntu 25.10 and on Debian 13, 0.25.9 on Ubuntu 26.04. Upstream also rules out installing it from npm. Take the official binary instead:

```bash
curl -LO https://github.com/tree-sitter/tree-sitter/releases/latest/download/tree-sitter-linux-x64.gz
gunzip tree-sitter-linux-x64.gz
chmod +x tree-sitter-linux-x64
sudo mv tree-sitter-linux-x64 /usr/local/bin/tree-sitter
```

On ARM machines, replace `x64` with `arm64`. Then confirm it with `tree-sitter --version`. Without it, LazyVim still starts, but parser installation fails and syntax highlighting stays plain.

You also want a terminal that supports true colour and undercurl: kitty, WezTerm, Alacritty, Ghostty, or iTerm2 all qualify.

---

## Installing NeoVim

Whichever route you take, finish by running `nvim --version` and confirming you are on 0.11.2 or newer.

### macOS

```bash
brew install neovim
```

Homebrew tracks NeoVim closely, so this gives you a current version.

### Ubuntu / Debian

**Do not use `apt-get install neovim`.** As the table above shows, every current release ships a version LazyVim cannot use. Install the official binary instead:

```bash
curl -LO https://github.com/neovim/neovim/releases/latest/download/nvim-linux-x86_64.tar.gz
sudo rm -rf /opt/nvim-linux-x86_64
sudo tar -C /opt -xzf nvim-linux-x86_64.tar.gz
```

Then add it to your `PATH` by appending this line to `~/.bashrc` (or `~/.zshrc` if you use zsh):

```bash
export PATH="$PATH:/opt/nvim-linux-x86_64/bin"
```

Open a new terminal and check:

```bash
nvim --version
```

On ARM machines, replace `x86_64` with `arm64` in the download URL and the paths.

To update later, run the same three commands again with a fresh download.

There is also a community PPA (`ppa:neovim-ppa/unstable`) that carries current builds. It works, but it is called "unstable" for a reason and it is not maintained by the NeoVim project. The tarball above is the route the official docs recommend, and it is easy to reverse - delete `/opt/nvim-linux-x86_64` and remove the `PATH` line.

### Arch / Omarchy

```bash
sudo pacman -S neovim
```

Arch's `extra` repository tracks upstream closely, so this gives you a current version.

---

## Installing LazyVim

LazyVim is not a program you install. It is a **starter configuration** you copy into place, which NeoVim then loads on startup.

### 1. Back up any existing NeoVim setup

NeoVim keeps files in four separate places, and leaving old ones behind causes confusing errors that look like LazyVim bugs. Move all four:

```bash
# required
mv ~/.config/nvim{,.bak}

# optional but strongly recommended
mv ~/.local/share/nvim{,.bak}
mv ~/.local/state/nvim{,.bak}
mv ~/.cache/nvim{,.bak}
```

`mv ~/.config/nvim{,.bak}` is shell shorthand for `mv ~/.config/nvim ~/.config/nvim.bak`. If a directory does not exist, `mv` will say so and you can safely ignore it.

Nothing is deleted here. To undo the whole installation later, delete the new directories and move the `.bak` ones back.

### 2. Copy the starter configuration

```bash
git clone https://github.com/LazyVim/starter ~/.config/nvim
```

### 3. Remove the `.git` folder

```bash
rm -rf ~/.config/nvim/.git
```

**Do this.** The starter is a template you are meant to own and edit, not an upstream repository you track. If you keep the `.git` folder and later run `git pull` in there, you are pulling template changes on top of your own configuration, which fights your edits and can leave you with merge conflicts inside your editor config.

LazyVim itself is a plugin, and plugins update through `:Lazy update` from inside NeoVim - never by pulling the starter repo. See [Keeping LazyVim Updated](#keeping-lazyvim-updated).

Removing `.git` also means you can put your config in *your own* repository later, which is what most people eventually want.

### 4. Start NeoVim

```bash
nvim
```

The first launch downloads and installs every plugin. This takes a minute or two and you will see a progress window filling up. Let it finish.

### 5. Check your work

Inside NeoVim, type this and press Enter:

```
:LazyHealth
```

That loads every plugin and reports whether each one is happy. LazyVim recommends running it right after installation. For NeoVim's own diagnostics, use:

```
:checkhealth
```

Both produce a long report. You are looking for red `ERROR` lines. Warnings about optional providers (Python, Node, Ruby, Perl) are normal and harmless unless you specifically want those features.

If you see boxes or question marks instead of icons, your terminal font is not a Nerd Font. Go back to [Before You Install](#before-you-install).

---

---

## Your First Time Opening NeoVim

When you open NeoVim for the first time, you'll see a welcome screen or an empty editor. Here's what to know:

- Press `i` to enter **Insert Mode** and start typing
- Press `Escape` to go back to **Normal Mode**
- Type `:q` and press `Enter` to quit (if you haven't made changes)
- Type `:q!` and press `Enter` to quit without saving

That's it! You can already create files and edit them.

---

## Understanding Modes

NeoVim has different modes. This is key to understanding how it works:

### Normal Mode
- The default mode when you open NeoVim
- Used for navigation and commands
- **You can't type regular text here** - keys trigger actions instead
- Press `Escape` to always return to Normal Mode

### Insert Mode
- Where you actually type text
- Enter by pressing `i`, `a`, `o`, or other insert commands
- Everything you type appears in the file
- Press `Escape` to return to Normal Mode

### Visual Mode
- Used for selecting text
- Enter by pressing `v` in Normal Mode
- Move the cursor to select text
- Once selected, you can delete, copy, or modify the selection
- Press `Escape` to return to Normal Mode

### Visual Line Mode
- Selects whole lines at a time
- Enter by pressing `V` (capital V) in Normal Mode
- Move up and down with `j` and `k` to grow the selection
- This is the one you want most of the time - selecting three whole lines to delete or indent is a far more common job than selecting exactly seven characters
- Press `Escape` to return to Normal Mode

### Visual Block Mode
- Selects a rectangle, not lines
- Enter by pressing `Ctrl+v` in Normal Mode
- Move the cursor to grow the block in any direction
- Its signature trick: select a column, press `I`, type, then press `Escape`, and your text is inserted on **every** selected line at once. Excellent for commenting out a block or adding a prefix to a list.
- Press `Escape` to return to Normal Mode

**Windows users:** `Ctrl+v` is usually "paste" in your terminal, so it may never reach NeoVim. `Ctrl+q` does the same thing.

### Command Mode
- Used to run commands
- Enter by pressing `:` in Normal Mode
- Type the command and press `Enter`
- Press `Escape` to cancel

**Tip:** Always think about which mode you're in. If things seem weird, press `Escape` a few times to get back to Normal Mode.

---

## Navigation

Navigation is the foundation of NeoVim. Unlike most editors, you use keys instead of arrow keys for efficiency.

### The Four Direction Keys (Normal Mode)

| Key | Direction |
|-----|-----------|
| `h` | Left |
| `j` | Down |
| `k` | Up |
| `l` | Right |

**Yes, you can use arrow keys**, but the hjkl keys are standard in NeoVim and you'll see them everywhere. Try to get used to them.

### Faster Navigation

| Command | Action |
|---------|--------|
| `w` | Jump to the start of the next word |
| `b` | Jump to the start of the previous word |
| `e` | Jump to the end of the current word |
| `0` | Jump to the start of the line |
| `$` | Jump to the end of the line |
| `G` | Jump to the end of the file |
| `gg` | Jump to the start of the file |
| `{line_number}G` | Jump to a specific line (e.g., `42G` goes to line 42) |
| `Ctrl+f` | Page down |
| `Ctrl+b` | Page up |
| `Ctrl+d` | Half a page down |
| `Ctrl+u` | Half a page up |

> **⚠️ If you use Herdr (or tmux):** `Ctrl+b` is also the default *prefix key* in Herdr and tmux. Inside one of their panes, the multiplexer grabs `Ctrl+b` before NeoVim ever sees it, so page-up silently stops working. Use `Ctrl+u` instead - most Vim users prefer it anyway - or change the multiplexer's prefix. The [Herdr guide](herdr-guide.md#the-prefix-key) covers how.

### Search Navigation

| Command | Action |
|---------|--------|
| `/search_term` | Find the next occurrence of search_term |
| `n` | Go to next match |
| `N` | Go to previous match |

**Example:** Type `/function` to find the next occurrence of "function", then press `n` to go to the next one.

---

## Basic Editing

### Starting Insert Mode

| Command | Action |
|---------|--------|
| `i` | Insert before the cursor |
| `a` | Append after the cursor |
| `I` | Insert at the start of the line |
| `A` | Append at the end of the line |
| `o` | Create a new line below and insert |
| `O` | Create a new line above and insert |

**Most common:** Use `i` to start typing, then press `Escape` when done.

### Saving and Quitting

| Command | Action |
|---------|--------|
| `:w` | Save (write) the file |
| `:q` | Quit NeoVim |
| `:wq` or `ZZ` | Save and quit |
| `:q!` | Quit without saving |
| `:wqa` | Save and quit all files |

**Tip:** Type `:w` and press `Enter` frequently while editing. You won't lose work this way.

---

## Common Tasks

### Copy (Yank)

In Normal Mode:

| Command | Action |
|---------|--------|
| `yy` | Copy the current line |
| `y$` | Copy from cursor to end of line |
| `y0` | Copy from start of line to cursor |
| `yw` | Copy from the cursor to the start of the next word |
| `yiw` | Copy the whole word the cursor is in |

**With visual selection:**
1. Press `v` to enter Visual Mode
2. Move cursor to select text
3. Press `y` to copy

### Paste (Put)

In Normal Mode:

| Command | Action |
|---------|--------|
| `p` | Paste after the cursor |
| `P` | Paste before the cursor |

### Delete

In Normal Mode:

| Command | Action |
|---------|--------|
| `dd` | Delete the current line |
| `d$` | Delete from cursor to end of line |
| `d0` | Delete from start of line to cursor |
| `dw` | Delete from the cursor to the start of the next word |
| `diw` | Delete the whole word the cursor is in |
| `x` | Delete the character under the cursor |
| `X` | Delete the character before the cursor |

**With visual selection:**
1. Press `v` to enter Visual Mode
2. Move cursor to select text
3. Press `d` to delete

### Undo and Redo

| Command | Action |
|---------|--------|
| `u` | Undo the last change |
| `Ctrl+r` | Redo the last undo |

You can press `u` multiple times to undo multiple changes.

### Find and Replace

| Command | Action |
|---------|--------|
| `:s/old/new` | Replace first occurrence on current line |
| `:s/old/new/g` | Replace all occurrences on current line |
| `:%s/old/new/g` | Replace all occurrences in the entire file |
| `:%s/old/new/gc` | Replace all with confirmation for each |

**Example:** `:%s/function/func/g` replaces all instances of "function" with "func".

### Indentation

| Command | Action |
|---------|--------|
| `>>` | Indent the current line (add spaces) |
| `<<` | Unindent the current line (remove spaces) |
| `>}` | Indent from the cursor line to the end of the paragraph |
| `<}` | Unindent from the cursor line to the end of the paragraph |

**With visual selection:**
1. Press `v` to select text
2. Press `>` to indent or `<` to unindent
3. Repeat by pressing `.`

---

## LazyVim Essentials

Everything above this point is plain NeoVim and works in any Vim. This section is the part LazyVim adds on top.

### The leader key

LazyVim builds almost all of its commands on a **leader key**, and in LazyVim the leader is the **Space bar**.

`<leader>ff` in the documentation means: press Space, release, press `f`, press `f`. Three separate taps, not a chord.

**The single most useful thing to know about LazyVim:** press Space and then wait. A menu appears listing every command that starts with Space, grouped by category. You do not have to memorize anything - you can browse. The same works after `g`, `z`, `]`, and `[`.

### The commands worth knowing on day one

| Keys | What it does |
|------|--------------|
| `<leader>ff` | Find files in the project by name |
| `<leader><space>` | The same file finder, fewer keys |
| `<leader>/` | Search the *text* of every file in the project |
| `<leader>e` | Toggle the file explorer sidebar |
| `<leader>,` | Switch between files you already have open |
| `<leader>bd` | Close the current file |
| `Shift+L` / `Shift+H` | Next / previous open file |
| `<leader>qq` | Quit everything |
| `<leader>l` | Open the plugin manager |
| `<leader>sk` | Search every keybinding you have |
| `<leader>?` | Show the keybindings for this file type |

### Moving between splits

LazyVim binds these directly, so you can skip the `Ctrl+w` prefix that plain Vim needs:

| Keys | What it does |
|------|--------------|
| `Ctrl+h` / `Ctrl+j` / `Ctrl+k` / `Ctrl+l` | Move to the split left / below / above / right |
| `<leader>-` | Split the window below |
| `<leader>\|` | Split the window to the right |
| `<leader>wd` | Close the current split |

### Code navigation

These need a language server, which LazyVim installs automatically the first time you open a file in a supported language.

| Keys | What it does |
|------|--------------|
| `gd` | Go to where this thing is defined |
| `gr` | Find everywhere it is used |
| `K` | Show documentation for what is under the cursor |
| `<leader>cr` | Rename it everywhere |
| `<leader>ca` | Offer available fixes ("code actions") |
| `<leader>cf` | Format the file |
| `]d` / `[d` | Jump to the next / previous problem |

### Saving

`Ctrl+s` saves, in Normal *and* Insert mode. It is the one concession LazyVim makes to muscle memory from other editors, and it is a good one.

---

## Settings and Configuration

### LazyVim Configuration Files

LazyVim stores settings in `~/.config/nvim/`. The main files are:

- `init.lua` - Main entry point
- `lua/config/options.lua` - Editor settings
- `lua/config/keymaps.lua` - Custom keybindings

### Common Settings You Might Want to Change

Open the settings file:
```bash
nvim ~/.config/nvim/lua/config/options.lua
```

Common options to modify:

```lua
-- Line numbers
vim.opt.number = true              -- Show line numbers
vim.opt.relativenumber = true      -- Show relative line numbers

-- Indentation
vim.opt.tabstop = 2                -- Tab width (spaces)
vim.opt.shiftwidth = 2             -- Indent width
vim.opt.expandtab = true           -- Use spaces instead of tabs

-- Display
vim.opt.wrap = true                -- Wrap long lines
vim.opt.cursorline = true          -- Highlight current line
vim.opt.colorcolumn = "80"         -- Show a line at column 80

-- Behavior
vim.opt.ignorecase = true          -- Case-insensitive search
vim.opt.smartcase = true           -- Unless you use uppercase
vim.opt.mouse = "a"                -- Enable mouse support
```

### How to Change Settings

1. Open the settings file: `nvim ~/.config/nvim/lua/config/options.lua`
2. Find the setting you want to change (or add a new line)
3. Change `true` to `false` or vice versa
4. Save: Press `Escape`, then type `:w`, then press `Enter`
5. The changes take effect next time you open NeoVim (or run `:source %` to reload now)

### Adding Custom Keybindings

Edit `~/.config/nvim/lua/config/keymaps.lua`:

```lua
-- Format: vim.keymap.set(mode, keys, action, options)

-- Save every open file with <leader>W, from Normal and Visual mode
vim.keymap.set({ "n", "v" }, "<leader>W", "<cmd>wall<cr>", { desc = "Save all files" })

-- Clear search highlighting with <leader>h
vim.keymap.set("n", "<leader>h", "<cmd>nohlsearch<cr>", { desc = "Clear highlights" })
```

- `"n"` = Normal Mode
- `"i"` = Insert Mode
- `"v"` = Visual Mode
- `"<leader>"` = Space key (by default)
- `desc` is the label LazyVim shows in the Space menu, so always set it

**Do not guess at plugin commands.** A mapping that calls a command a plugin does not provide fails silently until you press the key, and then reports an error you have to decode. To find the real command, run `<leader>sC` to search the commands that actually exist, or `<leader>sk` to see what an existing key is already bound to and copy that.

**Before you add a mapping, check whether LazyVim already has one.** It ships several hundred. `<leader>sk` searches them all.

---

## File Operations

### Opening Files

| Command | Action |
|---------|--------|
| `nvim filename` | Open a file from the terminal |
| `:e filename` | Open a file within NeoVim |
| `<leader>e` | Open the file explorer sidebar (LazyVim) |
| `<leader>ff` | Fuzzy-find files in the project (LazyVim) |
| `<leader><space>` | The same file finder, on a faster key (LazyVim) |

`<leader>` is the Space bar in LazyVim. So `<leader>ff` means: press Space, then `f`, then `f`.

Plain Vim has a bare-bones directory browser you reach with `:e .`, and older tutorials point you at it. In LazyVim you want `<leader>e` instead: it opens LazyVim's own file explorer in a sidebar, which is better in every way and is the one the rest of LazyVim's keybindings expect.

### Creating New Files

1. Open NeoVim: `nvim`
2. Type your content
3. Save with `:w filename` to give it a name

### Working with Multiple Files

| Command | Action |
|---------|--------|
| `:split filename` | Open file in a split window |
| `:vsplit filename` | Open file in a vertical split |
| `:tabnew filename` | Open file in a new tab |
| `:bnext` | Switch to the next open file (buffer) |
| `:bprev` | Switch to the previous open file (buffer) |
| `:ls` | List everything you have open |

In LazyVim, `Shift+L` and `Shift+H` do `:bnext` and `:bprev` without typing a command, and `<leader>,` opens a searchable list of open files.

**A note on the word "buffer":** every file you open becomes a *buffer* - NeoVim's in-memory copy of it. A *window* is a viewport showing one buffer, and closing a window does not close the buffer. This is why `:bnext` is the command you want and not `:next`; `:next` walks the list of files you named on the command line when you launched NeoVim, which is almost never what a beginner means.

---

## Search and Replace

### Basic Search

Press `/` in Normal Mode:
```
/search_term
```

Then press `Enter`. NeoVim will highlight matches.

- Press `n` to go to next match
- Press `N` to go to previous match
- Press `Escape` to exit search

### Find and Replace

```
:%s/old_text/new_text/g
```

- `%` - Apply to whole file
- `s` - Substitute command
- `/old_text/new_text/` - What to replace and what to replace it with
- `g` - Global (all occurrences on each line)

Add `c` to confirm each replacement:
```
:%s/old_text/new_text/gc
```

---

## Windows and Splits

### Creating Splits

| Command | Action |
|---------|--------|
| `:split` | Horizontal split |
| `:vsplit` | Vertical split |
| `Ctrl+w s` | Horizontal split |
| `Ctrl+w v` | Vertical split |

### Navigating Between Splits

| Command | Action |
|---------|--------|
| `Ctrl+w h` | Move to left split |
| `Ctrl+w j` | Move to split below |
| `Ctrl+w k` | Move to split above |
| `Ctrl+w l` | Move to right split |
| `Ctrl+w w` | Cycle through splits |

### Resizing Splits

| Command | Action |
|---------|--------|
| `Ctrl+w +` | Increase split height |
| `Ctrl+w -` | Decrease split height |
| `Ctrl+w >` | Increase split width |
| `Ctrl+w <` | Decrease split width |

### Closing Splits

| Command | Action |
|---------|--------|
| `:close` | Close current split |
| `:only` | Close all other splits |

---

## Keeping LazyVim Updated

### The right way

Everything - LazyVim itself and every plugin it installed - updates from **inside NeoVim**, through the plugin manager:

```
:Lazy update
```

That is the whole answer. Run it every week or two.

Other useful plugin-manager commands:

| Command | What it does |
|---------|--------------|
| `:Lazy` | Open the plugin manager UI (`?` for help, `q` to close) |
| `:Lazy update` | Update LazyVim and all plugins |
| `:Lazy sync` | Install missing, update existing, remove leftovers |
| `:Lazy health` | Check the plugin manager itself |
| `:LazyHealth` | Load every plugin and check that each one is working |
| `:LazyExtras` | Browse and toggle LazyVim's optional language and tool packs |

`<leader>l` opens the same UI without typing anything.

### ⚠️ Never update by pulling the starter repository

You may see advice like `cd ~/.config/nvim && git pull origin main`. **Do not do this.**

The LazyVim starter is a **template**, not an upstream you track. `~/.config/nvim` is *your* configuration - your options, your keymaps, your plugin choices. Pulling template changes on top of it fights your own edits and can drop you into merge conflicts inside the very config your editor needs in order to start.

This is why [Installing LazyVim](#installing-lazyvim) tells you to `rm -rf ~/.config/nvim/.git`. With no git remote there is nothing to pull, and the trap closes itself.

### Updating NeoVim itself

NeoVim is separate from LazyVim and updates through however you installed it: `brew upgrade neovim`, `sudo pacman -Syu neovim`, or downloading a fresh tarball. After a NeoVim upgrade it is worth running `:LazyHealth` once.

### Version-locking, if you want it

`:Lazy update` writes a `lazy-lock.json` file in `~/.config/nvim` recording the exact version of every plugin. Keep that file in your own git repository and you can reproduce your setup on another machine with `:Lazy restore`, or roll back an update that broke something.

---

## Essential Keybindings Reference

### Quick Reference

| Action | Keys |
|--------|------|
| **Navigation** | |
| Move left/right/up/down | `h` `l` `j` `k` |
| Jump to start of next word | `w` |
| Jump to start of line | `0` |
| Jump to end of line | `$` |
| Jump to end of file | `G` |
| Jump to start of file | `gg` |
| **Editing** | |
| Insert before cursor | `i` |
| Append after cursor | `a` |
| New line below | `o` |
| Delete line | `dd` |
| Delete to start of next word | `dw` |
| Delete the whole word under the cursor | `diw` |
| Copy line | `yy` |
| Copy to start of next word | `yw` |
| Copy the whole word under the cursor | `yiw` |
| Paste after cursor | `p` |
| Paste before cursor | `P` |
| Undo | `u` |
| Redo | `Ctrl+r` |
| **Other** | |
| Save | `:w` |
| Quit | `:q` |
| Save and quit | `:wq` |
| Search | `/` |
| Replace | `:%s/old/new/g` |
| Select all | `gg V G` |

---

## Tips for Getting Better

1. **Don't try to learn everything at once.** Master navigation first (hjkl), then basic editing (i, dd, yy, p).

2. **Use the help system.** Type `:help topic` to learn about any command. For example, `:help :w` explains the save command.

3. **Practice the basics daily.** Opening files, navigating, editing, saving. Do this for a week before learning advanced features.

4. **Use LazyVim's features gradually.** You get fuzzy file search (`<leader>ff`), project-wide text search (`<leader>/`), a file explorer (`<leader>e`), and dozens of plugins for free. Explore them as you get comfortable.

   The fastest way to discover them: press Space and wait a moment. A menu of everything available appears.

5. **Keep this guide handy.** Bookmark it and refer back when you forget a command.

6. **Get comfortable with search and replace.** It's one of the most powerful features once you learn it.

7. **Don't use the mouse.** It might feel slower at first, but keyboard navigation is much faster once you're used to it.

---

## Troubleshooting

**"I can't type anything"**
- You're in Normal Mode. Press `i` to enter Insert Mode.

**"My text disappeared"**
- You probably pressed `d`. Press `u` to undo.

**"NeoVim won't quit"**
- Make sure you're in Normal Mode (press `Escape`), then type `:q!` to force quit.

**"My changes aren't saved"**
- Type `:w` to save. You'll see "written to filename" at the bottom.

**"Plugins won't install"**
- Open the plugin manager with `:Lazy` and look at what it says. Press `?` inside it for help, `q` to close.
- Run `:Lazy sync` to install anything missing, update the rest, and remove leftovers.
- Then run `:LazyHealth` and `:checkhealth` and read the red `ERROR` lines.
- Plugin installation needs `git` 2.19 or newer. Check with `git --version`.

**"LazyVim won't start, or throws errors immediately"**
- Almost always an out-of-date NeoVim. Run `nvim --version`; you need 0.11.2 or newer. See [Before You Install](#before-you-install).

**"My interface is full of boxes and question marks"**
- Your terminal is not using a Nerd Font. Install one and set it as your terminal's font. See [Before You Install](#before-you-install).

**"Ctrl+b doesn't page up any more"**
- You are inside Herdr or tmux, and it claimed `Ctrl+b` as its prefix key. Use `Ctrl+u` instead, or change the multiplexer's prefix.

**"Which key was that again?"**
- Press Space and wait. LazyVim pops up every command that starts with Space. Do the same after `g`, `z`, or `]`.
- `<leader>sk` opens a searchable list of every keybinding you have.

---

## Next Steps

- Practice basic navigation and editing for a week before adding anything new
- Learn search and replace properly
- Press Space and browse the menu until the commands you use most stop needing a lookup
- Read `:help` pages for topics that interest you
- Customize your keybindings and settings

**Official documentation:**

- LazyVim: <https://lazyvim.org>
- LazyVim keymaps, the authoritative list: <https://lazyvim.org/keymaps>
- NeoVim: <https://neovim.io/doc/>
- Inside the editor: `:help`, and `:help user-manual` for the guided tour

If you also use Herdr, see the [Herdr guide](herdr-guide.md) - and read its note about the `Ctrl+b` collision before it confuses you.
