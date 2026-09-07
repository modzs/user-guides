# User Guides

A collection of comprehensive beginner guides for developer tools and terminal utilities. These guides are designed for people completely new to each tool, with step-by-step instructions, reference tables, and troubleshooting sections.

## Available Guides

- **[NeoVim and LazyVim Guide](neovim-lazyvim-guide.md)** - Getting started with the NeoVim text editor and the LazyVim configuration
  - What to check and install *before* you start, including the version trap that stops most beginners
  - Installation on macOS, Ubuntu/Debian, and Arch/Omarchy Linux
  - Modes, navigation, and everyday editing
  - LazyVim's leader-key commands, splits, and code navigation
  - Settings, configuration, and how to update it safely

- **[Herdr Guide](herdr-guide.md)** - Getting started with Herdr, a terminal workspace manager for AI coding agents
  - Installation on Linux, macOS, and Windows
  - Sessions, workspaces, tabs, and panes, and what each one is for
  - The sidebar, the mouse, and the prefix key
  - Git worktree workspaces
  - Running, monitoring, and scripting AI coding agents
  - Configuration, workflow patterns, and troubleshooting

> **If you plan to read both:** `Ctrl+b` means two different things in these two tools - page-up in NeoVim, and the prefix key in Herdr. Inside a Herdr pane, Herdr wins. Both guides explain what to do about it; it catches everyone once.

## Getting Started

Just want to read the guides? You do not need git, a GitHub account, or any of the material further down this page.

### Read them in your browser

Click either guide in the list above. GitHub renders them, and that is the whole process.

### Read them offline

If you would rather have a local copy:

```bash
git clone https://github.com/modzs/user-guides.git
cd user-guides
```

Then open whichever guide you want:

```bash
less neovim-lazyvim-guide.md     # read in the terminal
xdg-open herdr-guide.md          # open in your default app (Linux)
open herdr-guide.md              # open in your default app (macOS)
```

To pick up later changes:

```bash
git pull origin master
```

That is everything most readers need. The rest of this page is for people who want to keep their own customized copy or send improvements back.

---

## Contributing and Customizing

This section is entirely optional. Skip it unless you want to change these guides or add your own.

### Why fork?

Forking creates your own copy of this repository under your GitHub account. You can change it freely without affecting the original. It is worth doing if you want to:

- Add notes specific to your setup
- Customize the guides for your team
- Add sections for other tools you use
- Contribute improvements back to the original project

### Fork and clone

