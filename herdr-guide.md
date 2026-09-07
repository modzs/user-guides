# Herdr Guide for Beginners

A complete guide to getting started with Herdr, a terminal workspace manager built for running AI coding agents.

**Platform Support:** This guide covers Linux, macOS, and Windows. The concepts and keybindings are the same everywhere; only the installation steps differ.

**Version note:** Everything in this guide was checked against **Herdr 0.8.2**. Herdr moves quickly, so if a command here disagrees with your machine, your machine is right. Run `herdr --help` and `herdr --version` to see what your build actually supports.

## Table of Contents
1. [What is Herdr?](#what-is-herdr)
2. [Installation](#installation)
3. [The Big Picture](#the-big-picture)
4. [Your First Session](#your-first-session)
5. [What You Actually See](#what-you-actually-see)
6. [Start With the Mouse](#start-with-the-mouse)
7. [The Prefix Key](#the-prefix-key)
8. [Workspaces](#workspaces)
9. [Tabs and Panes](#tabs-and-panes)
10. [Git Worktrees](#git-worktrees)
11. [Working with AI Agents](#working-with-ai-agents)
12. [Driving Herdr from Scripts](#driving-herdr-from-scripts)
13. [Configuration](#configuration)
14. [Workflow Patterns](#workflow-patterns)
15. [Command Reference](#command-reference)
16. [Keybinding Reference](#keybinding-reference)
17. [Troubleshooting](#troubleshooting)
18. [Next Steps](#next-steps)

---

## What is Herdr?

**Herdr** is a *terminal workspace manager*. Two ideas are packed into that phrase, and it is worth unpacking both before you install anything.

**First, it is a multiplexer.** A background program (the *server*) owns your real terminal programs. What you look at (the *client*) is just a window onto them. Close your terminal, lose your SSH connection, or shut the laptop lid, and the programs keep running. Open Herdr again and they are exactly where you left them.

If you have never used a multiplexer before: think of it like the difference between a phone call and a voicemail box. Normally, closing a terminal is a hung-up call - whatever was running dies with it. Herdr moves the programs somewhere that outlives the call.

**Second, it is agent-aware.** Herdr recognizes AI coding agents running inside its panes and tracks what each one is doing. A sidebar shows, across every project you have open, which agent is `working`, which is `blocked` waiting on you, and which is `done`. That is the part no ordinary multiplexer does, and it is the reason Herdr exists.

### Why use it?

- **Nothing dies when you disconnect.** Long agent runs survive closed terminals, dropped SSH, and reboots of your *local* machine when the server is remote.
- **You can see every agent at once.** One glance tells you which of six projects needs a human right now.
- **It is mouse-first.** You can click panes, drag borders, and use right-click menus. You do not have to memorize a single keybinding to be productive on day one.
- **It is scriptable.** A local socket API and a `herdr` CLI let scripts - and agents themselves - create panes, start other agents, read their output, and wait on their state.
- **Git worktrees are built in.** Working on three branches of one repo at once is a first-class operation, not a manual chore.

### Is Herdr tmux?

No, and this matters. Herdr is genuinely inspired by tmux and shares its prefix-key idea, but it is a different program with a different structure and a different command set. **tmux commands, tmux config syntax, and `.tmux.conf` advice do not work in Herdr.** If you find a "Herdr" tip that looks exactly like tmux with the name swapped, it is wrong. Check it against `herdr --help` on your own machine.

---

## Installation

### The supported path (Linux and macOS)

```bash
curl -fsSL https://herdr.dev/install.sh | sh
```

Then start it:

```bash
herdr
```

### The supported path (Windows)

In PowerShell:

```powershell
powershell -ExecutionPolicy Bypass -c "irm https://herdr.dev/install.ps1 | iex"
```

If your workplace's endpoint security blocks that, use Command Prompt instead:

```cmd
curl.exe -fsSLo install.cmd https://herdr.dev/install.cmd && install.cmd && del install.cmd
```

### Package managers

| Manager | Command | Notes |
|---------|---------|-------|
| Homebrew (macOS/Linux) | `brew install herdr` | In `homebrew/core`. Update with `brew upgrade herdr`. |
| mise | `mise use -g herdr` | If mise says `herdr not found in mise tool registry`, update mise. |
| Nix | `nix profile install github:herdrdev/herdr/v0.x.y` | Replace `v0.x.y` with a real release tag. |
| Omarchy Linux | `sudo pacman -S herdr` | Comes from Omarchy's own `[omarchy]` repository. |

**Plain Arch Linux:** Herdr is **not** in the official Arch repositories. Omarchy ships it because Omarchy adds its own repo. On stock Arch, use the installer script above. There are community-maintained AUR packages named `herdr` and `herdr-bin`, but they are not published or supported by the Herdr project, so treat them the way you treat any AUR package.

**Debian, Ubuntu, and Fedora:** there is no official `apt` or `dnf` package. Use the installer script, or download a binary directly.

### Downloading a binary yourself

Releases live at <https://github.com/herdrdev/herdr/releases>. Pick the asset matching your machine:

| System | Asset |
|--------|-------|
| Linux x86_64 | `herdr-linux-x86_64` |
| Linux aarch64 | `herdr-linux-aarch64` |
| macOS Intel | `herdr-macos-x86_64` |
| macOS Apple silicon | `herdr-macos-aarch64` |
| Windows x86_64 | `herdr-windows-x86_64.zip` |

On Linux or macOS:

```bash
chmod +x herdr-linux-x86_64
mv herdr-linux-x86_64 ~/.local/bin/herdr
```

On Windows, keep the whole extracted folder together. The zip contains `herdr.exe` plus a runtime it needs; copying only the `.exe` will not work.

### Check that it worked

```bash
herdr --version
```

You should see something like `herdr 0.8.2`. If your shell says `command not found`, restart your terminal so it picks up the new `PATH`.

### Updating

If you installed with the `herdr.dev` installer:

```bash
herdr update
```

If you installed with Homebrew, mise, or Nix, update through **that** tool instead. `herdr update` is only for installs Herdr manages itself.

Direct installs follow the `stable` channel. You can opt into early builds, which get fixes sooner but can regress:

```bash
herdr channel show          # which channel am I on?
herdr channel set preview   # early builds from the development branch
herdr channel set stable    # back to normal
```

Installing a new binary does not always replace a server that is already running. If `herdr status` says a restart is needed, stop the server and start Herdr again:

```bash
herdr server stop
herdr
```

Be aware that stopping the server **exits every program running in its panes**. Finish or park your work first.

---

## The Big Picture

Herdr nests four things inside each other. Learn them in this order, because each one only makes sense in terms of the one above it.

```
Session          the persistent background server (you usually need exactly one)
  └── Workspace  one project, repo, task, or investigation
        └── Tab  one layout inside that project ("agents", "logs", "tests")
              └── Pane  one real terminal
```

Plus one thing that is not a container:

```
Agent            a coding agent Herdr recognizes running inside a pane
```

### Session

A **session** is the background server itself: one persistent namespace holding all your workspaces. Running `herdr` with no arguments launches or attaches to the `default` session.

**Most people never need a second session.** Named sessions are completely separate universes - separate sockets, separate saved state, no shared workspaces. Use one only when you truly want isolation, such as a sandbox you can throw away without touching your real work.

```bash
herdr                      # the default session
herdr --session work       # launch or attach to a session named "work"
herdr session list         # what sessions exist?
```

### Workspace

A **workspace** is the project-level container, and it is the level you will actually use every day. **One workspace per repo, per task, or per investigation.**

This is the answer to "why would I make a workspace instead of a tab?" A workspace is where the sidebar summarizes state. If `api-server` and `frontend` are separate workspaces, the sidebar can tell you at a glance that the agent in `api-server` is blocked while the one in `frontend` is still working. If you cram both into one workspace, that distinction disappears.

### Tab

A **tab** is a layout *inside* a workspace. Same project, different view. A typical repo workspace might have three tabs:

- `agents` - the coding agents doing the work
- `logs` - a dev server and a log tail
- `tests` - a test watcher

Tabs are for splitting up *your own* attention within one project. Workspaces are for separating *projects*.

### Pane

A **pane** is a real terminal running a real program. Panes can be split to the right or downward, resized, renamed, zoomed, and closed. A pane keeps running when you detach.

### Agent

An **agent** is a coding agent that Herdr has recognized inside a pane. Herdr detects agents automatically by watching the foreground process and the screen contents. Every agent has a state:

| State | What it means |
|-------|---------------|
| `blocked` | It needs input, approval, or a decision **from you**. |
| `working` | It is actively running. |
| `done` | It finished while you were not looking. |
| `idle` | It is ready for input, and you have already seen it. |
| `unknown` | An agent is there, but Herdr cannot confidently classify it. Not the same as "finished successfully". |

The difference between `done` and `idle` is only whether you have looked at that tab yet. That is deliberate: `done` is the queue of things that finished behind your back.

---

## Your First Session

Five steps, in order.

**1. Go to a project and start Herdr.**

```bash
cd ~/projects/my-app
herdr
```

Herdr launches its background server, creates a workspace for you automatically, and shows you a first-run onboarding screen.

**2. Start your coding agent in the pane.**

```bash
claude
```

Or `codex`, `pi`, `opencode`, or another supported agent. You do not have to tell Herdr about it. It notices, and the sidebar starts tracking its state.

**3. Click around.** Click panes, tabs, and workspaces to focus them. Drag the borders between panes to resize. Right-click for a menu that includes splitting and creating tabs. Drag-select text to copy it - no `Ctrl+C` needed.

**4. Detach.** Press `Ctrl+b` then `q`, or just close the terminal window. Everything keeps running.

**5. Come back.**

```bash
herdr
```

You are back exactly where you were, and the agent has kept working the whole time.

When you genuinely want to stop everything and exit every program in every pane:

```bash
herdr server stop
```

---

## What You Actually See

Prose only gets you so far here, so: the Herdr screen is divided into a **sidebar** down the left and a **pane area** filling the rest, with a **tab bar** across the top of the pane area.

Here is the layout, schematically. Real Herdr is drawn in color with rounded borders and theme-dependent icons, so treat this as a map rather than a screenshot:

```
┌────────────────────────┬──────────────────────────────────────────────────┐
│ SPACES                 │  1: agents │ 2: logs │ 3: tests                  │
│ ● api-server           ├─────────────────────────┬────────────────────────┤
│   main  +2 ~1          │                         │                        │
│ ● frontend             │  claude                 │  $ npm run dev         │
│   feat/login  ~3       │  > implementing the     │  ready on :3000        │
│ ● notes                │    login handler...     │                        │
│   main                 │                         │                        │
│                        │                         │                        │
│ AGENTS                 │                         │                        │
│ ● api-server / agents  │                         │                        │
│   claude               │                         │                        │
│ ● frontend / agents    │                         │                        │
│   codex                │                         │                        │
└────────────────────────┴─────────────────────────┴────────────────────────┘
      sidebar (26 cols)                     pane area
```

**The sidebar is the point of Herdr.** It has two sections:

- **Spaces** lists your workspaces, each with a status mark and, for git repos, the branch and a short dirty-state summary.
- **Agents** lists every recognized agent across *all* workspaces, with its state mark, where it lives, and what it is.

Those coloured marks are the whole value proposition. You do not have to visit six projects to find out which one is waiting on you; the sidebar already told you.

Press `Ctrl+b` then `b` to collapse or expand the sidebar when you want the full width for a pane.

### Checking on things from outside

You do not have to be attached to inspect a session. From any ordinary terminal:

```bash
herdr status
```

Real output:

```
client:
  version: 0.8.2
  channel: stable
  protocol: 20

server:
  status: running
  version: 0.8.2
  protocol: 20
  compatible: yes
  socket: /home/you/.config/herdr/herdr.sock

update:
  restart_needed: no
```

And to see the sessions on the machine:

```bash
herdr session list
```

```
name                 status   directory                                        socket
default              running  /home/you/.config/herdr                          /home/you/.config/herdr/herdr.sock
```

---

## Start With the Mouse

This section is short because the advice is short: **use the mouse first.** Herdr is mouse-native, and you can do everything through it. Learning keybindings is an optional optimization for later, not an entry requirement.

| What you want | How |
|---------------|-----|
| Focus a pane, tab, workspace, or agent | Click it |
| Resize panes | Drag the border between them |
| Split a pane, create a tab, rename things | Right-click for the menu |
| Copy text | Drag-select it (it copies automatically) |
| Copy a single word or token | Double-click it |
| Open a link in a pane | `Ctrl`-click it |

If you would rather your terminal handled clicks normally - for example so `Cmd`-click opens URLs the way it does everywhere else - turn Herdr's mouse capture off in your config:

```toml
[ui]
mouse_capture = false
```

---

## The Prefix Key

### What a prefix is, and why one exists

Herdr sits between your terminal and the programs inside it. Those programs already claim nearly every key: `Ctrl+C` interrupts, `Ctrl+R` searches history, editors take most of the rest. If Herdr grabbed keys directly, it would break the programs it is hosting.

So Herdr reserves exactly one key combination - the **prefix** - and listens for a single command right after it.

**The default prefix is `Ctrl+b`.**

`prefix+c` is shorthand for: hold `Ctrl` and press `b`, release both, then press `c`. It is two separate keystrokes, not a three-key chord. This trips up almost everyone at first.

Press `Ctrl+b` then `?` at any time to see every binding your build actually has. That live list beats any documentation, including this page.

### ⚠️ Ctrl+b collides with NeoVim

If you also use NeoVim or Vim, you have a conflict waiting for you: **`Ctrl+b` is page-up in Vim and the prefix key in Herdr.** Inside a Herdr pane, Herdr sees `Ctrl+b` first and swallows it, so page-up stops working in your editor.

Your options:

1. **Use `Ctrl+u`** to scroll up by half a page in Vim. Most Vim users prefer it anyway, and nothing conflicts.
2. **Change Herdr's prefix.** Put this in `~/.config/herdr/config.toml` and reload:
   ```toml
   [keys]
   prefix = "ctrl+a"
   ```
3. **Live with it** and only page up with `PageUp`.

Option 1 costs nothing and is what most people land on.

### The five bindings worth learning first

| Action | Keys |
|--------|------|
| New tab | `Ctrl+b` then `c` |
| Split right / split down | `Ctrl+b` then `v` / `Ctrl+b` then `-` |
| Move between panes | `Ctrl+b` then `h` `j` `k` `l` |
| Workspace navigation | `Ctrl+b` then `w` |
| Detach, leaving everything running | `Ctrl+b` then `q` |

The `h` `j` `k` `l` directions are the same left/down/up/right that Vim uses. Everything else can stay on the mouse until you want it.

The full list is in the [Keybinding Reference](#keybinding-reference) below.

---

## Workspaces

### Creating one

Interactively: `Ctrl+b` then `Shift+N`.

From the command line:

```bash
herdr workspace create --cwd ~/projects/api --label api --no-focus
```

`--no-focus` means "make it, but do not yank my attention over to it". That is the default for creation commands; `--focus` opts in to jumping there.

Real response (Herdr prints JSON so scripts can use it, trimmed here for readability):

```json
{"result":{"type":"workspace_created",
  "workspace":{"workspace_id":"w1","label":"api","tab_count":1,"pane_count":1},
  "tab":{"tab_id":"w1:t1","label":"1"},
  "root_pane":{"pane_id":"w1:p1","cwd":"/home/you/projects/api"}}}
```

Notice the IDs: `w1` is the workspace, `w1:t1` is its first tab, `w1:p1` is its first pane. **Creating a workspace also creates a tab and a pane inside it** - you never have to build those by hand.

### Listing and moving between them

```bash
herdr workspace list
```

```json
{"result":{"type":"workspace_list","workspaces":[
  {"workspace_id":"w1","label":"api","agent_status":"unknown","focused":true,
   "tab_count":1,"pane_count":1}]}}
```

| Action | Keys | Command |
|--------|------|---------|
| Workspace picker | `Ctrl+b` then `w` | - |
| Go-to picker (jump anywhere) | `Ctrl+b` then `g` | - |
| New workspace | `Ctrl+b` then `Shift+N` | `herdr workspace create` |
| Rename workspace | `Ctrl+b` then `Shift+W` | `herdr workspace rename <id> <label>` |
| Close workspace | `Ctrl+b` then `Shift+D` | `herdr workspace close <id>` |
| Toggle the sidebar | `Ctrl+b` then `b` | - |

Closing a workspace closes its tabs and panes, which exits the programs in them. By default Herdr asks first; that is `ui.confirm_close`.

---

## Tabs and Panes

### Tabs

| Action | Keys | Command |
|--------|------|---------|
| New tab | `Ctrl+b` then `c` | `herdr tab create --label logs` |
| Next / previous tab | `Ctrl+b` then `n` / `p` | - |
| Jump to tab 1-9 | `Ctrl+b` then `1`..`9` | `herdr tab focus <tab_id>` |
| Rename tab | `Ctrl+b` then `Shift+T` | `herdr tab rename <tab_id> <label>` |
| Close tab | `Ctrl+b` then `Shift+X` | `herdr tab close <tab_id>` |

```bash
herdr tab list --workspace w1
```

```json
{"result":{"type":"tab_list","tabs":[
  {"tab_id":"w1:t1","label":"1","number":1,"pane_count":2,"focused":true},
  {"tab_id":"w1:t2","label":"logs","number":2,"pane_count":1,"focused":false}]}}
```

Closing a workspace's **last** tab closes the workspace too.

### Panes

| Action | Keys | Command |
|--------|------|---------|
| Split right | `Ctrl+b` then `v` | `herdr pane split <id> --direction right` |
| Split down | `Ctrl+b` then `-` | `herdr pane split <id> --direction down` |
| Move left/down/up/right | `Ctrl+b` then `h`/`j`/`k`/`l` | `herdr pane focus --direction left` |
| Cycle to next pane | `Ctrl+b` then `Tab` | - |
| Cycle to previous pane | `Ctrl+b` then `Shift+Tab` | - |
| Swap panes | `Ctrl+b` then `Shift+H`/`J`/`K`/`L` | `herdr pane swap --direction left` |
| Zoom (fill the tab) | `Ctrl+b` then `z` | `herdr pane zoom --toggle` |
| Resize mode | `Ctrl+b` then `r` | `herdr pane resize --direction left` |
| Rename pane | `Ctrl+b` then `Shift+P` | `herdr pane rename <id> <label>` |
| Close pane | `Ctrl+b` then `x` | `herdr pane close <id>` |

Two naming notes that catch people out:

- Herdr calls a **right**-hand split "vertical" (`split_vertical`, `Ctrl+b v`) because the *divider* is vertical, and a **downward** split "horizontal" (`split_horizontal`, `Ctrl+b -`). The key characters are the memorable part: `v` for a vertical divider, `-` for a horizontal one.
- **Zoom is not fullscreen.** `Ctrl+b z` makes the focused pane temporarily fill its tab. Press it again to go back. Nothing is lost.

Resize mode (`Ctrl+b r`) is a *mode*: you enter it, then use `h` `j` `k` `l` repeatedly to nudge the split, then press `Esc`. You do not re-press the prefix each time.

### Reading the layout

```bash
herdr pane layout --pane w1:p2
```

Real output from a tab split once to the right:

```json
{"result":{"type":"pane_layout","layout":{
  "workspace_id":"w1","tab_id":"w1:t1","zoomed":false,
  "focused_pane_id":"w1:p1",
  "area":{"x":26,"y":1,"width":94,"height":39},
  "panes":[
    {"pane_id":"w1:p1","focused":true, "rect":{"x":26,"y":1,"width":47,"height":39}},
    {"pane_id":"w1:p2","focused":false,"rect":{"x":73,"y":1,"width":47,"height":39}}],
  "splits":[{"id":"split_0_root","direction":"right","ratio":0.5}]}}}
```

That `"x":26` is the sidebar's width, and `"y":1` is the tab bar - the two numbers behind the diagram earlier in this guide.

### Scrollback and copying

- **Mouse:** drag-select. It copies automatically.
- **Keyboard:** `Ctrl+b` then `[` enters copy mode. Move with `h` `j` `k` `l`, `w`/`b`/`e` by word, `PageUp`/`PageDown`, or `Ctrl+u`/`Ctrl+d`. Press `/` to search forward, `?` backward, then `n`/`N` to repeat. Press `v` or `Space` to start selecting, `y` or `Enter` to copy, `q` or `Esc` to leave.
- **Open the scrollback in your editor:** `Ctrl+b` then `e`.

Copy mode does not pause the program. Output keeps arriving while you read history.

Note the prefix collision again: inside copy mode, `Ctrl+b` still means "prefix", so it will *not* page up. Use `PageUp` or `Ctrl+u`, or change your prefix.

---

## Git Worktrees

This is a feature most multiplexers do not have, and it is worth understanding even if you have never used `git worktree` before.

### The problem it solves

Normally a git repository has one checked-out branch at a time. To work on a second branch you stash, switch, and lose your build state, running servers, and open editors.

A **worktree** is a second directory containing a second branch of the same repository. Both are live at once. No stashing, no switching, no lost state.

Herdr makes each worktree its **own workspace**, grouped with the parent repo in the sidebar. So "work on three branches simultaneously, each with its own agent" stops being an ordeal.

### Using it

Interactively: `Ctrl+b` then `Shift+G` creates a worktree from the current workspace.

From the command line:

```bash
herdr worktree create --workspace w1 --branch feature-login
```

If `feature-login` already exists locally, Herdr checks it out. Otherwise it creates the branch from `--base`, or from `HEAD` if you do not say. Without `--path`, the checkout goes under the directory set by `worktrees.directory` in your config.

Real response, trimmed:

```json
{"result":{"type":"worktree_created",
  "workspace":{"workspace_id":"w2","label":"feature-login",
    "worktree":{"checkout_path":"/tmp/demo/wt/feature-login",
                "is_linked_worktree":true,
                "repo_name":"demo-repo",
                "repo_root":"/tmp/demo/demo-repo"}},
  "root_pane":{"pane_id":"w2:p1","cwd":"/tmp/demo/wt/feature-login"}}}
```

And listing them:

```bash
herdr worktree list --workspace w1
```

```json
{"result":{"type":"worktree_list",
  "source":{"repo_name":"demo-repo","repo_root":"/tmp/demo/demo-repo"},
  "worktrees":[
    {"branch":"main","path":"/tmp/demo/demo-repo",
     "is_linked_worktree":false,"open_workspace_id":"w1"},
    {"branch":"feature-login","path":"/tmp/demo/wt/feature-login",
     "is_linked_worktree":true,"open_workspace_id":"w2"}]}}
```

Git agrees, because these are ordinary git worktrees:

```
$ git worktree list
/tmp/demo/demo-repo        e7b6bf2 [main]
/tmp/demo/wt/feature-login e7b6bf2 [feature-login]
```

### Removing one - read this before you close anything

There are two different operations and they are not interchangeable:

| Command | What it does |
|---------|--------------|
| `herdr workspace close <id>` | Closes the Herdr workspace only. **The checkout stays on disk.** |
| `herdr worktree remove --workspace <id>` | Runs `git worktree remove` and deletes the checkout directory. |

`worktree remove` **never deletes the branch**. If git refuses because the checkout has uncommitted changes, it will tell you; pass `--force` only when you are sure you want to throw that work away.

If you run `worktree list` outside a git repository, you get a clear error rather than silence:

```json
{"error":{"code":"not_git_worktree",
  "message":"Herdr worktree actions require a path inside a Git work tree"}}
```

---

## Working with AI Agents

This is what Herdr is for. Everything above is the scaffolding.

### Just run it

The simplest thing works: open a pane in your project and run your agent.

```bash
claude
```

Herdr detects it. No registration, no config. The sidebar starts showing its state immediately.

Supported agents include `pi`, `claude`, `codex`, `gemini`, `cursor`, `devin`, `agy`, `cline`, `omp`, `mastracode`, `opencode`, `copilot`, `kimi`, `kiro`, `droid`, `amp`, `grok`, `hermes`, `kilo`, `qodercli`, `qwen`, and `maki`. The current list for your build is in `herdr agent start --help`.

### Integrations make detection better

Herdr works out of the box by reading the screen. For some agents it can do better if you install a small hook:

```bash
herdr integration install claude
herdr integration status
```

Real output:

```
pi: not installed (/home/you/.pi/agent/extensions/herdr-agent-state.ts)
omp: not installed (/home/you/.omp/agent/extensions/herdr-omp-agent-state.ts)
claude: current (v8) (/home/you/.claude/hooks/herdr-agent-state.sh)
codex: not installed (/home/you/.codex/herdr-agent-state.sh)
...
```

Depending on the agent, an integration gives lifecycle state, native session restore, or both. Installing one does not necessarily replace screen detection - for Claude Code, the integration adds session restore while state still comes from the screen.

### Seeing what your agents are doing

```bash
herdr agent list
```

With one agent running, trimmed to the interesting fields:

```json
{"result":{"type":"agent_list","agents":[
  {"name":"helper","agent":"claude","agent_status":"idle",
   "pane_id":"w1:p1","tab_id":"w1:t1","workspace_id":"w1",
   "cwd":"/tmp/demo/demo-repo",
   "interactive_ready":true,
   "terminal_title":"✳ Claude Code"}]}}
```

`herdr agent get <target>` shows one agent. A *target* is either the agent's name, or the pane ID hosting it (`w1:p1`).

That `"name":"helper"` above only appears because the agent was launched with `herdr agent start helper`. **An agent you started by typing `claude` yourself has no name**, so you address it by pane ID. Give it one whenever a stable handle is useful:

```bash
herdr agent rename w1:p1 helper
```

Read what is on its screen without stealing focus:

```bash
herdr agent read helper --source visible
herdr agent read helper --source recent-unwrapped --lines 120
```

| Source | Use it for |
|--------|-----------|
| `visible` | What is on screen right now. Best for checking a UI state. |
| `recent` | Recent scrollback, wrapped as displayed. |
| `recent-unwrapped` | Recent scrollback with soft wraps removed. Best for logs. |
| `detection` | The snapshot Herdr's own agent detection looks at. |

Reading through the CLI does **not** mark an agent as seen, so a `done` agent stays `done` in your sidebar until you actually look at its tab.

### Talking to an agent

```bash
herdr agent prompt helper "Add tests for the login handler"
```

That submits the text plus Enter, and it works even while the agent is already busy.

To submit and then **wait** until the agent settles:

```bash
herdr agent prompt helper "Review the current diff" --wait --timeout 200000
```

Or just wait on an agent that is already going:

```bash
herdr agent wait helper --until idle --until done --timeout 120000
```

Timeouts are in milliseconds. Without `--timeout`, waits are indefinite.

For raw keystrokes - dismissing a dialog, answering a menu, interrupting - use `send-keys`:

```bash
herdr agent send-keys helper esc
herdr agent send-keys helper ctrl+c
herdr agent send-keys helper down enter
```

### A real, complete transcript

Here is an actual run: start Claude Code in a pane, get past its first-run prompt, and give it work. This is copied from a live session, not invented.

**Start it:**

```bash
$ herdr agent start helper --kind claude --pane w1:p1
{"error":{"code":"agent_not_ready",
  "message":"agent helper is blocked during startup and is not ready for prompts"}}
```

That is not a failure, and it is worth understanding. `agent start` waits until the agent is ready for prompts. Claude Code opened a first-run trust prompt instead, so Herdr correctly reported `blocked` and returned rather than hanging. **The agent is running.** The name `helper` already works for reading and sending keys.

**See what it is stuck on:**

```bash
$ herdr agent list
{"result":{"agents":[{"name":"helper","agent":"claude",
  "agent_status":"blocked","launch_pending":true,"pane_id":"w1:p1"}]}}

$ herdr agent read helper --source visible
```

```
demo-repo main ❯ claude

──────────────────────────────────────────────────────────────────────────
 Accessing workspace:

 /tmp/demo/demo-repo

 Quick safety check: Is this a project you created or one you trust? ...

 ❯ No, exit
   Yes, I trust this folder

 Enter to confirm · Esc to cancel
```

If you want to know *why* Herdr called that `blocked`, ask it:

```bash
$ herdr agent explain helper
agent: claude
state: blocked
manifest: remote:.../agent-detection/remote/claude.toml 2026.09.04.1
rule: live_blocked_form (region=after_last_horizontal_rule priority=980)
evidence: " Accessing workspace:\n\n /tmp/demo/demo-repo\n\n Quick safety check: ..."
```

**Answer the prompt and wait for it to be ready:**

```bash
$ herdr agent send-keys helper down enter
{"result":{"type":"ok"}}

$ herdr agent wait helper --until idle --timeout 120000
{"result":{"agent":{"name":"helper","agent":"claude","agent_status":"idle",
  "interactive_ready":true,
  "agent_session":{"kind":"id","value":"c249d897-f10c-4be7-959b-2442ac22ad0e"},
  "terminal_title":"✳ Claude Code"}}}
```

**Give it work and wait for the answer:**

```bash
$ herdr agent prompt helper "Reply with exactly the word READY" \
    --wait --until idle --until done --timeout 200000
{"result":{"type":"agent_prompted","agent":{"name":"helper",
  "agent_status":"idle","terminal_title":"✳ READY"}}}
```

The whole loop - start, unblock, prompt, wait, read - runs from an ordinary terminal, with no attached UI, while you are doing something else.

### Agent commands at a glance

| Command | What it does |
|---------|--------------|
| `herdr agent list` | Every recognized agent in the session |
| `herdr agent get <target>` | One agent's full state |
| `herdr agent read <target>` | Read its terminal output |
| `herdr agent prompt <target> <text>` | Submit a prompt (add `--wait` to block) |
| `herdr agent wait <target>` | Block until it reaches a state |
| `herdr agent send-keys <target> <keys>` | Send raw keys |
| `herdr agent focus <target>` | Jump the UI to it (marks it seen) |
| `herdr agent attach <target>` | Attach your terminal straight to it |
| `herdr agent start <name> --kind K --pane P` | Launch a supported agent in an existing pane |
| `herdr agent rename <target> <name>` | Give a running agent a stable name |
| `herdr agent explain <target>` | Why Herdr classified it that way |

A few rules that will save you confusion:

- **`agent start` never creates layout.** It needs a pane that already exists and is sitting at a shell prompt. Create the pane first with `pane split` or `tab create`.
- **Agent names must match `[a-z][a-z0-9_-]{0,31}`** and be unique among live agents. The name follows whatever agent currently occupies that pane, and clears when it exits.
- **`agent prompt` on a `blocked` agent returns `agent_blocked` and sends nothing.** That is a safety feature: read the dialog and answer it with `send-keys` rather than blindly typing into an approval prompt.

### Teaching an agent to drive Herdr

Herdr ships its own instruction file for coding agents:

```bash
herdr --skill
```

That prints a skill document telling an agent how to control Herdr from inside a pane - splitting panes, running commands without stealing focus, reading output, and waiting on other agents. Save it wherever your agent reads instructions from, and it can start orchestrating work for you.

The same file is at <https://raw.githubusercontent.com/herdrdev/herdr/master/skills/herdr/SKILL.md>.

---

## Driving Herdr from Scripts

Every `herdr` subcommand under `workspace`, `tab`, `pane`, `agent`, and `worktree` talks to the running server and prints JSON. That makes Herdr scriptable.

**Capture IDs from responses. Never guess them.**

```bash
created=$(herdr workspace create --cwd ~/project --label api --no-focus)
pane_id=$(printf '%s\n' "$created" | jq -r '.result.root_pane.pane_id')

split=$(herdr pane split "$pane_id" --direction right --no-focus)
review_pane=$(printf '%s\n' "$split" | jq -r '.result.pane.pane_id')
```

### Panes versus agents

Use **pane** commands for ordinary programs - shells, servers, tests, build watchers. Use **agent** commands when you need Herdr to understand agent lifecycle state.

```bash
herdr pane run w1:p3 "npm test -- --watch"
herdr pane wait-output w1:p3 --regex "passed|failed" --timeout 120000
```

`pane run` submits the command with Enter atomically, which is what you want. `pane send-text` types text *without* Enter, and `pane send-keys` sends key presses - both are lower-level tools for when you need precise control.

A real `wait-output` result:

```json
{"result":{"type":"output_matched",
  "pane_id":"w1:p2",
  "matched_line":"echo hello-from-herdr",
  "read":{"source":"recent_unwrapped","text":"echo hello-from-herdr"}}}
```

Note that `wait-output` checks the snapshot **immediately**, including text that is already on screen. In the run above it matched the echoed command line itself, not the output. If that matters, match on something only the output can contain.

### Handy environment variables

Inside a Herdr pane, these are set for you:

| Variable | Meaning |
|----------|---------|
| `HERDR_ENV` | `1` when you are inside a Herdr-managed pane |
| `HERDR_PANE_ID` | This pane's ID |
| `HERDR_TAB_ID` | This tab's ID |
| `HERDR_WORKSPACE_ID` | This workspace's ID |

Because of `HERDR_PANE_ID`, many pane commands accept `--current` to mean "the pane I am running in":

```bash
herdr pane split --current --direction right --no-focus
```

Full details: <https://herdr.dev/docs/agent-automation/> and <https://herdr.dev/docs/cli-reference/>.

---

## Configuration

### Where the file lives

- Linux and macOS: `~/.config/herdr/config.toml`
- Windows: `%APPDATA%\herdr\config.toml`

**Herdr works fine with no config file at all.** Only create one when you want to change something.

To see the options with their defaults and an explanation of each:

```bash
herdr --default-config
```

That output is the authoritative schema for everything it prints, but it is not quite the whole list: on 0.8.2 it leaves out `copy_mode` and the `swap_pane_left` / `swap_pane_down` / `swap_pane_up` / `swap_pane_right` family, all of which are real and rebindable. Before concluding a key does not exist, check the complete reference at <https://herdr.dev/docs/configuration/#keybindings>.

### A starter config

This is a realistic beginner config - a theme, a shell, and a saner prefix if you use Vim:

```toml
[theme]
name = "catppuccin"

[terminal]
# Empty means "use $SHELL"
default_shell = ""

[keys]
# Change this if Ctrl+b fights your editor
prefix = "ctrl+b"

[ui]
sidebar_width = 26
confirm_close = true
```

### The sections that exist

| Section | What it controls |
|---------|------------------|
| `[theme]` | Built-in theme, light/dark auto-switching, individual colour overrides |
| `[terminal]` | Default shell, login-shell mode, working directory for new panes |
| `[update]` | Update channel and background version checks |
| `[keys]` | The prefix and every action binding |
| `[server]` | Virtual terminal size when no client is attached |
| `[worktrees]` | Where `worktree create` puts new checkouts |
| `[ui]` | Sidebar, mouse, borders, tab bar, window title, notifications, sounds |
| `[session]` | Resuming agent sessions after a server restart |
| `[remote]` | SSH behaviour for `herdr --remote` |
| `[experimental]` | Opt-in features that may change |
| `[advanced]` | Scrollback buffer size |

Available built-in themes: `catppuccin`, `terminal`, `tokyo-night`, `dracula`, `nord`, `gruvbox`, `one-dark`, `solarized`, `kanagawa`, `rose-pine`, `vesper`.

### Changing keybindings

Bindings live under `[keys]` and use explicit syntax. `"prefix+n"` requires the prefix first; `"ctrl+alt+n"` is a direct shortcut with no prefix.

```toml
[keys]
prefix = "ctrl+a"

# Give yourself direct chords in addition to the prefix versions
focus_pane_left  = ["prefix+h", "ctrl+alt+h"]
focus_pane_down  = ["prefix+j", "ctrl+alt+j"]
focus_pane_up    = ["prefix+k", "ctrl+alt+k"]
focus_pane_right = ["prefix+l", "ctrl+alt+l"]
```

If you go prefix-free, `ctrl+alt` is the safest modifier family - terminals and desktop environments leave it mostly alone. Avoid `ctrl+alt+t` (opens a terminal on Ubuntu and Fedora), `ctrl+alt+l` (KDE lock screen), `ctrl+alt+arrows` (GNOME workspaces), and `ctrl+alt+F1`-`F12` (Linux virtual consoles).

### Applying and checking changes

```bash
herdr config check          # validate the file and print problems
herdr server reload-config  # apply to the running server
```

Or press `Ctrl+b` then `Shift+R` inside Herdr.

If you break your keybindings badly enough to lock yourself out:

```bash
herdr config reset-keys
```

That backs up `config.toml` and strips your custom bindings.

---

## Workflow Patterns

### One repo, one workspace, three tabs

The everyday shape. One workspace for the project; tabs for the kinds of work.

```
workspace: my-app
  tab "agents"  - split into two panes: Claude on the left, Codex on the right
  tab "logs"    - dev server, log tail
  tab "tests"   - test watcher
```

Switch tabs with `Ctrl+b n` / `Ctrl+b p`, or jump directly with `Ctrl+b 1`, `Ctrl+b 2`, `Ctrl+b 3`.

### Several projects at once

One workspace per project. The sidebar becomes your dashboard: you stop checking on projects and start responding to whichever one turns `blocked`.

```
SPACES
● api-server    (working)
● frontend      (blocked  ← this one wants you)
● docs          (done)
```

Move between them with `Ctrl+b w`.

### Three branches of one repo

```bash
herdr worktree create --workspace w1 --branch feature-login
herdr worktree create --workspace w1 --branch bugfix-auth
```

Three workspaces, three checkouts, three agents, one repo, zero stashing. Herdr groups the worktrees under the parent in the sidebar so it stays legible.

### A supervised agent run

Start work, go do something else, get told when it needs you:

```bash
herdr agent prompt reviewer "Review the diff on this branch" --wait --until blocked --until done
herdr agent read reviewer --source recent-unwrapped --lines 120
```

The command blocks until the agent either finishes or needs a decision, then you read the result.

### One agent supervising others

With `herdr --skill` installed into your agent's instructions, a lead agent can split a pane, start a second agent in it, hand it a task, wait, and collect the result - all through the same CLI you have been reading about.

---

## Command Reference

### Launch and status

| Command | Description |
|---------|-------------|
| `herdr` | Launch or attach to the default session |
| `herdr --session <name>` | Launch or attach to a named session |
| `herdr --remote <ssh-target>` | Attach to a Herdr server on another machine |
| `herdr status` | Client and server status |
| `herdr --version` | Print the version |
| `herdr --help` | Full command list for your build |
| `herdr --default-config` | Print the commented default config |
| `herdr --skill` | Print the agent skill file |
| `herdr update` | Update an installer-managed install |
| `herdr channel show` / `set <stable\|preview>` | Update channel |
| `herdr completion <shell>` | Generate shell completions |

### Sessions and server

| Command | Description |
|---------|-------------|
| `herdr session list` | List sessions |
| `herdr session attach <name>` | Attach to a named session |
| `herdr session stop <name>` | Stop a session (exits its programs) |
| `herdr session delete <name>` | Delete a stopped session |
| `herdr server stop` | Stop the default session's server |
| `herdr server reload-config` | Reload `config.toml` |

### Workspaces, tabs, worktrees

| Command | Description |
|---------|-------------|
| `herdr workspace list` | List workspaces |
| `herdr workspace create [--cwd P] [--label T] [--focus]` | Create one |
| `herdr workspace rename <id> <label>` | Rename |
| `herdr workspace close <id>` | Close (leaves files alone) |
| `herdr tab list [--workspace <id>]` | List tabs |
| `herdr tab create [--label T]` | Create a tab |
| `herdr tab rename <id> <label>` / `tab close <id>` | Rename / close |
| `herdr worktree list` | List worktrees for a repo |
| `herdr worktree create --branch <name>` | Create a worktree workspace |
| `herdr worktree open --branch <name>` | Open an existing worktree |
| `herdr worktree remove --workspace <id>` | Delete the checkout |

### Panes

| Command | Description |
|---------|-------------|
| `herdr pane list` / `pane get <id>` / `pane current` | Inspect panes |
| `herdr pane split <id> --direction right\|down` | Split |
| `herdr pane focus --direction left\|right\|up\|down` | Move focus |
| `herdr pane resize --direction <d> [--amount F]` | Resize |
| `herdr pane zoom <id> --toggle` | Zoom |
| `herdr pane rename <id> <label>` / `pane close <id>` | Rename / close |
| `herdr pane read <id> [--source S] [--lines N]` | Read output |
| `herdr pane run <id> <command>` | Run a command (submits Enter) |
| `herdr pane send-text <id> <text>` | Type text, no Enter |
| `herdr pane send-keys <id> <key>...` | Send key presses |
| `herdr pane wait-output <id> --match T \| --regex P` | Wait for output |
| `herdr pane layout` | Show the split tree and geometry |

### Agents

| Command | Description |
|---------|-------------|
| `herdr agent list` / `agent get <target>` | Inspect agents |
| `herdr agent read <target> [--source S] [--lines N]` | Read output |
| `herdr agent prompt <target> <text> [--wait] [--until S]` | Submit a prompt |
| `herdr agent wait <target> [--until S] [--timeout MS]` | Wait for a state |
| `herdr agent send-keys <target> <key>...` | Send keys |
| `herdr agent focus <target>` / `agent attach <target>` | Focus / attach |
| `herdr agent start <name> --kind K --pane P` | Launch an agent in a pane |
| `herdr agent rename <target> <name>` | Name a running agent |
| `herdr agent explain <target>` | Explain the detected state |

### Integrations and notifications

| Command | Description |
|---------|-------------|
| `herdr integration status` | Which integrations are installed |
| `herdr integration install <agent>` | Install one |
| `herdr integration uninstall <agent>` | Remove one |
| `herdr notification show <title> [--body T]` | Show a notification |

---

## Keybinding Reference

Every one of these is `Ctrl+b`, released, then the listed key. All of them are configurable under `[keys]`.

### Panes

| Action | Key |
|--------|-----|
| Split right | `v` |
| Split down | `-` |
| Move left / down / up / right | `h` / `j` / `k` / `l` |
| Cycle to next pane | `Tab` |
| Cycle to previous pane | `Shift+Tab` |
| Swap panes | `Shift+H` / `Shift+J` / `Shift+K` / `Shift+L` |
| Zoom the focused pane | `z` |
| Resize mode | `r` |
| Rename pane | `Shift+P` |
| Close pane | `x` |
| Copy mode | `[` |
| Edit scrollback in your editor | `e` |

### Tabs

| Action | Key |
|--------|-----|
| New tab | `c` |
| Next / previous tab | `n` / `p` |
| Jump to tab 1-9 | `1`..`9` |
| Rename tab | `Shift+T` |
| Close tab | `Shift+X` |

### Workspaces

| Action | Key |
|--------|-----|
| Workspace picker | `w` |
| Go-to picker | `g` |
| New workspace | `Shift+N` |
| Rename workspace | `Shift+W` |
| Close workspace | `Shift+D` |
| New git worktree | `Shift+G` |

### Session and UI

| Action | Key |
|--------|-----|
| Show all keybindings | `?` |
| Settings | `s` |
| Toggle the sidebar | `b` |
| Reload config | `Shift+R` |
| Open notification target | `o` |
| Detach (leave everything running) | `q` |

When in doubt, press `Ctrl+b` then `?`. That list comes from your running build and is always correct.

---

## Troubleshooting

### "command not found: herdr"

Restart your terminal so it reloads `PATH`. If that does not fix it, check that the install directory is on your `PATH`. For package-manager installs, update through that package manager.

### A keybinding does nothing

Your operating system or your outer terminal ate the key before Herdr could see it. This is by far the most common cause.

- Press `Ctrl+b` then `?` to confirm the binding actually exists in your build.
- Free the chord in your terminal's settings, or pick a different one in `[keys]`.
- `ctrl+alt` chords are the most reliable family, with the exceptions listed in [Configuration](#configuration).

### Ctrl+b does not page up in Vim any more

Working as designed: `Ctrl+b` is Herdr's prefix, so Herdr takes it first. See [The Prefix Key](#the-prefix-key) for the three ways out. `Ctrl+u` in Vim is the easiest.

### I updated Herdr but it still reports the old version

Installing a new binary does not always replace a running server. Check with `herdr status`, then:

```bash
herdr server stop
herdr
```

That exits the programs in the panes, so park your work first. For a named session, use `herdr session stop <name>`.

### My agent shows the wrong state, or none at all

```bash
herdr agent list                    # what does Herdr see?
herdr agent explain <target>        # why did it decide that?
herdr integration status            # is an integration available?
```

`unknown` means Herdr cannot classify the agent confidently. It does **not** mean the work succeeded.

### `agent start` returned `agent_not_ready`

Usually the agent opened a prompt during startup - a trust dialog, a login, a model picker - so Herdr reported `blocked` and stopped waiting rather than hanging forever. The agent is still running. Read the screen and answer it:

```bash
herdr agent read <name> --source visible
herdr agent send-keys <name> down enter
herdr agent wait <name> --until idle
```

### `agent start` says the pane is not available

`agent start` requires a pane sitting at its interactive shell prompt, with nothing else in the foreground. Quit whatever is running in that pane, or split a fresh one:

```bash
herdr pane split --current --direction right --no-focus
```

### Enter, Tab, or Backspace fires twice

An old terminal bug, not a Herdr bug. Update your terminal: kitty 0.33.0+, foot 1.20.0+, or Alacritty 0.15.0+. This bites people on long-term-support Linux distributions with old terminal packages.

### Herdr will not launch from inside a Herdr pane

Deliberate. Nesting is blocked by default. If you are already inside Herdr, you are already attached. `echo $HERDR_ENV` prints `1` when you are inside a pane.

### Where are the logs?

```
~/.config/herdr/herdr.log
~/.config/herdr/herdr-client.log
~/.config/herdr/herdr-server.log
```

Named sessions log under `sessions/<name>/` in the same directory. For more detail, set `HERDR_LOG=herdr=debug` before starting.

### I closed a pane and lost something

Closing a pane exits its program. Herdr does not keep a graveyard. Two habits prevent the problem: give agents real work through `agent prompt` so their output lives in the agent's own transcript, and use `Ctrl+b e` to open a pane's scrollback in your editor before you close it.

---

## Next Steps

- Spend a week using only the mouse plus `Ctrl+b q` to detach. Add keybindings when you notice yourself reaching for the same menu repeatedly.
- Press `Ctrl+b` then `?` and read the list once. It is short.
- Run `herdr --default-config` and skim it. You will discover options you did not know you wanted.
- Try one git worktree on a real repo. It is the feature people underestimate most.
- Install `herdr --skill` into your coding agent and let it start driving Herdr for you.

**Official documentation:**

- Docs home: <https://herdr.dev/docs/>
- Concepts: <https://herdr.dev/docs/concepts/>
- Keyboard: <https://herdr.dev/docs/keyboard/>
- CLI reference: <https://herdr.dev/docs/cli-reference/>
- Agent automation: <https://herdr.dev/docs/agent-automation/>
- Configuration: <https://herdr.dev/docs/configuration/>
- Troubleshooting: <https://herdr.dev/docs/troubleshooting/>

If you also use NeoVim, see the [NeoVim and LazyVim guide](neovim-lazyvim-guide.md) - and read its note about the `Ctrl+b` collision before you get confused by it.
