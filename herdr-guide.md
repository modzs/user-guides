# Herdr Guide for Beginners

A complete guide to getting started with Herdr, a terminal multiplexer for AI agent workflows.

**Platform Support:** This guide covers macOS, Ubuntu/Debian Linux Server, and Arch/Omarchy Linux. The core features are identical across all platforms - only installation methods differ.

## Table of Contents
1. [What is Herdr?](#what-is-herdr)
2. [Installation](#installation)
3. [Basic Concepts](#basic-concepts)
4. [Starting Herdr](#starting-herdr)
5. [Sessions](#sessions)
6. [Windows and Panes](#windows-and-panes)
7. [Working with AI Agents](#working-with-ai-agents)
8. [Navigation](#navigation)
9. [Common Operations](#common-operations)
10. [Configuration](#configuration)
11. [Workflow Patterns](#workflow-patterns)
12. [Essential Commands Reference](#essential-commands-reference)
13. [Troubleshooting](#troubleshooting)

---

## What is Herdr?

**Herdr** is a terminal multiplexer designed specifically for managing AI agent workflows. It lets you:

- Run multiple AI agents simultaneously
- Organize agents into sessions and windows
- Monitor agent output in real-time
- Switch between different agents and tasks easily
- Keep your workflow organized and efficient

Think of it as a command center for coordinating multiple AI agents. Instead of juggling terminal windows, Herdr keeps everything organized in one place.

### Why Use Herdr?

- **Parallel Agent Execution**: Run multiple AI agents at the same time without losing track of any
- **Better Organization**: Group related agents and tasks logically
- **Easy Monitoring**: See all agent outputs at once or focus on one
- **Persistent Sessions**: Your work stays organized even if you disconnect
- **Workflow Management**: Build complex multi-agent workflows efficiently

---

## Installation

### macOS

**Using Homebrew (easiest):**
```bash
brew install herdr
```

**Using Cargo (Rust package manager):**
```bash
cargo install herdr
```

**Building from Git:**
```bash
git clone https://github.com/herdr-dev/herdr.git
cd herdr
cargo build --release
sudo mv target/release/herdr /usr/local/bin/
```

### Linux - Ubuntu/Debian Server

**Using apt (if available in your repositories):**
```bash
sudo apt-get update
sudo apt-get install herdr
```

**Using Cargo (Rust package manager - works on all Linux systems):**
```bash
# First install Rust if you don't have it
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
source $HOME/.cargo/env

# Then install Herdr
cargo install herdr
```

**Building from Git:**
```bash
# Install git if needed
sudo apt-get install git

# Clone and build
git clone https://github.com/herdr-dev/herdr.git
cd herdr
cargo build --release
sudo mv target/release/herdr /usr/local/bin/
```

### Linux - Arch/Omarchy

**Using pacman:**
```bash
sudo pacman -S herdr
```

**Using AUR helper (yay or paru):**
```bash
yay -S herdr
# or
paru -S herdr
```

**Using Cargo:**
```bash
cargo install herdr
```

**Building from Git:**
```bash
# Install git and build tools if needed
sudo pacman -S git base-devel

# Clone and build
git clone https://github.com/herdr-dev/herdr.git
cd herdr
cargo build --release
sudo mv target/release/herdr /usr/local/bin/
```

### Verify Installation

After installation, verify it works on any system:

```bash
herdr --version
```

You should see the version number printed. If not, troubleshoot:

**On macOS:**
```bash
which herdr
echo $PATH
```

**On Linux:**
```bash
which herdr
echo $PATH
# May need to log out and back in for PATH to update
```

If `herdr` is not found, make sure the installation directory is in your PATH.

---

## Basic Concepts

Before using Herdr, understand these key concepts:

### Sessions
- The top-level container for your work
- A single session can contain multiple windows
- You can have multiple sessions running at once
- Think of sessions as different "projects"

**Example:** You might have one session for "ML Training" and another for "API Development"

### Windows
- Subdivisions within a session
- Each window can contain multiple panes
- Windows are like tabs within a project
- You can have multiple windows per session

**Example:** Within the "ML Training" session, you might have a window for "Data Processing" and another for "Model Training"

### Panes
- The actual terminal areas where agents run or output appears
- Panes are subdivisions of a window
- You can split windows into multiple panes
- Each pane can show different agent output

**Example:** In the "Model Training" window, you might have one pane showing the training agent output and another showing monitoring data

### Agents
- The AI programs that actually do work
- Run within panes
- Send output that you can view in Herdr
- Can be simple commands or complex workflows

---

## Starting Herdr

### First Time Setup

Open a terminal and type:
```bash
herdr
```

This creates a new session with a default name and opens it. You'll see a terminal prompt where you can start agents.

### Creating a Named Session

```bash
herdr new-session my-project
```

This creates a session called "my-project" instead of using a default name.

### Listing Sessions

```bash
herdr list-sessions
```

Shows all currently running sessions and which one is active.

### Attaching to a Session

If you have a session that's already running:
```bash
herdr attach my-project
```

This connects you to the "my-project" session.

### Detaching from a Session

Press `Ctrl+b` then `d` (you can do this without releasing Ctrl for the first one).

The session keeps running in the background even after you detach.

---

## Sessions

### Creating Sessions

**From the command line:**
```bash
herdr new-session workflow-1
```

**Inside Herdr:**
1. Press `Ctrl+b` then `c` to create a new window (if this is what you want)
2. Or use the command: `:new-session workflow-2`

### Renaming a Session

```bash
herdr rename-session old-name new-name
```

Or inside Herdr:
```
:rename-session new-name
```

### Switching Between Sessions

If you have multiple sessions:
```bash
herdr select-session session-name
```

Or use the keyboard shortcut (after pressing `Ctrl+b`):
- Type the session name
- Or use `Ctrl+b` then `s` for an interactive session list

### Listing Session Details

```bash
herdr list-sessions -details
```

Shows which windows and panes are in each session.

### Killing a Session

```bash
herdr kill-session session-name
```

This stops the session and closes all its windows and panes. **Warning:** This will stop any running agents.

---

## Windows and Panes

### Understanding the Layout

A typical Herdr layout looks like this:

```
┌─────────────────────────────────────┐
│ Herdr: my-session                   │
├─────────────────────────────────────┤
│ Window 1: data-processing           │
│ ┌──────────────────┬────────────────┤
│ │ Pane 1           │ Pane 2         │
│ │ Agent 1 output   │ Agent 2 output │
│ ├──────────────────┼────────────────┤
│ │ Pane 3           │ Pane 4         │
│ │ Agent 3 output   │ Monitoring     │
│ └──────────────────┴────────────────┤
└─────────────────────────────────────┘
```

### Creating Windows

**From the command line:**
```bash
herdr new-window -t my-session -n my-window
```

**Inside a session (press `Ctrl+b` then `c`):**
This creates a new window in the current session.

**With a name:**
```bash
herdr new-window -t my-session -n agent-monitoring
```

### Splitting Panes

**Horizontal split** (one on top, one on bottom):
Press `Ctrl+b` then `"` (or `Shift+'`)

**Vertical split** (one on left, one on right):
Press `Ctrl+b` then `%`

### Navigating Panes

**Move to the next pane:**
Press `Ctrl+b` then `o`

**Move in a specific direction:**
- `Ctrl+b` then `←` - Move to pane on the left
- `Ctrl+b` then `→` - Move to pane on the right
- `Ctrl+b` then `↑` - Move to pane above
- `Ctrl+b` then `↓` - Move to pane below

**Cycle through panes:**
Press `Ctrl+b` then `o` repeatedly

### Resizing Panes

**Increase pane size:**
- `Ctrl+b` then `Alt+↑` - Increase height
- `Ctrl+b` then `Alt+↓` - Decrease height
- `Ctrl+b` then `Alt+→` - Increase width
- `Ctrl+b` then `Alt+←` - Decrease width

**Or using mouse:**
Move your cursor to the pane border and drag.

### Closing Panes

Press `Ctrl+b` then `x` to close the current pane.

Type `exit` and press `Enter` - this stops the current agent and closes the pane.

### Renaming Windows

Press `Ctrl+b` then `,` and type the new name.

Or from the command line:
```bash
herdr rename-window -t my-session:old-name new-name
```

---

## Working with AI Agents

### Starting an Agent in a Pane

In any pane, simply run the agent command:
```bash
claude code --task "analyze this codebase"
```

Or any other AI agent command specific to your workflow.

### Monitoring Agent Output

- The pane automatically shows the agent's output
- You can scroll up and down to see previous output
- Press `Space` or `Page Down` to scroll forward
- Press `Page Up` to scroll backward
- Press `Ctrl+b` then `]` to enter copy mode for selecting text

### Running Multiple Agents in Parallel

Create multiple panes in the same window and start different agents in each:

1. Create a window: `Ctrl+b` then `c`
2. Split horizontally: `Ctrl+b` then `"` 
3. In the top pane, start Agent 1
4. Move to bottom pane: `Ctrl+b` then `↓`
5. Start Agent 2

Now both agents run in parallel and you see both outputs.

### Coordinating Agent Workflows

**Sequential workflow** (one agent waits for another):
1. In pane 1: Start Agent A and wait for it to complete
2. Copy the output (manually or with copy mode)
3. In pane 2: Use the output from Agent A as input to Agent B

**Parallel workflow** (multiple agents run simultaneously):
1. Split the window into multiple panes
2. Start different agents in each pane
3. Monitor all outputs at once
4. Combine results once agents complete

---

## Navigation

### Switch Between Windows

**Next window:**
Press `Ctrl+b` then `n`

**Previous window:**
Press `Ctrl+b` then `p`

**Direct selection:**
Press `Ctrl+b` then the window number (0, 1, 2, etc.)

**List all windows:**
Press `Ctrl+b` then `w` for an interactive list

### Switch Between Sessions

**Interactive session selector:**
Press `Ctrl+b` then `s`

**From the command line:**
```bash
herdr select-session my-session
```

### Jump to a Specific Pane

Press `Ctrl+b` then the pane number shown in the status bar.

Or use arrow keys after pressing `Ctrl+b`.

### Searching Within a Pane

**Enter search mode:**
Press `Ctrl+b` then `/`

**Type your search term**, then press `Enter`.

**Navigate matches:**
- Press `n` for next match
- Press `N` for previous match
- Press `Escape` to exit search

---

## Common Operations

### Viewing Agent Status

In any pane, you can check what's running:
```bash
ps aux | grep agent
```

Or use Herdr's built-in status:
```bash
herdr info
```

### Copying Agent Output

**Enter copy mode:**
Press `Ctrl+b` then `[`

**Select text:**
- Use arrow keys or vim keys (hjkl) to move
- Hold `Shift` and use arrows to select (or use space to start selection)

**Copy selected text:**
Press `Enter` or `y`

**Exit copy mode:**
Press `Escape`

### Viewing Historical Output

Each pane keeps a scrollback buffer of previous output.

**Scroll up:**
Press `Ctrl+b` then `[` to enter copy mode, then scroll with arrow keys.

**Scroll down:**
Exit copy mode with `Escape`.

### Clearing a Pane

In a pane, type:
```bash
clear
```

Or press `Ctrl+l` (this is a terminal feature, not Herdr-specific).

### Sending Commands to a Pane

You can send text directly from another pane:
```bash
herdr send-keys -t my-session:my-window.1 "command to run" Enter
```

This sends the command to the specified pane.

### Synchronizing Panes

To run the same command in multiple panes at once:

**Enable synchronized panes:**
Press `Ctrl+b` then `:` and type `set synchronize-panes on`

**Type commands** - they'll go to all panes

**Disable when done:**
Press `Ctrl+b` then `:` and type `set synchronize-panes off`

---

## Configuration

### Configuration File Location

Herdr looks for configuration in:
- `~/.config/herdr/config.toml`
- `~/.herdr/config`

Create one of these files to customize Herdr.

### Common Configuration Options

Create `~/.config/herdr/config.toml`:

```toml
# Default terminal
terminal = "zsh"

# Prefix key (default is Ctrl+b)
prefix = "Ctrl+b"

# Mouse support
mouse = true

# Scrollback buffer size
scrollback = 10000

# Default session name
default_session = "main"

# Colors
theme = "dark"

# Status bar
status_bar = true

# Window name prefix
window_prefix = "@"
```

### Custom Keybindings

Add to your config file:

```toml
[keybindings]
# Format: action = "key_combination"
# Examples:
new_window = "Ctrl+b c"
new_session = "Ctrl+b n"
kill_pane = "Ctrl+b x"
next_window = "Ctrl+b n"
```

### Reloading Configuration

After changing config, reload without restarting:
```bash
herdr reload-config
```

Or restart Herdr by detaching and re-attaching.

---

## Workflow Patterns

### Pattern 1: Sequential Agent Workflow

Useful for tasks that depend on each other.

```
Session: document-analysis
├── Window 1: input-processing
│   ├── Pane 1: Run Agent A (reads input)
├── Window 2: analysis
│   ├── Pane 1: Run Agent B (analyzes output from A)
├── Window 3: output
│   ├── Pane 1: Run Agent C (formats results)
```

**How to set up:**
1. Create session: `herdr new-session document-analysis`
2. Create three windows for each stage
3. In each window, start the appropriate agent
4. Monitor outputs and pass data between agents manually

### Pattern 2: Parallel Agent Workflow

Useful when agents can work independently.

```
Session: parallel-processing
├── Window 1: multi-agent
│   ├── Pane 1: Agent 1 (works on dataset 1)
│   ├── Pane 2: Agent 2 (works on dataset 2)
│   ├── Pane 3: Agent 3 (works on dataset 3)
│   └── Pane 4: Monitoring pane
```

**How to set up:**
1. Create session: `herdr new-session parallel-processing`
2. Split window into 4 panes
3. Start agents in panes 1-3
4. Use pane 4 for monitoring all agents
5. Combine results once all complete

### Pattern 3: Development + Testing Workflow

Useful for iterative development with AI assistance.

```
Session: ml-development
├── Window 1: development
│   ├── Pane 1: Code editor (NeoVim)
│   ├── Pane 2: AI Agent (provides suggestions)
├── Window 2: testing
│   ├── Pane 1: Test runner
│   ├── Pane 2: Test monitoring
```

**How to set up:**
1. Create session: `herdr new-session ml-development`
2. Split first window into 2 panes
3. In pane 1, open your editor
4. In pane 2, start your AI assistant
5. Create second window for tests
6. Run tests and monitor results

### Pattern 4: Multi-Project Workflow

Useful when working on multiple projects.

```
Session 1: project-alpha
├── Windows for different agents/tasks

Session 2: project-beta
├── Windows for different agents/tasks

Session 3: shared-utilities
├── Monitoring agents
├── Shared resources
```

**How to set up:**
1. Create multiple sessions: `herdr new-session project-alpha`
2. Create sessions for each project
3. In each session, organize windows by task
4. Switch between sessions as needed: `herdr select-session project-name`

---

## Essential Commands Reference

### Session Commands

| Command | Action |
|---------|--------|
| `herdr` | Start new default session |
| `herdr new-session name` | Create named session |
| `herdr attach name` | Connect to session |
| `herdr detach` | Disconnect (Ctrl+b d) |
| `herdr kill-session name` | Stop session |
| `herdr list-sessions` | Show all sessions |
| `herdr rename-session old new` | Rename session |

### Window Commands

| Shortcut | Action |
|----------|--------|
| `Ctrl+b c` | New window |
| `Ctrl+b n` | Next window |
| `Ctrl+b p` | Previous window |
| `Ctrl+b 0-9` | Jump to window number |
| `Ctrl+b w` | Window list |
| `Ctrl+b ,` | Rename window |
| `Ctrl+b &` | Kill window |

### Pane Commands

| Shortcut | Action |
|----------|--------|
| `Ctrl+b "` | Horizontal split |
| `Ctrl+b %` | Vertical split |
| `Ctrl+b o` | Next pane |
| `Ctrl+b ↑↓←→` | Move to pane |
| `Ctrl+b x` | Close pane |
| `Ctrl+b z` | Zoom pane (toggle full screen) |
| `Ctrl+b Ctrl+↑↓←→` | Resize pane |

### Copy/Scrollback Commands

| Shortcut | Action |
|----------|--------|
| `Ctrl+b [` | Enter copy mode |
| `Space` or `Enter` | Start selection |
| `Enter` | Copy selection |
| `Escape` | Exit copy mode |
| `/` | Search in copy mode |

### Miscellaneous

| Command | Action |
|---------|--------|
| `Ctrl+b :` | Enter command mode |
| `Ctrl+b ?` | Show keybindings |
| `Ctrl+b d` | Detach session |
| `Ctrl+b t` | Show time |

---

## Troubleshooting

### "Session not found"

Herdr couldn't find the session you're trying to attach to.

**Solution:**
1. List sessions: `herdr list-sessions`
2. Make sure you're using the correct name
3. Create a new session if needed: `herdr new-session my-session`

### "Can't split pane"

You've run out of space or the pane is too small.

**Solution:**
1. Close some panes to free space
2. Make the window larger
3. Use a different layout

### "Agent output is not showing"

The agent might have exited or there's an issue with the pane.

**Solution:**
1. Check if the agent is still running: `ps aux | grep agent`
2. In the pane, press `Enter` to see if it's stuck
3. Try running the agent again: `command to run`
4. If the pane is frozen, kill it and create a new one: `Ctrl+b x`

### "Can't remember the keybindings"

This is normal - there are many keybindings.

**Solution:**
1. Press `Ctrl+b ?` to see all keybindings
2. Keep this guide open for reference
3. Create custom keybindings for your most-used commands
4. Practice the most common ones (new window, split, navigate)

### "Herdr won't start"

The installation might be incomplete or corrupted.

**Solution:**
```bash
# Reinstall
brew reinstall herdr

# Or from source
cargo install --force herdr

# Verify installation
herdr --version
```

### "Session keeps disconnecting"

Your network connection might be dropping.

**Solution:**
1. Use detach and re-attach instead of closing: `Ctrl+b d`
2. Sessions persist even if you disconnect
3. Reconnect with: `herdr attach session-name`

### "Too many panes - can't see anything"

You've created too many panes and they're too small to be useful.

**Solution:**
1. Close some panes: `Ctrl+b x`
2. Use windows to organize instead of panes
3. Use zoom mode for one pane: `Ctrl+b z`
4. Keep the number of visible panes to 2-4 maximum

### "Forgot to save something before closing a pane"

As long as you haven't killed the session, your scrollback is still there.

**Solution:**
1. Create a new pane in the same window
2. Press `Ctrl+b [` to enter copy mode
3. Scroll up to find the text you need
4. Copy it with `Enter` and paste in the new pane

---

## Tips for Getting Better

1. **Start simple.** Create one session, practice splitting panes, running agents. Master this before complex workflows.

2. **Use meaningful names.** Name your sessions and windows clearly so you know what they're for.

3. **Organize by task.** Each window should represent a logical task or stage in your workflow.

4. **Monitor regularly.** Periodically switch between panes to see progress on all your agents.

5. **Customize keybindings.** If you have a workflow-specific set of commands, create shortcuts for them.

6. **Document your workflows.** Write down patterns you use frequently so you can recreate them quickly.

7. **Use synchronize-panes for repeated tasks.** When you need to run the same command in multiple panes, enable synchronization.

8. **Keep scrollback accessible.** Always be able to scroll up to see historical output and results.

---

## Next Steps

- Set up your first session with a named workflow
- Practice creating windows and panes
- Start your first AI agent and monitor its output
- Create a multi-window workflow for a real task
- Customize your configuration file
- Build a workflow pattern that matches your work style
