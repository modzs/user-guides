# Firstmate Guide for Beginners

A complete guide to installing and running Firstmate on Omarchy Linux, with Herdr as the
terminal backend.

**Platform Support:** This guide covers Omarchy Linux specifically, because every
`omarchy ...` command and every keybinding path in it is Omarchy's. Firstmate itself runs
on macOS and other Linux distributions; only the installation and desktop-configuration
steps here are Omarchy-only.

**Version note:** Everything in this guide was checked against **Herdr 0.8.2** (Omarchy's
packaged build), **Omarchy's shipped Hyprland configuration**, and the **Firstmate
repository as of commit `f3b7e74`, September 2026**. Firstmate moves fast and Herdr moves
faster. If a command here disagrees with your machine, your machine is right. The
authorities are `herdr --help`, `herdr --version`, and the `AGENTS.md` and `docs/` files
inside your own Firstmate clone.

## Table of Contents
1. [What Firstmate Actually Is](#what-firstmate-actually-is)
2. [Key Concepts](#key-concepts)
3. [Why Pick the Herdr Backend](#why-pick-the-herdr-backend)
4. [Before You Install](#before-you-install)
5. [Installation](#installation)
6. [Selecting the Herdr Backend](#selecting-the-herdr-backend)
7. [Starting Herdr and Launching Your First Mate](#starting-herdr-and-launching-your-first-mate)
8. [Giving Your First Order](#giving-your-first-order)
9. [Watching the Crew](#watching-the-crew)
10. [Omarchy Desktop Configuration](#omarchy-desktop-configuration)
11. [Keeping Your Fork Up to Date](#keeping-your-fork-up-to-date)
12. [Troubleshooting](#troubleshooting)
13. [Next Steps](#next-steps)

---

## What Firstmate Actually Is

Firstmate is an **agent distro**: a portable directory of instructions, skills, helper
scripts, policies, and state conventions that turns a general-purpose coding agent into a
specialized one.

This is the single most important thing to understand before you start:

> **There is no app to install and no `firstmate` binary.** The cloned repository *is* the
> product. You launch an ordinary coding agent with the repo as its working directory, the
> agent reads `AGENTS.md`, and from that point on it behaves as your first mate.

Firstmate's own README puts it plainly: it "is not a model, not a harness, not a skill, not
an MCP server, and not a CLI." If you finish the installation expecting a program to run
and cannot find one, nothing has gone wrong.

You talk to exactly one agent, the first mate. It spawns autonomous crewmates, gives each
a clean git worktree, supervises them, and hands back finished pull requests, approved
merges, or standalone investigation reports.

- **You** give one order to the first mate
- **First mate** breaks the work into parallel tasks
- **Crewmates** work independently, each in its own worktree and its own visible terminal
- **First mate** supervises, consolidates, and reports back

Upstream repository: <https://github.com/kunchenguid/firstmate>

### Which coding agent runs it

Firstmate's README lists seven verified primary harnesses: **Claude Code, Grok, Pi,
`pi-signed`, Codex, OpenCode, and Cursor Agent CLI**. Of those, Claude Code, Grok, and Pi
are named as equal co-primary recommendations.

This guide uses **Claude Code** throughout, because it needs the least setup: you launch
it with a bare `claude` and nothing else. Grok and Cursor Agent CLI need `--trust` so their
project hooks load, and Pi needs you to approve a trust prompt once per clone. If you use
one of those, read "Recommended harnesses" in Firstmate's README before you launch, then
follow the rest of this guide unchanged.

## Key Concepts

| Concept | Meaning |
|---|---|
| **Captain** | You. The first mate escalates real decisions to you, and only real decisions. |
| **First mate** | Your primary agent session. The only one you talk to. |
| **Crewmates** | Secondary agents spawned for parallel tasks. |
| **Secondmates** | Optional persistent second mates that run from their own isolated Firstmate homes, either on this machine or on an SSH-reachable host. |
| **Ship tasks** | Tasks that deliver authorized changes: commits, pull requests, merges. |
| **Scout tasks** | Research-only tasks that leave a standalone investigation report and make no commits. |
| **Worktrees** | Clean per-task git worktrees, provided by [treehouse](https://github.com/kunchenguid/treehouse), so parallel work on one repository never collides. |
| **Backend** | Where crewmate terminals get created. `tmux` is the fallback default; `herdr` is what this guide sets up. |
| **Herdr** | A terminal workspace manager built for AI coding agents. It organizes terminals into workspaces, tabs, and panes, and tracks agent state natively. |
| **Presentation spaces** | A Herdr-only visual projection that puts each new crewmate or scout in its own disposable single-task workspace. |

If Herdr itself is new to you, read the [Herdr Guide](herdr-guide.md) in this repository
first. It covers sessions, workspaces, tabs, panes, and the prefix key from scratch. This
guide assumes only that you can start Herdr and open a pane.

## Why Pick the Herdr Backend

Firstmate can create crewmate terminals in several backends. Two of them are properly
supported: `tmux` is the verified reference backend, and `herdr` has its own required CI
lane. (`zellij`, `orca`, and `cmux` are experimental with no dedicated real-backend CI
lane.) Pick Herdr when you want Herdr's native per-pane agent state instead of Firstmate
working the state out for itself.

| | tmux | Herdr |
|---|---|---|
| Agent state | Firstmate classifies it itself, from a screen capture plus foreground-process probes | Firstmate reads Herdr's own per-pane state: `idle`, `working`, `blocked`, `done`, `unknown` |
| Status updates | Polling | Polling, plus optional push events when Herdr protocol 16 and `python3` are both available; polling remains the permanent fallback either way |
| Task layout | One tmux window per task | Herdr workspaces and tabs, one `fm-<id>` tab per task, plus presentation spaces |
| Standing in Firstmate | Verified reference backend, and the default when nothing else is selected | Its own required CI lane |

Firstmate requires **Herdr protocol 14 or newer**. The Herdr 0.8.2 that Omarchy ships
speaks protocol 20, so any current Omarchy install clears that bar comfortably. You can
check your own build's protocol number with:

```bash
herdr api schema --json | head -5
```

### Herdr's active limits, honestly stated

Firstmate's `docs/herdr-backend.md` keeps a list of what the Herdr backend does not do
yet. At the time of writing:

- Presentation-space *ordering* needs protocol 16 and Python, and is best-effort only.
- Workspace and tab labels are mutable and can collide. They are never placement authority.
- A first mate running *outside* Herdr cannot resolve a launcher workspace, so a colliding
  home label refuses new spawns until you clear the collision.
- Mid-session secondmate agent-process liveness is not implemented.
- Only tmux and Herdr can host the away-mode supervisor terminal.

Read that file in your own clone rather than trusting this list; it is the authority and
it changes.

---

## Before You Install

Firstmate needs two sets of tools: a universal set every install needs, and a small extra
set that depends on which backend you chose.

**Universal:** node, git, `gh` (authenticated with `gh auth login`), `no-mistakes`, plus
the `gh-axi`, `chrome-devtools-axi`, `lavish-axi`, `tasks-axi`, and `quota-axi` helper
CLIs.

**Extra, for the Herdr backend specifically:** `herdr` itself, `jq` (the adapter parses
Herdr's JSON output), and `treehouse` (the worktree provider). `python3` is optional, and
only buys you protocol-16 presentation-space ordering and push-event subscription.

You do not have to install all of that by hand before you start, and you should not try.
**On session start the first mate detects what is missing or too old and prints each
problem with either an exact install command or manual instructions.** It installs the
tools it can install automatically only after you say go. The practical approach is:

1. Install the handful of things below.
2. Launch the first mate.
3. Do what its toolchain report tells you.

Check what you already have:

```bash
git --version
gh --version
node --version
jq --version
claude --version
python3 --version      # optional
```

Install anything missing from that list:

```bash
omarchy pkg add git github-cli nodejs jq
omarchy pkg add python                    # only if you want the optional features
```

Claude Code is deliberately not in that `omarchy pkg add` line. Step 2 below installs it
through Omarchy's own agent command, which is the route this guide uses. Using Claude Code
also requires a Claude account, and Claude Code has to be signed in before the first mate
can do anything at all. Step 2 covers that too.

Note that on Arch, and therefore on Omarchy, the Python 3 package is called `python`, not
`python3`. The binary it puts on your `PATH` is still `python3`.

`omarchy pkg add` installs Arch packages only if they are missing, so it is safe to run
even when some of them are already there. If you manage node through a version manager
such as `mise` or `nvm`, keep doing that and leave `nodejs` out of the command above.

## Installation

### Step 1: Install Herdr

```bash
omarchy pkg add herdr
```

Herdr ships in Omarchy's own `[omarchy]` pacman repository, so this is a plain repository
install with no AUR build required. Verify it:

```bash
herdr --version
```

You should see `herdr 0.8.2` or newer.

> **Updating Herdr on Omarchy:** use `omarchy update`, which updates Omarchy and your
> system packages. Do **not** use `herdr update` here. Herdr's own documentation is
> explicit that `herdr update` is only for installs managed by Herdr's own installer;
> package-manager installs are updated through the package manager. The [Herdr
> Guide](herdr-guide.md) in this repository covers the other install methods and their
> update paths.

### Step 2: Install Claude Code

```bash
omarchy default agent claude
```

**That command opens a separate terminal window, and the work happens in that window.**
Your own prompt returns immediately. When Claude Code is not installed yet, Omarchy re-runs
itself in a floating window and does the `mise` install there, so closing that window early
aborts the install and leaves you with no `claude` and no `~/.claude`. Stay in the new
window until you are finished.

The command itself does three things: it installs Claude Code through `mise` if `mise where
claude` finds nothing, writes `claude` into `~/.config/omarchy/defaults/agent` as Omarchy's
default coding agent, and then launches it. (`mise` resolves the name `claude` to
`aqua:anthropics/claude-code`.)

Claude Code requires a Claude account and does nothing until it is signed in, so complete
the sign-in on this first launch, then quit. That first launch is also what creates
`~/.claude`, which the next step needs.

This guide does not reproduce the sign-in procedure, because that flow changes and it was
not verified here. Anthropic documents it under "Step 2: Log in to your account" in the
[Claude Code quickstart](https://code.claude.com/docs/en/quickstart). From the shell,
`claude auth status` shows whether you are signed in and `claude auth login` signs you in.

Note that Omarchy launches the agent as `claude --permission-mode auto`. That is why this
guide starts the first mate with a plain `claude` from inside the repo later on, rather
than through Omarchy's agent launcher.

If you already have Claude Code installed and signed in by some other route, you can skip
this step. All the next step requires is that your Claude configuration directory exists.

### Step 3: Install the Herdr integration for Claude Code

**Your Claude configuration directory has to exist before you run this install.** Herdr
checks for the directory, not for the `claude` binary, so if Step 2 has not created
`~/.claude` yet, create it yourself:

```bash
mkdir -p ~/.claude
```

If you have set `CLAUDE_CONFIG_DIR`, that is the directory Herdr checks instead. Then:

```bash
herdr integration install claude
```

Confirm it took:

```bash
herdr integration status
```

The `claude:` line should say `current`, followed by a version number and the path to the
hook. On the machine this guide was checked on it printed:

```
claude: current (v8) (/home/you/.claude/hooks/herdr-agent-state.sh)
```

Your version number will differ as Herdr revises the hook; `current` is the part that
matters. `herdr integration status` lists every agent Herdr knows about, so expect a long
list with `not installed` against the agents you do not use.

**Be clear about what this does and does not do.** Herdr classifies Claude Code's state
from its screen output whether or not this integration is installed. The hook adds
*session identity*: it reports Claude Code's own session reference to Herdr, which is what
lets Herdr resume your Claude panes after a server restart. It is genuinely useful, and it
is not what makes agent-state tracking work.

Herdr's own documentation splits integrations into two kinds. Pi, OMP, Kimi Code CLI,
OpenCode, Kilo Code CLI, and MastraCode get *lifecycle authority*, where their hooks author
the state directly. Claude Code, Codex, Copilot, Cursor Agent CLI and the rest get
*session identity* only. Claude Code is in the second group.

Installing writes `hooks/herdr-agent-state.sh` and adds Herdr entries to your
`settings.json`. Uninstalling removes both.

### Step 4: Authenticate with GitHub

```bash
gh auth login
```

Choose SSH if you plan to push, since the clone commands below use an SSH remote. HTTPS
works too if you let `gh` manage your credentials.

### Step 5: Fork Firstmate (optional)

Forking is optional. Fork if you want to keep local customizations under version control.
Otherwise clone upstream directly and skip to Step 6.

1. Open <https://github.com/kunchenguid/firstmate>
2. Click **Fork**
3. Select your account
4. Your fork lands at `https://github.com/YOUR_USERNAME/firstmate`

### Step 6: Clone

```bash
mkdir -p ~/Projects
cd ~/Projects

# Your fork:
git clone git@github.com:YOUR_USERNAME/firstmate
# Or upstream directly:
# git clone https://github.com/kunchenguid/firstmate

cd firstmate
```

**The directory name is case sensitive.** This guide uses `~/Projects` with a capital P.
If you type `~/projects` the `cd` fails, and any command chained after `&&` never runs.
Worse, a chain joined with `;` launches your agent from the wrong directory, where there is
no `AGENTS.md` to read, and you get an ordinary coding agent that has never heard of
Firstmate. That single typo is the most common reason Firstmate appears to "do nothing".

Note that `projects/` *inside* the repo is a completely different thing. It is gitignored,
and it is where Firstmate clones the projects you ask it to work on.

### Step 7: Add the upstream remote

Only needed if you forked in Step 5.

```bash
git remote add upstream https://github.com/kunchenguid/firstmate
git remote -v
```

`git remote -v` prints a `(fetch)` and a `(push)` line for every remote, including ones you
cannot actually push to:

```
origin    git@github.com:YOUR_USERNAME/firstmate (fetch)
origin    git@github.com:YOUR_USERNAME/firstmate (push)
upstream  https://github.com/kunchenguid/firstmate (fetch)
upstream  https://github.com/kunchenguid/firstmate (push)
```

`origin` is your fork and you can push to it. `upstream` is the original repository and you
normally only fetch from it.

## Selecting the Herdr Backend

This step only records *which backend Firstmate should use when it spawns crewmates*. It
does not start Herdr, and it does not start Firstmate.

Firstmate resolves the backend for each new spawn in this exact order, first match wins:

1. An explicit `--backend` flag, authorized for that one task
2. `FM_BACKEND` in the environment
3. The first non-empty line of the local, gitignored `config/backend` file
4. Auto-detection from `$TMUX`, `HERDR_ENV=1`, or cmux runtime signals
5. Default: `tmux`

Pick one of these four options.

**Option A: persistent, per-clone (recommended)**

```bash
mkdir -p config
echo herdr > config/backend
```

`config/` is gitignored in full, so this never fights with upstream updates.

**Option B: one launch only**

```bash
FM_BACKEND=herdr claude
```

**Option C: rely on auto-detection**

If you launch your agent from inside a Herdr pane, Firstmate auto-detects Herdr and prints
a notice on stderr naming `config/backend` and `--backend tmux` as the ways to opt out.

Two things worth knowing about auto-detection:

- `HERDR_ENV=1` is exported **by Herdr** into every pane it manages, alongside
  `HERDR_PANE_ID` and `HERDR_SOCKET_PATH`. You do not set it yourself, and setting it by
  hand outside Herdr does not make auto-detection work.
- A tmux pane nested inside Herdr resolves to **tmux**, because the innermost multiplexer
  wins.

**Option D: just tell the first mate**

Ask it in chat to use the Herdr backend.

### Presentation spaces

By default, on Herdr 0.8.0 and newer, each new crewmate or scout is placed in its own
disposable single-task workspace. You do not need to configure anything to get this.

The version floor exists for a specific reason. Projecting each task into its own workspace
makes every task cleanup a workspace-emptying removal, and that is exactly the removal
shape that Herdr's pre-0.8.0 focus defect touches. On 0.8.0 and newer, every
workspace-removal primitive preserves focus, so the projection is safe to turn on by
default.

Two details matter if you are on an older Herdr:

- Below the floor, an install that has never configured this creates each `fm-<id>` task
  tab directly in the first mate's own workspace instead, and warns once per detected
  release. Upgrading Herdr is the fix.
- An explicit opt-in is honored *below* the floor too, so a home that deliberately turned
  the projection on is never silently downgraded.

The switch is a local, gitignored file:

```bash
echo off > config/herdr-presentation-spaces   # opt out
echo on  > config/herdr-presentation-spaces   # force on, even below the floor
```

Values are compared with whitespace stripped and case ignored. An empty file counts as a
deliberate opt-in, which is the historical form of the setting. An unrecognized value warns
and falls back to the default rather than failing your spawn over a purely visual setting.

## Starting Herdr and Launching Your First Mate

### Step 1: Start a Herdr session

Herdr is a separate program. Start it before launching your agent.

Omarchy already ships a keybinding for this: **`SUPER + CTRL + RETURN`** launches or
attaches the persistent Herdr session. (`SUPER + CTRL + K` shows Herdr's own keybindings.)
The equivalent commands are:

```bash
omarchy launch terminal herdr
```

or plain `herdr` in any terminal, which launches or attaches the persistent session and
starts the background server if it is not already running. You do not need to run a daemon
by hand. There is no `herdr daemon` command; the headless server subcommand is
`herdr server`, and the client starts it for you.

### Step 2: Launch Claude Code inside the repo

From a pane **inside your Herdr session**:

```bash
cd ~/Projects/firstmate
claude
```

That is the whole launch. If you set `config/backend` earlier you do not need to prefix
anything; otherwise use `FM_BACKEND=herdr claude`.

The first time you launch Claude Code in a folder it has never seen, it shows a workspace
trust dialog. Answer it once and it will not ask again for that folder. (Firstmate
pre-registers this trust for the task worktrees it creates, so your crewmates never hit it.)

### What actually happens, and what does not

This is the part that surprises people, so it is worth being blunt.

**`claude` starts Claude Code and nothing else visible happens.** There is no Firstmate
splash screen, no backend verification banner, no automatic Herdr layout. What makes the
session a first mate is simply that the agent reads `AGENTS.md` from its working directory.

`FM_BACKEND=herdr` in particular **does not launch Herdr.** It only records where crewmate
terminals should be created later. If Herdr is not already running, that variable changes
nothing you can see.

The one thing you should see is the toolchain report described in [Before You
Install](#before-you-install): the first mate checks its tools at session start and tells
you what is missing.

Nothing Herdr-related happens until you give the first mate an actual task. Only then does
it acquire worktrees and create Herdr workspaces and tabs.

If you ran the launch command and concluded "this just started Claude, it did not start
Herdr or Firstmate", then you saw the expected behavior. Check two things:

1. That you are really in `~/Projects/firstmate` (capital P), so `AGENTS.md` was loaded.
2. That you actually gave it an order.

## Giving Your First Order

You just talk to it in plain English:

```
fix the flaky login test in my project xyz and add dark mode
```

**There is no command syntax to learn.** Firstmate's README writes its examples as
`ahoy! look at my github project xyz, then fix the flaky login test and add dark mode`,
and that reads well, but `ahoy` is nautical flavor rather than a required prefix or a
parsed keyword. Nothing in Firstmate parses it. Dropping it changes nothing.

> Do not confuse that with **`/ahoy`**, which is a real and completely different thing: a
> user-invocable skill that recaps what has happened this session and walks you through
> decisions you have not answered yet. Use `/ahoy` to catch up, not to start work.

The first mate checks its toolchain, asks your consent before installing anything, clones
the target project under `projects/`, and spawns crewmates in the active backend. Minutes
later it reports back. Firstmate's README shows the shape of that exchange:

```
  PR ready for review, captain: https://github.com/you/xyz/pull/42
  (fix flaky login test - risk: low - CI green)

> alright merge it
```

More examples of orders:

```
scout the authentication system and summarize security issues
ship a dark mode toggle to the UI
fix three failing tests AND update docs AND refactor the auth module
```

"Ship" and "scout" name the two task shapes: a scout task leaves a standalone
investigation report and makes no commits, a ship task delivers authorized changes. Saying
the word steers which shape the first mate picks, but this is still ordinary conversation
and not a command parser. Chaining requests with `AND` is how you get parallel crewmates.

## Watching the Crew

You launched the first mate from a pane inside Herdr, so its tasks appear next to the
workspace that pane lives in. With presentation spaces on, which is the default on Herdr
0.8.0 and newer, every crewmate or scout gets its own disposable single-task workspace
holding one `fm-<id>` tab, bound to the first mate's own workspace as its placement
reference. Attach to the Herdr session and look at the workspaces around the first mate's
to watch them. Where ordering is available they sit in one contiguous block immediately
after the first mate's workspace, but ordering is best-effort and needs protocol 16 and
`python3`; without it they land wherever Herdr puts them.

Turn presentation spaces off and the `fm-<id>` tabs are created directly in the first
mate's own workspace instead. A workspace labeled `firstmate` is a third case, and not the
one this guide produces: that home-labeled workspace is maintained only when the first mate
runs *outside* Herdr and so has no launcher workspace to inherit.

You usually do not need to attach at all. Supervise from the first mate session instead:

```bash
bin/fm-peek.sh <id>                           # print the tail of a task's terminal
FM_HOME=<home> bin/fm-send.sh <id> 'message'  # steer a running task
```

`fm-peek.sh` takes an optional second argument for how many lines to print, defaulting to
40. Both scripts document their full usage in their file headers.

Firstmate creates workspaces and tabs with `--no-focus`, so spawning a crewmate does not
steal your focus. The one exception is the very first workspace in a completely empty Herdr
session, which has to become focused because no prior target exists.

**Do not name a personal workspace `firstmate` or `2ndmate-<id>`.** Those are the home
labels Firstmate uses for its own containers. Herdr does not enforce label uniqueness, so a
collision makes placement unresolvable, and Firstmate refuses to spawn rather than guessing
which workspace it meant.

---

## Omarchy Desktop Configuration

### A launcher script

Omarchy already gives you `SUPER + CTRL + RETURN` for the Herdr session itself, so the
thing worth adding is a terminal that opens straight into the Firstmate directory, for
the `git` and config work that happens outside Herdr.

Create `~/.local/bin/firstmate-terminal`:

```bash
mkdir -p ~/.local/bin
cat > ~/.local/bin/firstmate-terminal <<'EOF'
#!/bin/bash
set -euo pipefail
exec foot -D "$HOME/Projects/firstmate"
EOF
chmod +x ~/.local/bin/firstmate-terminal
```

The shebang must be the very first line of the file, which is why the heredoc above starts
immediately with `#!/bin/bash`.

`foot -D <dir>` sets the directory the terminal starts in, and `foot` is Omarchy's
terminal. If you use a different terminal, check its own `--help` for the flag that sets
the starting directory rather than guessing.

Do not export `HERDR_ENV=1` in a script like this. Herdr sets it inside the panes it
manages; setting it by hand outside Herdr only misleads Firstmate's auto-detection, and it
does not make an ordinary terminal into a Herdr pane.

### A keyboard shortcut

Omarchy configures Hyprland in **Lua**, not hyprlang. Keybindings live in
`~/.config/hypr/bindings.lua`, which loads after Omarchy's defaults, so your entries
override them.

**Check what is already bound before you choose a combination:**

```bash
omarchy menu keybindings --print
```

This matters more than you would expect. Omarchy's stock bindings are dense, and the
obvious choices are taken: `SUPER + F` is Full screen, `SUPER + SHIFT + F` is the File
manager, `SUPER + ALT + F` is Full width, and `SUPER + CTRL + F` is Tiled full screen. If
you bind a combination that is already in use without unbinding it first, your binding will
not take effect.

`SUPER + SHIFT + CTRL + F` is free on a stock Omarchy install, so the example below needs
no unbind. Open the file:

```bash
omarchy launch config editor ~/.config/hypr/bindings.lua
```

Add:

```lua
o.bind("SUPER + SHIFT + CTRL + F", "Firstmate", "firstmate-terminal")
```

The signature is `o.bind(keys, description, dispatcher, options)`. The fourth argument is
optional, and a plain string dispatcher like the one above is run as a command.

If you would rather take over a combination that is already used, unbind it first:

```lua
hl.unbind("SUPER + F")
o.bind("SUPER + F", "Firstmate", "firstmate-terminal")
```

Save, then apply and check for config errors:

```bash
hyprctl reload
hyprctl configerrors
```

`hyprctl configerrors` prints nothing when the configuration is clean. If it lists an
error, fix it before you go looking for why your key does nothing.

---

## Keeping Your Fork Up to Date

This section is only relevant if you forked in Step 5.

> **Firstmate's default branch is `main`.** That is different from this guides repository,
> whose default branch is `master`. The commands below are correct for Firstmate as
> written.

### Pull upstream fixes

```bash
git fetch upstream
git checkout main
git merge upstream/main
git push origin main
```

### Keep your customizations out of the way

Most local settings are already gitignored, so they never cause merge conflicts:

- `config/` is gitignored in full, including `config/backend` and
  `config/herdr-presentation-spaces`
- `projects/`, `state/`, `data/`, and `.env` are gitignored too

Prefer these over editing tracked files. If you must change tracked files, keep those
changes on a branch.

### Custom branch workflow

Keep `main` as a clean mirror of upstream and do your own work on a branch. The point is
that you never merge *your branch into main*. You do merge upstream into main, which is
what keeps it current.

```bash
git checkout -b my-custom-config
# make changes
git add -A
git commit -m "Add my custom Firstmate configuration"
git push origin my-custom-config
```

Refresh `main` from upstream, then bring those updates into your branch:

```bash
git checkout main
git fetch upstream
git merge upstream/main
git push origin main

git checkout my-custom-config
git merge main
```

### Resolve conflicts

```bash
git status
nvim path/to/conflicted/file
```

Omarchy ships NeoVim as `nvim`, not `vim`. If NeoVim is new to you, this repository's
[NeoVim and LazyVim Guide](neovim-lazyvim-guide.md) covers it from scratch. To use whatever
editor Omarchy is configured to open instead, run
`omarchy launch config editor path/to/conflicted/file`.

Conflict markers look like this:

```
<<<<<<< HEAD
your local changes
=======
upstream changes
>>>>>>> upstream/main
```

Keep what you want, delete all three marker lines, then:

```bash
git add path/to/conflicted/file
git commit
```

### Sync script

If you do this often, wrap it up:

```bash
cat > ~/.local/bin/sync-firstmate <<'EOF'
#!/bin/bash
set -euo pipefail
cd ~/Projects/firstmate
git fetch upstream
git checkout main
git merge upstream/main
git push origin main
echo "Firstmate fork synced with upstream"
EOF
chmod +x ~/.local/bin/sync-firstmate
```

---

## Troubleshooting

### "It just launched Claude, nothing else happened."

Expected. See [What actually happens, and what does
not](#what-actually-happens-and-what-does-not). Confirm you are in `~/Projects/firstmate`
with a capital P, and that you have actually given the first mate an order.

### Claude Code asks you to sign in instead of showing the toolchain report

Claude Code requires an account, so an install that has never been signed in stops at its
sign-in prompt before the first mate ever reads `AGENTS.md`. Complete the sign-in and
relaunch. From the shell:

```bash
claude auth status     # are you signed in?
claude auth login      # sign in
```

Anthropic documents the procedure under "Step 2: Log in to your account" in the [Claude
Code quickstart](https://code.claude.com/docs/en/quickstart); this guide does not reproduce
it.

### Herdr not found

```bash
herdr --version
omarchy pkg add herdr
```

If `herdr --version` still fails after installing, restart your terminal so it picks up the
new `PATH`.

### `claude directory not found at <path>. install claude code first`

`herdr integration install claude` prints this and writes nothing. It is checking for your
Claude configuration directory, not for the `claude` binary. Either install Claude Code
with `omarchy default agent claude` and let it launch once, which creates the directory, or,
if Claude Code is already installed, create the directory yourself:

```bash
mkdir -p ~/.claude
```

If `CLAUDE_CONFIG_DIR` is set, that is the path Herdr is reporting.

### The first mate says a tool is missing

Do what its report says. It prints either an exact install command or manual instructions
for each problem, and it installs the automatable ones only after you approve. That report
is more current than this guide.

### Agent state is stuck on `unknown`

`unknown` means Herdr sees an agent but cannot classify its state confidently. It does not
mean the task finished.

For Claude Code, state comes from Herdr's screen-manifest detection, so `unknown` usually
means the agent's current screen does not match a rule Herdr recognizes. Installing the
Claude Code integration will *not* fix this, because that integration supplies session
identity rather than state. Ask Herdr what it is seeing:

```bash
herdr agent explain <target>
```

### The wrong backend is being used

Check the resolution order from [Selecting the Herdr
Backend](#selecting-the-herdr-backend):

```bash
echo "$FM_BACKEND"
cat config/backend
echo "$TMUX"          # non-empty means tmux wins over Herdr
echo "$HERDR_ENV"     # 1 only inside a Herdr-managed pane
```

Remember that the innermost multiplexer wins: an agent in a tmux pane nested inside Herdr
resolves to tmux.

### A spawn refuses with a placement error

This happens when a first mate running *outside* Herdr cannot resolve which workspace
launched it, and the home label it would fall back to is ambiguous, because two workspaces
share the `firstmate` label or a personal workspace is named `firstmate`. Rename yours.
Firstmate refuses rather than guessing.

### Presentation spaces are not appearing

They are on by default only on Herdr 0.8.0 and newer. Check your version and the opt-out
file:

```bash
herdr --version
cat config/herdr-presentation-spaces   # "off" disables, "on" forces on
```

If you are below the floor, upgrading Herdr is the real fix; writing `on` into that file is
the override if you cannot upgrade yet.

### Cleaning up

Do not run `herdr server stop` to tidy up an active session. It stops the whole Herdr
server and every pane process under it, including work you did not mean to end.

Detach instead, or close the specific tabs you are done with. Firstmate's own guarded Herdr
lifecycle helper, `bin/fm-herdr-lab.sh`, exists for isolated verification labs and is not a
general cleanup tool for your everyday session.

---

## Next Steps

Everything below lives in your Firstmate clone. These are the files worth reading, in the
order worth reading them:

1. `README.md` - features, requirements, and quick start
2. `AGENTS.md` - the distro itself, and the authoritative behavior contract
3. `docs/herdr-backend.md` - the Herdr backend in full detail, including its current limits
4. `docs/configuration.md` - backend selection, the toolchain list, and every other knob
5. `docs/architecture.md` - how the whole system fits together
6. `docs/tmux-backend.md` - the default backend, useful for comparison

Firstmate also ships user-invocable skills: `/ahoy`, `/bearings`, `/afk`,
`/updatefirstmate`, and `/stow`. Its README's "Built-in skills" table describes what each
one does.

Herdr's own documentation is at <https://herdr.dev>. `herdr --help` is the authority on its
CLI syntax, and `herdr --skill` prints its agent-facing reference. For Herdr itself, start
with the [Herdr Guide](herdr-guide.md) in this repository.
