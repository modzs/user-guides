# NeoVim and LazyVim Guide for Beginners

A complete guide to getting started with NeoVim and LazyVim, from installation to everyday workflows.

**Platform Support:** This guide covers macOS, Ubuntu/Debian Linux, and Arch/Omarchy Linux. The core features are identical across all platforms - only installation methods differ.

## Table of Contents
1. [What are NeoVim and LazyVim?](#what-are-neovim-and-lazyvim)
2. [Installation](#installation)
3. [Your First Time Opening NeoVim](#your-first-time-opening-neovim)
4. [Understanding Modes](#understanding-modes)
5. [Navigation](#navigation)
6. [Basic Editing](#basic-editing)
7. [Common Tasks](#common-tasks)
8. [Settings and Configuration](#settings-and-configuration)
9. [File Operations](#file-operations)
10. [Search and Replace](#search-and-replace)
11. [Windows and Splits](#windows-and-splits)
12. [Essential Keybindings Reference](#essential-keybindings-reference)

---

## What are NeoVim and LazyVim?

**NeoVim** is a modern text editor that's a fork and enhancement of Vim. It's powerful, fast, and works entirely from the keyboard - no need to reach for the mouse.

**LazyVim** is a starter configuration for NeoVim that comes pre-configured with useful plugins and settings. It makes NeoVim easier to use for beginners while being customizable for advanced users.

Think of it this way: NeoVim is the engine, LazyVim is the car with all the features already installed and set up nicely.

---

## Installation

### macOS

**Using Homebrew:**
```bash
brew install neovim
```

**Using Git and building from source:**
```bash
git clone https://github.com/neovim/neovim.git
cd neovim
make CMAKE_BUILD_TYPE=Release
sudo make install
```

### Linux - Ubuntu/Debian

**Using apt:**
```bash
sudo apt-get update
sudo apt-get install neovim
```

**Using Git and building from source:**
```bash
# Install build dependencies
sudo apt-get install build-essential cmake git

# Clone and build
git clone https://github.com/neovim/neovim.git
cd neovim
make CMAKE_BUILD_TYPE=Release
sudo make install
```

### Linux - Arch/Omarchy

**Using pacman:**
```bash
sudo pacman -S neovim
```

**Using Git and building from source:**
```bash
# Clone the repository
git clone https://github.com/neovim/neovim.git
cd neovim

# Build and install
make CMAKE_BUILD_TYPE=Release
sudo make install
```

### Setting up LazyVim

1. **Backup your existing NeoVim config (if you have one):**
   ```bash
   mv ~/.config/nvim ~/.config/nvim.bak
   ```

2. **Clone the LazyVim starter config using Git:**
   ```bash
   git clone https://github.com/LazyVim/starter ~/.config/nvim
   ```

3. **Option A - Remove the .git folder (recommended for beginners):**
   This makes it a regular configuration, not a git repository:
   ```bash
   rm -rf ~/.config/nvim/.git
   ```

4. **Option B - Keep it as a git repository (for updates):**
   If you want to receive LazyVim updates via git:
   ```bash
   cd ~/.config/nvim
   git pull origin main
   ```

5. **Open NeoVim:**
   ```bash
   nvim
   ```
   LazyVim will automatically install plugins on first launch (this may take a minute or two).

### Verifying Installation

After opening NeoVim for the first time, you should see:
- A welcome screen with LazyVim information
- Plugin installation happening in the background
- No errors in red text

If you see errors, try:
```bash
# Inside NeoVim
:checkhealth
```

This shows you any missing dependencies or configuration issues.

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
| `yw` | Copy the current word |

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
| `dw` | Delete the current word |
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
| `>}` | Indent the current paragraph |
| `<}` | Unindent the current paragraph |

**With visual selection:**
1. Press `v` to select text
2. Press `>` to indent or `<` to unindent
3. Repeat by pressing `.`

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
-- Example: Map <leader>e to open file explorer
local keymap = vim.keymap

-- Format: keymap.set(mode, keys, action, options)
keymap.set("n", "<leader>ff", "<cmd>Telescope find_files<cr>", { noremap = true })
```

- `"n"` = Normal Mode
- `"i"` = Insert Mode
- `"v"` = Visual Mode
- `"<leader>"` = Space key (by default)

---

## File Operations

### Opening Files

| Command | Action |
|---------|--------|
| `nvim filename` | Open a file from the terminal |
| `:e filename` | Open a file within NeoVim |
| `:e .` | Open the file browser |
| `Ctrl+p` | Quick file search (with LazyVim) |

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
| `:next` | Switch to next file |
| `:prev` | Switch to previous file |

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
| Delete word | `dw` |
| Copy line | `yy` |
| Copy word | `yw` |
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

4. **Use LazyVim's features gradually.** You get fuzzy search (`Ctrl+p`), file browser, and many plugins for free. Explore them as you get comfortable.

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
- Close NeoVim and run `nvim` again. LazyVim installs plugins on startup.

---

## Next Steps

- Practice basic navigation and editing for a week
- Learn search and replace
- Explore LazyVim's file browser and search features
- Read `:help` pages for topics that interest you
- Customize your keybindings and settings
