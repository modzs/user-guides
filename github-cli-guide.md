# GitHub and GitHub CLI Guide for Beginners

A complete guide to understanding GitHub, Git, and using the GitHub CLI (gh) to manage your code projects. This guide is designed for people completely new to development and version control.

**Platform Support:** This guide covers macOS, Ubuntu/Debian Linux, and Arch/Omarchy Linux. Installation methods differ, but the commands work the same across all platforms.

## Table of Contents

1. [What is GitHub?](#what-is-github)
2. [Key Concepts and Analogies](#key-concepts-and-analogies)
3. [Installation](#installation)
4. [Getting Started](#getting-started)
5. [Understanding Repositories](#understanding-repositories)
6. [Branches Explained](#branches-explained)
7. [Making Changes and Committing](#making-changes-and-committing)
8. [Push, Pull, and Merge](#push-pull-and-merge)
9. [Pull Requests](#pull-requests)
10. [Common Workflows](#common-workflows)
11. [GitHub CLI Command Reference](#github-cli-command-reference)
12. [Troubleshooting](#troubleshooting)

---

## What is GitHub?

**GitHub** is a platform where people store and collaborate on code projects. Think of it like Google Docs, but for code instead of documents.

### The Basic Idea

Imagine you and your friends are writing a book together:
- Without GitHub, you'd email copies back and forth, and it gets confusing with versions like "final_v2_really_final.doc"
- With GitHub, there's one central location where everyone can see the latest version, suggest changes, and keep track of who changed what

**GitHub** is that central location. It stores your project and all its history, allowing multiple people to work on the same code without stepping on each other's toes.

### What You Can Do

- **Store your code** in the cloud (no more losing it if your computer breaks)
- **Track changes** over time (see who changed what and when)
- **Work with others** on the same project simultaneously
- **Suggest changes** without affecting the main code
- **Review code** before it gets added to the main project

---

## Key Concepts and Analogies

Let's start with the vocabulary you'll see everywhere. These terms might sound technical, but they represent everyday ideas.

### Repository (Repo)

**What it is:** A repository is your entire project folder stored on GitHub, including all files and the complete history of changes.

**Analogy:** Think of a repository like a filing cabinet that keeps not only all your current documents, but also saves every version that's ever existed. You can look back and see "on January 5th, this file looked like this."

**Example:** A website project might have a repository containing HTML files, CSS files, JavaScript code, images, and a complete history of every change made.

### Commit

**What it is:** A commit is a "snapshot" of your code at a point in time. It's when you save your changes with a description of what you changed.

**Analogy:** Imagine you're editing a Google Doc and you make some changes. With GitHub, you don't just save silently - you explicitly say "I'm saving this version with changes to fix the login button" and give it a message. That's a commit.

**Example:**
```
Commit 1: "Added login button to homepage"
Commit 2: "Fixed spacing on login form"
Commit 3: "Added error message validation"
```

Each commit is like a checkpoint in a video game - you can always go back to it.

### Branch

**What it is:** A branch is an independent line of development. It's a copy of your code where you can make changes without affecting the main version.

**Analogy:** Imagine the main code is like a movie script. Before premiering, you might make a copy of the script and try changes to Act 2. The original script stays the same, and your copy has your experiments. When your changes work well, you can merge them back into the original.

**Example:** You might have:
- `main` branch - the official, working version
- `fix-login-bug` branch - where you're fixing a problem with login
- `new-features` branch - where you're testing new ideas

This lets multiple people work on different things without conflicts.

### Push

**What it is:** "Push" means uploading your local code changes to GitHub. It's sending your commits from your computer to the cloud.

**Analogy:** You've been working on your laptop offline. Pushing is like uploading your work to Google Drive so others can see it and it's backed up.

**Command:** `git push`

**Example:**
```bash
git push
# This sends your local commits to GitHub
```

### Pull

**What it is:** "Pull" means downloading changes from GitHub to your computer. It's getting the latest version that other people may have pushed.

**Analogy:** Your teammates have been working on the code and uploaded their changes to GitHub. Pulling is like clicking "refresh" on Google Drive to see what they've done.

**Command:** `git pull`

**Example:**
```bash
git pull
# This downloads the latest changes from GitHub to your computer
```

### Merge

**What it is:** Merging combines changes from two branches into one. Usually, it means taking work from one branch and putting it into another branch (often `main`).

**Analogy:** You've been working on Act 2 of the movie script on a separate copy. When you're happy with it, you merge it back into the official script. Act 2 now has your changes, and everything is in one place again.

**When it happens:** Usually through Pull Requests (see below).

**Example:**
```
Before merge:
main branch:    "Version 1 - Basic login"
fix-login branch: "Version 1 - Basic login + fixed bug"

After merge:
main branch: "Version 1 - Basic login + fixed bug"
```

### Pull Request (PR)

**What it is:** A pull request is a formal way to propose changes. It says "I've made changes on my branch - please review them and merge them into main if you approve."

**Analogy:** Instead of just changing the official script, you say "I made these changes - here's what I changed and why. Can you review it and let me know if it looks good before I add it to the official version?"

**Why it matters:** It allows code review, discussion, and quality checking before changes become official.

**Example:**
1. You create a branch and fix a bug
2. You create a Pull Request saying "Fixed the login bug"
3. A teammate reviews your code and says "Looks good!"
4. You merge the Pull Request into main

---

## Installation

### Installing Git

Git is the underlying system that GitHub uses. You need it installed on your computer.

#### macOS

**Using Homebrew (easiest):**
```bash
brew install git
```

**Check if it worked:**
```bash
git --version
```

#### Ubuntu/Debian Linux

**Using apt:**
```bash
sudo apt-get update
sudo apt-get install git
```

**Check if it worked:**
```bash
git --version
```

#### Arch/Omarchy Linux

**Using pacman:**
```bash
sudo pacman -S git
```

**Check if it worked:**
```bash
git --version
```

### Installing GitHub CLI (gh)

The GitHub CLI is a command-line tool that makes working with GitHub much easier. It lets you do GitHub tasks directly from your terminal.

#### macOS

**Using Homebrew (easiest):**
```bash
brew install gh
```

#### Ubuntu/Debian Linux

**Using apt:**
```bash
sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-key C99B11DEB97541F0
sudo apt-add-repository https://cli.github.com/packages
sudo apt update
sudo apt install gh
```

#### Arch/Omarchy Linux

**Using pacman:**
```bash
sudo pacman -S github-cli
```

### Verify the Installation

Check if both git and GitHub CLI are installed:
```bash
git --version
gh --version
```

You should see version numbers like:
```
git version 2.40.0
gh version 1.14.0 (2024-01-15)
```

---

## Getting Started

### 1. Create a GitHub Account

1. Go to [github.com](https://github.com)
2. Click **Sign up**
3. Enter your email, create a password, and choose a username
4. Verify your email address
5. Complete the setup flow

Your GitHub profile is now ready!

### 2. Authenticate with GitHub CLI

The GitHub CLI needs permission to act on your behalf when you use GitHub commands.

**Log in to GitHub CLI:**
```bash
gh auth login
```

You'll be asked several questions:

```
? What account do you want to log into? GitHub.com
? What is your preferred protocol for Git operations? HTTPS
? Authenticate Git with your GitHub credentials? Yes
? How would you like to authenticate GitHub CLI? Paste an authentication token
```

**To get an authentication token:**
1. Go to github.com/settings/tokens
2. Click **Generate new token**
3. Give it a name like "my-gh-token"
4. Select scope: `repo`, `read:user`, `gist`
5. Click **Generate token**
6. Copy the token (you'll only see it once)
7. Paste it into the terminal when prompted

**Verify authentication:**
```bash
gh auth status
```

You should see:
```
Logged in to github.com as your-username
```

---

## Understanding Repositories

### What's in a Repository?

A repository contains:
- **All your project files** (code, images, documentation, etc.)
- **A .git folder** (hidden) that stores all history and version information
- **README.md** (optional but common) - explains what the project is
- **.gitignore** (optional) - tells Git which files to ignore

### Create a New Repository

There are two main ways:

#### Option 1: Create on GitHub.com (then clone to your computer)

1. Go to [github.com/new](https://github.com/new)
2. Give your repository a name (e.g., "my-first-project")
3. Add a description (optional)
4. Choose "Public" (anyone can see it) or "Private" (only you)
5. Check "Add a README file"
6. Click **Create repository**

GitHub creates the repository online. Now clone it to your computer:

```bash
gh repo clone your-username/my-first-project
cd my-first-project
```

#### Option 2: Create on Your Computer (then push to GitHub)

If you already have a project folder:

```bash
cd my-first-project
git init
git add .
git commit -m "Initial commit"
```

Then create the repository on GitHub:

```bash
gh repo create my-first-project --public
```

### Clone an Existing Repository

"Clone" means downloading a repository from GitHub to your computer.

```bash
gh repo clone username/repository-name
cd repository-name
```

**Example:**
```bash
gh repo clone modzs/user-guides
cd user-guides
```

This downloads the entire project including all history. Now you have it on your computer to read and modify.

---

## Branches Explained

### Why Use Branches?

Branches let multiple people work simultaneously without interfering:
- Person A works on "add-dark-mode" branch
- Person B works on "fix-search-bug" branch
- Main branch stays stable and working

When their work is done and reviewed, both branches get merged back into main.

### Creating a Branch

**Create and switch to a new branch:**
```bash
git checkout -b my-new-feature
```

This creates a branch named `my-new-feature` and switches to it.

**Or using newer syntax:**
```bash
git switch -c my-new-feature
```

**List all branches:**
```bash
git branch -a
```

You'll see something like:
```
  main
* my-new-feature
  feature-x
```

The `*` shows which branch you're currently on.

### Switching Branches

**Switch to an existing branch:**
```bash
git checkout main
# or
git switch main
```

**Important:** Always commit your changes before switching branches. Otherwise, you might lose work or create conflicts.

### Branch Naming Convention

Use clear, descriptive names:
- `fix-login-bug`
- `add-dark-mode`
- `update-documentation`
- `refactor-database-queries`

Avoid:
- `test`, `temp`, `new` (too vague)
- `my-branch` (whose? which person?)

---

## Making Changes and Committing

### The Basic Workflow

1. Make changes to files
2. Stage the changes (tell Git which ones to save)
3. Commit the changes (save with a message)
4. Push to GitHub (upload to the cloud)

### Step 1: Make Your Changes

Edit files however you like. Use any text editor (VS Code, Sublime, vim, nano, etc.).

**Example:** Open a file and add some code.

### Step 2: Check Your Status

See what's changed:
```bash
git status
```

You'll see something like:
```
On branch main
Changes not staged for commit:
  modified:   index.html
  modified:   style.css

Untracked files:
  new-file.txt
```

- **modified** - files you changed
- **Untracked** - new files Git doesn't know about yet

### Step 3: Stage Your Changes

Tell Git which changes you want to save:

**Stage specific files:**
```bash
git add index.html
git add style.css
```

**Stage all changes:**
```bash
git add .
```

**Check what's staged:**
```bash
git status
```

Staged files now show as "Changes to be committed."

### Step 4: Commit with a Message

Save your changes with a description:

```bash
git commit -m "Fixed navigation menu styling"
```

The message should be:
- Short and clear
- Written in present tense ("Add feature" not "Added feature")
- Explain WHAT you changed, not HOW

**Examples of good messages:**
- "Fix login button alignment"
- "Add user authentication"
- "Update email validation"

**Examples of bad messages:**
- "stuff" (too vague)
- "fixed it" (what did you fix?)
- "working on things" (not descriptive)

### View Commit History

See what you've committed:

```bash
git log --oneline
```

Output:
```
a1b2c3d Fix login button alignment
e4f5g6h Add user authentication
i7j8k9l Initial commit
```

Each line is a commit with its ID and message.

---

## Push, Pull, and Merge

### Push: Send Your Work to GitHub

After committing locally, push your changes to GitHub:

```bash
git push
```

Or explicitly:
```bash
git push origin my-branch-name
```

**What happens:**
- Your commits are sent to GitHub
- The branch is updated on GitHub
- Others can see your changes

### Pull: Get Latest Changes

Get the latest changes from GitHub:

```bash
git pull
```

This is actually two commands in one:
1. `git fetch` - downloads changes from GitHub
2. `git merge` - combines them with your local code

### Updating Your Branch

Your teammates pushed changes to `main`. To include them in your branch:

```bash
git checkout main
git pull
git checkout my-branch
git merge main
```

Or more directly:
```bash
git fetch origin
git merge origin/main
```

### Merge Conflicts

Sometimes Git can't automatically merge changes. This happens when you and someone else edited the same lines.

**You'll see:**
```
git status
# shows: both modified
```

**Open the conflicted file. You'll see:**
```
<<<<<<< HEAD
your version of the code
=======
their version of the code
>>>>>>> branch-name
```

**To fix it:**
1. Decide which version to keep (or combine both)
2. Delete the conflict markers (`<<<<<<<`, `=======`, `>>>>>>>`)
3. Save the file
4. Stage and commit:
```bash
git add filename
git commit -m "Resolve merge conflict in filename"
```

---

## Pull Requests

A Pull Request (PR) is the formal way to propose changes for review before merging into main.

### Why Use Pull Requests?

- **Code review** - others check your work before it goes live
- **Discussion** - team members can suggest improvements
- **Quality control** - prevents bugs from reaching production
- **History** - keeps a record of what changed and why

### Creating a Pull Request

**Step 1: Commit and push your branch**
```bash
git commit -m "Your commit message"
git push origin my-feature-branch
```

**Step 2: Create the PR using GitHub CLI**
```bash
gh pr create --title "Brief description" --body "Detailed explanation"
```

Or interactively:
```bash
gh pr create
```

You'll be prompted for:
- Title (required) - summary of changes
- Body (optional) - detailed explanation
- Base branch - where to merge into (usually `main`)
- Head branch - your feature branch

**Example:**
```
Title: Fix login button bug

Body:
The login button wasn't clickable on mobile devices.
This PR fixes the click event handler and adds proper styling.

Fixes: #123
```

### Reviewing a Pull Request

When someone creates a PR, you can review it on GitHub.com or using the CLI:

```bash
gh pr view PR_NUMBER
```

Or list all open PRs:
```bash
gh pr list
```

### Approving and Merging

Once the code is reviewed and approved:

**Using the GitHub.com website:**
1. Go to the PR
2. Click "Approve"
3. Click "Merge pull request"
4. Confirm

**Using GitHub CLI:**
```bash
gh pr merge PR_NUMBER
```

This merges your branch into main and closes the PR.

---

## Common Workflows

### Workflow 1: Solo Project (You're the Only Developer)

1. **Create a repository**
   ```bash
   gh repo create my-project --public
   gh repo clone your-username/my-project
   cd my-project
   ```

2. **Make changes on a branch**
   ```bash
   git checkout -b new-feature
   # edit files...
   ```

3. **Commit and push**
   ```bash
   git add .
   git commit -m "Add new feature"
   git push origin new-feature
   ```

4. **Merge to main**
   ```bash
   git checkout main
   git pull
   git merge new-feature
   git push origin main
   ```

### Workflow 2: Team Project (Multiple Developers)

1. **Clone the team repository**
   ```bash
   gh repo clone team/project-name
   cd project-name
   ```

2. **Create a feature branch**
   ```bash
   git checkout -b fix-login-bug
   ```

3. **Make changes and commit**
   ```bash
   # edit files...
   git add .
   git commit -m "Fix login validation"
   git push origin fix-login-bug
   ```

4. **Create a Pull Request**
   ```bash
   gh pr create --title "Fix login validation" --body "Fixes issue #45"
   ```

5. **Wait for review**
   - A teammate reviews your code
   - They comment or approve
   - You make changes if needed

6. **Merge when approved**
   ```bash
   gh pr merge
   ```

7. **Sync back to your computer**
   ```bash
   git checkout main
   git pull
   ```

### Workflow 3: Contributing to Open Source

1. **Fork the repository** (using GitHub.com or CLI)
   ```bash
   gh repo fork original-owner/project
   ```

2. **Clone your fork**
   ```bash
   gh repo clone your-username/project
   cd project
   ```

3. **Create a branch for your contribution**
   ```bash
   git checkout -b my-improvement
   ```

4. **Make changes and commit**
   ```bash
   git add .
   git commit -m "Improve documentation"
   git push origin my-improvement
   ```

5. **Create a Pull Request to the original repository**
   ```bash
   gh pr create --repo original-owner/project --title "Improve documentation"
   ```

6. **Maintainer reviews and merges** - or requests changes

---

## GitHub CLI Command Reference

### Authentication

```bash
gh auth login              # Log into GitHub
gh auth logout             # Log out
gh auth status             # Check login status
gh auth refresh            # Refresh authentication
```

### Repository Management

```bash
gh repo create NAME        # Create new repository
gh repo clone USERNAME/REPO # Download repository
gh repo list               # List your repositories
gh repo view               # Show current repo info
gh repo delete REPO        # Delete a repository
gh repo fork REPO          # Fork a repository
```

### Working with Branches

```bash
git branch                           # List local branches
git branch -a                        # List all branches (local + remote)
git checkout -b BRANCH_NAME          # Create and switch to new branch
git checkout BRANCH_NAME             # Switch to existing branch
git branch -d BRANCH_NAME            # Delete a branch locally
git push origin --delete BRANCH_NAME # Delete branch on GitHub
```

### Commits and Changes

```bash
git status                 # See what's changed
git add FILE_NAME          # Stage specific file
git add .                  # Stage all changes
git commit -m "MESSAGE"    # Commit with message
git push                   # Push to GitHub
git pull                   # Pull from GitHub
git log --oneline          # View commit history
git log -n 5               # View last 5 commits
git diff                   # See specific changes
```

### Pull Requests

```bash
gh pr create                   # Create new pull request
gh pr list                     # List open pull requests
gh pr view PR_NUMBER          # View specific PR
gh pr checkout PR_NUMBER      # Switch to PR's branch
gh pr merge PR_NUMBER         # Merge a PR
gh pr close PR_NUMBER         # Close a PR without merging
gh pr review PR_NUMBER        # Review a PR
```

### Issues

```bash
gh issue create                    # Create new issue
gh issue list                      # List open issues
gh issue view ISSUE_NUMBER         # View specific issue
gh issue close ISSUE_NUMBER        # Close an issue
gh issue reopen ISSUE_NUMBER       # Reopen an issue
```

### Useful Combinations

```bash
# Start a new feature
git checkout -b new-feature
# ... make changes ...
git add .
git commit -m "Add new feature"
git push origin new-feature
gh pr create

# Update your branch with latest main
git fetch origin
git rebase origin/main

# See what you haven't pushed yet
git log origin/main..HEAD

# See detailed changes in a commit
git show COMMIT_ID
```

---

## Troubleshooting

### "fatal: not a git repository"

**Problem:** You're trying to use git commands outside a git repository.

**Solution:** Navigate to your project directory:
```bash
cd my-project
git status
```

### "Permission denied (publickey)"

**Problem:** SSH authentication failed when pushing.

**Solution:** Use HTTPS instead of SSH:
```bash
git remote set-url origin https://github.com/username/repository.git
```

Or set up SSH keys properly:
```bash
ssh-keygen -t ed25519 -C "your-email@example.com"
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519
# Copy ~/.ssh/id_ed25519.pub to GitHub SSH keys settings
```

### "Your branch is ahead of 'origin/main' by X commits"

**Problem:** You've committed locally but haven't pushed to GitHub yet.

**Solution:** Push your commits:
```bash
git push
```

### "Merge conflict"

**Problem:** Git couldn't automatically merge changes because you both edited the same lines.

**Solution:**
1. Open the conflicted file
2. Find the conflict markers (`<<<<<<<`, `=======`, `>>>>>>>`)
3. Manually choose which version to keep
4. Delete the markers
5. Save and commit:
```bash
git add .
git commit -m "Resolve merge conflict"
```

### "Changes would be overwritten by merge"

**Problem:** You have uncommitted changes and tried to switch branches or merge.

**Solution:** Commit your changes first:
```bash
git add .
git commit -m "Save work in progress"
```

Or temporarily stash them:
```bash
git stash
git switch branch-name
git stash pop
```

### "Please enter a commit message"

**Problem:** Your terminal opened an editor (likely vim) for the commit message.

**Solution:**
1. Type your message
2. Press `Escape`
3. Type `:wq` (write and quit)
4. Press `Enter`

To avoid this in the future, use `-m` flag:
```bash
git commit -m "Your message here"
```

### "The branch is not fully merged"

**Problem:** Git won't delete a branch because it has unique commits.

**Solution:** Use force delete:
```bash
git branch -D BRANCH_NAME
```

Only do this if you're sure you don't need those commits!

### "Can't find authentication token"

**Problem:** You got an error about authentication when using `gh` commands.

**Solution:** Re-authenticate:
```bash
gh auth logout
gh auth login
```

Follow the prompts to get a new token.

### "Detached HEAD state"

**Problem:** You checked out a commit ID instead of a branch and now you're in an odd state.

**Solution:** Go back to main:
```bash
git checkout main
```

If you made commits while detached, save them to a new branch:
```bash
git branch my-saved-work
git checkout main
```

---

## Key Takeaways

- **GitHub** is where code lives and teams collaborate
- **Git** is the system that tracks changes and versions
- **Commit** before you do anything else - it's your safety net
- **Branch** to work on features safely without affecting main
- **Push** to share your work, **Pull** to get others' work
- **Pull Requests** are how teams review code before merging
- **Good commit messages** are a gift to your future self

### Remember

- Commit often (many small commits are better than one giant one)
- Write clear commit messages (your team will thank you)
- Always pull before pushing (avoid conflicts)
- Never force push to `main` (unless you really know what you're doing)
- When in doubt, ask. The development community is generally helpful!

---

## Additional Resources

- [GitHub Docs](https://docs.github.com) - Official GitHub documentation
- [GitHub CLI Manual](https://cli.github.com/manual) - Official CLI reference
- [Git Book](https://git-scm.com/book/en/v2) - Free comprehensive Git guide
- [GitHub Learning Lab](https://github.skills) - Interactive tutorials

## Common Git Aliases

Make commands shorter by creating aliases:

```bash
git config --global alias.co checkout
git config --global alias.br branch
git config --global alias.ci commit
git config --global alias.st status
git config --global alias.unstage 'restore --staged'
git config --global alias.last 'log -1 HEAD'
git config --global alias.visual 'log --graph --oneline --all'
```

Then you can use:
```bash
git st          # instead of git status
git co -b name  # instead of git checkout -b name
git ci -m "msg" # instead of git commit -m "msg"
```

---

**Happy coding!** Start by creating your first repository and making some commits. The best way to learn is by doing.
