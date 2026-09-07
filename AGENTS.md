# Project agent memory

This file is the project's committed home for project-intrinsic agent knowledge: build, test, release, architecture, and sharp-edge notes that should travel with the code.

## What this repo is

Beginner guides for terminal tools and the agent workflows built on them, written for
readers who have never used a modal editor or a terminal multiplexer. Content files:
`README.md`, `herdr-guide.md`, `neovim-lazyvim-guide.md`, `firstmate-guide.md`, plus
`LICENSE`. No build, no tests, no CI.

## The rule that matters here

**Verify every command, flag, keybinding, config key, file path, and URL against the
actually-installed tool or fetched upstream documentation before writing it down.
Never write it from memory, and never by analogy to a similar tool.**

Verification surfaces, all cheap:

- Herdr: `herdr --help`, `herdr <subcommand> --help`, `herdr --default-config`,
  `herdr --version`, and <https://herdr.dev/llms.txt> for the upstream docs index,
  whose "Config reference" JSON lists every canonical `config.toml` key with its
  type and default. A tool's own output is authoritative for what it emits, not an
  exhaustive list of what exists: `herdr --default-config` on 0.8.2 omits
  `keys.copy_mode` and the `keys.swap_pane_*` family, which the config reference
  documents. Absence from a printout is not proof a thing does not exist, so check
  the upstream reference before deleting something as invented. Verify any link you
  cite by fetching it and confirming the page contains the specific thing you cite it
  for: an HTTP 200 and a resolving anchor prove neither.
- NeoVim/LazyVim: `nvim --version`, <https://lazyvim.org> (requirements and the
  authoritative keymap list). Probing a throwaway LazyVim install under a temporary
  `XDG_CONFIG_HOME`/`XDG_DATA_HOME` settles keymap questions in about two minutes;
  note that LazyVim's core keymaps only bind on the `VeryLazy` event, so a headless
  probe must fire that event before reading them.
- Distribution package versions: the Debian, Ubuntu Launchpad, Arch, and Homebrew
  JSON APIs answer "is the packaged version new enough" directly.
- Firstmate: its own clone is the only authority. `README.md` owns the supported primary
  harnesses and requirements, `docs/configuration.md` owns backend selection order and the
  toolchain list, `docs/herdr-backend.md` owns Herdr placement, presentation spaces, push
  events and the active limits, and `bin/*.sh` headers own each script's usage. Never
  describe Firstmate from memory.
- Omarchy: `omarchy <group> --help` for command shapes, `omarchy menu keybindings --print`
  for what is already bound, and `/usr/share/omarchy/default/hypr/` for the shipped Lua
  bindings and the `o.bind`/`hl.unbind` helpers. Most obvious `SUPER` combinations are
  already taken, so check before recommending one, and check `command -v` before naming a
  tool as present (Omarchy ships `nvim`, not `vim`).

If something cannot be verified, leave it out or state the limit. An honest gap is
fine; a confident invention is not.

Guides state the tool version they were checked against, and tell the reader to
believe their own machine over the guide. Keep that.

## Conventions

- The default branch is `master`, not `main`. Instructions saying `git pull origin
  master` are correct; do not "fix" them.
- Shown command output must be output actually observed, not reconstructed. Replace a
  real home directory with `/home/you/` when quoting it.
- The repository is MIT licensed: a `LICENSE` file exists and `README.md`'s `## License`
  section links to it. Keep those two in agreement.

## Maintaining this file

Keep this file for knowledge useful to almost every future agent session in this project.
Do not repeat what the codebase already shows; point to the authoritative file or command instead.
Prefer rewriting or pruning existing entries over appending new ones.
When updating this file, preserve this bar for all agents and keep entries concise.
