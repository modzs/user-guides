# User Guides

A collection of comprehensive beginner guides for developer tools and terminal utilities. These guides are designed for people completely new to each tool, with step-by-step instructions, reference tables, and troubleshooting sections.

## Available Guides

- **[GitHub and GitHub CLI Guide](github-cli-guide.md)** - Complete beginner's guide to GitHub and using the GitHub CLI
  - Key concepts with plain English analogies (push, pull, merge, branches, pull requests)
  - Installation of Git and GitHub CLI
  - Creating and cloning repositories
  - Working with branches and commits
  - Pull requests and code review
  - Common workflows for solo and team projects
  - GitHub CLI command reference and troubleshooting

- **[NeoVim and LazyVim Guide](neovim-lazyvim-guide.md)** - Complete beginner's guide to NeoVim and LazyVim text editor
  - Installation on macOS, Ubuntu/Debian, and Arch/Omarchy Linux
  - Understanding modes and navigation
  - Basic editing and common tasks
  - Settings and configuration
  - Windows, splits, and workflows

- **[Herdr Guide](herdr-guide.md)** - Complete beginner's guide to Herdr terminal multiplexer
  - Installation on macOS, Ubuntu/Debian, and Arch/Omarchy Linux
  - Sessions, windows, and panes
  - Working with AI agents
  - Workflow patterns
  - Configuration and management

## Getting Started

### Option 1: Clone This Repository (Read-Only)

If you just want to read and use these guides, clone the repository to your local machine.

#### Prerequisites
- Git installed on your system
- Basic familiarity with the terminal

#### Step-by-Step Instructions

1. **Open your terminal** and navigate to where you want to store these guides:
   ```bash
   cd ~  # or any directory you prefer
   ```

2. **Clone the repository:**
   ```bash
   git clone https://github.com/modzs/user-guides.git
   ```

3. **Navigate into the repository:**
   ```bash
   cd user-guides
   ```

4. **View the guides:**
   - Open any `.md` file in your text editor or terminal:
   ```bash
   # View with terminal pager
   less neovim-lazyvim-guide.md
   
   # Or open in your default editor
   open neovim-lazyvim-guide.md  # macOS
   xdg-open neovim-lazyvim-guide.md  # Linux
   ```

5. **Keep up with updates:**
   If the guides are updated in the repository, pull the latest changes:
   ```bash
   git pull origin master
   ```

### Option 2: Fork and Clone (Add Your Own Customizations)

If you want to add environment-specific details, create your own version, or contribute improvements, fork this repository first.

#### What is Forking?

Forking creates a complete copy of this repository under your GitHub account. You can make changes to your copy without affecting the original. This is useful if you want to:
- Add notes specific to your setup
- Customize guides for your team
- Add sections for other tools you use
- Contribute improvements back to the original project

#### Step-by-Step Fork and Clone Instructions

1. **Create a GitHub account** (if you don't have one)
   - Go to [github.com](https://github.com)
   - Sign up for a free account
   - Verify your email

2. **Fork the repository on GitHub:**
   - Navigate to the original repository: `https://github.com/modzs/user-guides`
   - Click the **Fork** button in the top-right corner
   - Select where to fork it (usually your personal account)
   - GitHub creates a copy under your account at `https://github.com/your-username/user-guides`

3. **Clone your fork to your local machine:**
   ```bash
   git clone https://github.com/your-username/user-guides.git
   ```
   Replace `your-username` with your actual GitHub username.

4. **Navigate into your local copy:**
   ```bash
   cd user-guides
   ```

5. **Add the original repository as "upstream":**
   This lets you pull updates from the original repository while keeping your own changes:
   ```bash
   git remote add upstream https://github.com/modzs/user-guides.git
   ```
   Replace `modzs` with the original repository owner's username.

6. **Verify your remotes:**
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

### Making Changes to Your Fork

#### Adding Environment-Specific Details

1. **Create a branch for your changes:**
   ```bash
   git checkout -b my-customizations
   ```

2. **Edit the guides:**
   - Open any guide file in your text editor
   - Add your notes, customizations, or environment-specific details
   - Save the file

3. **Stage and commit your changes:**
   ```bash
   git add neovim-lazyvim-guide.md  # or whichever file you edited
   git commit -m "Add custom setup notes for my environment"
   ```

4. **Push to your fork:**
   ```bash
   git push origin my-customizations
   ```

5. **Your fork now has your customizations** - you can access them anytime by cloning from your fork instead of the original.

#### Example: Adding Custom Notes

If you want to add a "My Setup" section to the NeoVim guide:

1. Open `neovim-lazyvim-guide.md` in your editor
2. Scroll to the end
3. Add a new section:
   ```markdown
   ## My Custom Setup

   ### My Environment
   - OS: Ubuntu 22.04
   - Shell: zsh
   - Custom plugins: ...
   ```

4. Save and commit:
   ```bash
   git add neovim-lazyvim-guide.md
   git commit -m "Add my custom NeoVim configuration notes"
   git push origin my-customizations
   ```

### Pulling Updates from the Original Repository

If the original repository (upstream) receives updates and you want to incorporate them:

1. **Fetch updates from upstream:**
   ```bash
   git fetch upstream
   ```

2. **Switch to your main branch:**
   ```bash
   git checkout master
   ```

3. **Merge upstream changes:**
   ```bash
   git merge upstream/master
   ```

4. **Push the merged changes to your fork:**
   ```bash
   git push origin master
   ```

### Contributing Back to the Original Repository

If you've made improvements you think others should benefit from:

1. **Commit and push your changes to your fork** (as shown above)

2. **Create a Pull Request:**
   - Go to your fork on GitHub: `https://github.com/your-username/user-guides`
   - Click **Pull Requests** tab
   - Click **New Pull Request**
   - Click **Create Pull Request**
   - Write a description of your changes
   - Click **Create Pull Request**

3. **The repository maintainer will review your changes** and either merge them or provide feedback.

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

These guides are maintained and updated regularly. To stay current:

- **Check for updates:** `git fetch upstream` (if forked) or `git pull origin` (if cloned)
- **Report issues:** Open an issue on GitHub
- **Suggest improvements:** Fork and submit a pull request
- **Share your customizations:** Keep your fork up-to-date and share the link with your team

## License

These guides are provided as-is for educational purposes. Feel free to use, modify, and share.

## Support

For questions or issues:
1. Check the **Troubleshooting** section in each guide
2. Review this README's troubleshooting section
3. Open an issue on the GitHub repository
4. Check the guides' FAQ or ask in the repository discussions

---

**Happy learning!** Start with the guide you need most, and refer back to this README whenever you need help with git or GitHub.