1. **Create a GitHub account** if you do not have one, at [github.com](https://github.com).

2. **Fork the repository:**
   - Go to <https://github.com/modzs/user-guides>
   - Click the **Fork** button in the top right
   - Choose where to fork it, usually your personal account
   - GitHub creates your copy at `https://github.com/your-username/user-guides`

3. **Clone your fork:**
   ```bash
   git clone https://github.com/your-username/user-guides.git
   cd user-guides
   ```
   Replace `your-username` with your actual GitHub username.

4. **Add the original repository as `upstream`:**
   This lets you pull in later changes from the original while keeping your own:
   ```bash
   git remote add upstream https://github.com/modzs/user-guides.git
   ```

5. **Check your remotes:**
   ```bash
   git remote -v
   ```
   You should see:
   ```
   origin    https://github.com/your-username/user-guides.git (fetch)
   origin    https://github.com/your-username/user-guides.git (push)
   upstream  https://github.com/modzs/user-guides.git (fetch)
   upstream  https://github.com/modzs/user-guides.git (push)
   ```

### Making changes

1. **Create a branch:**
   ```bash
   git checkout -b my-customizations
   ```

2. **Edit the guides** in your text editor and save.

3. **Commit:**
   ```bash
   git add neovim-lazyvim-guide.md      # or whichever file you edited
   git commit -m "Add custom setup notes for my environment"
   ```

4. **Push to your fork:**
   ```bash
   git push origin my-customizations
   ```

#### Example: adding your own notes

To add a "My Setup" section to the NeoVim guide, open `neovim-lazyvim-guide.md`, scroll to the end, and add:

```markdown
## My Custom Setup

### My Environment
- OS: Ubuntu 22.04
- Shell: zsh
- Custom plugins: ...
```

Then save and commit as above.

### Pulling in updates from the original

```bash
git fetch upstream
git checkout master
git merge upstream/master
git push origin master
```

### Sending your improvements back

If you have made a change others would benefit from, open a pull request.

1. **Push your branch to your fork** as shown above.

2. **Start the pull request:**
   - Go to your fork on GitHub: `https://github.com/your-username/user-guides`
   - Click the **Pull requests** tab, then **New pull request**

3. **Point it at the right places.** GitHub shows a "Comparing changes" page with two dropdowns. Check both:
   - **base repository:** `modzs/user-guides`, **base:** `master` - where your change is going
   - **head repository:** `your-username/user-guides`, **compare:** `my-customizations` - where it is coming from

   If you only see one repository and no way to pick another, click the **compare across forks** link. GitHub hides the base/head repository dropdowns until you do, and this is the step people miss.

4. **Review the diff** shown underneath. It should contain your changes and nothing else.

5. **Click "Create pull request".** This opens a form; it does not submit anything yet.

6. **Write a title and description** explaining what you changed and why.

7. **Click "Create pull request" again** to actually submit it.

The repository maintainer will review your changes and either merge them or leave feedback.

---

## Common Git Commands Reference

### Cloning

```bash
# Clone the original repository (read-only)
git clone https://github.com/modzs/user-guides.git

# Clone your fork (your copy)
git clone https://github.com/your-username/user-guides.git
```

### Working with Branches

```bash
# Create a new branch
git checkout -b branch-name

# Switch to an existing branch
git checkout branch-name

# List all branches
git branch -a

# Delete a branch
git branch -d branch-name
```

### Making Changes

```bash
# See what files have changed
git status

# Add a file to be committed
git add filename

# Add all changed files
git add .

# Commit your changes
git commit -m "Description of what you changed"

# Push changes to your fork
git push origin branch-name

# Pull latest changes from upstream
git fetch upstream
git merge upstream/master
```

### Viewing History

```bash
# See recent commits
git log --oneline

# See changes in a specific file
git log filename

# See differences in unstaged changes
git diff
```

## Troubleshooting

### "fatal: not a git repository"

**Problem:** You're trying to use git commands outside of a git repository.

**Solution:**
```bash
cd user-guides  # Navigate into the repository
```

### "Permission denied (publickey)"

**Problem:** SSH authentication failing when pushing to GitHub.

**Solution:** Use HTTPS instead of SSH:
```bash
git remote set-url origin https://github.com/your-username/user-guides.git
```

### "Your branch is ahead of 'origin/master' by X commits"

**Problem:** You've made local commits that haven't been pushed.

**Solution:**
```bash
git push origin branch-name
```

### "Merge conflict"

**Problem:** Git can't automatically merge changes from upstream.

**Solution:**
1. Open the conflicted file in your editor
2. Look for `<<<<<<<`, `=======`, `>>>>>>>`
3. Manually resolve the conflicts
4. Save the file
5. Commit: `git commit -m "Resolve merge conflicts"`

### Can't Remember the URL

**Check your remotes:**
```bash
git remote -v
```

This shows both your fork (origin) and the original repository (upstream).

## Guide Maintenance

These guides are updated when the tools they cover change or when someone reports a problem. There is no release schedule, so check the [commit history](https://github.com/modzs/user-guides/commits/master) if you want to know how current a guide is.

Both guides also state which version of the tool they were checked against. **Where a guide and your own machine disagree, believe your machine** - `herdr --help` and `nvim --version` are the authorities, not this repository.

- **Check for updates:** `git pull origin master` (if cloned) or `git fetch upstream` (if forked)
- **Report a problem:** Open an issue on GitHub. Corrections are especially welcome.
- **Suggest improvements:** Fork and submit a pull request

## License

These guides are MIT licensed - see [LICENSE](LICENSE). They are provided as-is for
educational purposes, so feel free to use, modify, and share them.

## Support

For questions or issues:
1. Check the **Troubleshooting** section in the relevant guide
2. Review this README's troubleshooting section, which covers git and GitHub problems
3. For a problem with the tool itself rather than the guide, check that tool's own documentation:
   - Herdr: <https://herdr.dev/docs/>
   - NeoVim: `:help` inside the editor, or <https://neovim.io/doc/>
   - LazyVim: <https://lazyvim.org>
4. Open an issue on the GitHub repository

---

**Happy learning!** Start with the guide you need most. Come back to this README when you want help with git or GitHub.
