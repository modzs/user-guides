# Project agent memory

This file is the project's committed home for project-intrinsic agent knowledge: build, test, release, architecture, and sharp-edge notes that should travel with the code.

## What this repo is

Beginner guides for terminal tools, written for readers who have never used a modal
editor or a terminal multiplexer. Three content files: `README.md`,
`herdr-guide.md`, `neovim-lazyvim-guide.md`. No build, no tests, no CI.

## The rule that matters here

**Verify every command, flag, keybinding, config key, file path, and URL against the
actually-installed tool or fetched upstream documentation before writing it down.
Never write it from memory, and never by analogy to a similar tool.**

Verification surfaces, all cheap:

- Herdr: `herdr --help`, `herdr <subcommand> --help`, `herdr --default-config`
  (authoritative config schema and every default keybinding), `herdr --version`,
  and <https://herdr.dev/llms.txt> for the upstream docs index.
- NeoVim/LazyVim: `nvim --version`, <https://lazyvim.org> (requirements and the
  authoritative keymap list). Probing a throwaway LazyVim install under a temporary
  `XDG_CONFIG_HOME`/`XDG_DATA_HOME` settles keymap questions in about two minutes;
  note that LazyVim's core keymaps only bind on the `VeryLazy` event, so a headless
  probe must fire that event before reading them.
- Distribution package versions: the Debian, Ubuntu Launchpad, Arch, and Homebrew
  JSON APIs answer "is the packaged version new enough" directly.

If something cannot be verified, leave it out or state the limit. An honest gap is
fine; a confident invention is not.

Guides state the tool version they were checked against, and tell the reader to
believe their own machine over the guide. Keep that.

## Conventions

- The default branch is `master`, not `main`. Instructions saying `git pull origin
  master` are correct; do not "fix" them.
- Shown command output must be output actually observed, not reconstructed. Replace a
  real home directory with `/home/you/` when quoting it.
- There is a `## License` section in `README.md` but no `LICENSE` file. That gap is the
  repository owner's call, so leave both alone unless asked.

## Maintaining this file

Keep this file for knowledge useful to almost every future agent session in this project.
Do not repeat what the codebase already shows; point to the authoritative file or command instead.
Prefer rewriting or pruning existing entries over appending new ones.
When updating this file, preserve this bar for all agents and keep entries concise.
