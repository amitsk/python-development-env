# Chapter 4: Version Control

[← Previous: Managing Dependencies](./03-dependencies.md) | [Back to README](./README.md) | [Next: Linting →](./05-linting.md)

## What is Version Control?

Version control records changes to files over time so you can get a specific version back. It is a history of the work: what changed, when, who, and why.

Without it, people invent a filing system:

```
my_project/
my_project_backup/
my_project_final/
my_project_final_v2/
my_project_actually_final/
```

That does not scale past one person and one week. Version control is the better habit.

## Why it matters

Every commit stores the diff, a timestamp, an author, and a message. That is an audit trail you can `git log` instead of reconstructing from memory.

More than one person can work on the same tree. Branches keep experiments off `main`. Merges and pull requests are how those experiments land. If a change was a mistake, you revert it; you do not restore a zip file named `final_v2`.

Code review sits on top of the same history. A pull request is a conversation about a commit range, not a folder of “please look at this.”

## Introduction to Git

Git is the version control system you will actually use. It is:

- **Distributed** — every clone has the full history. The network is for sharing, not for `git log`.
- **Fast** — most operations are local.
- **Free and open source** — created by Linus Torvalds in 2005, still the default in 2026.

### Key Resources

- [Git on GitHub](https://github.com/git/git)
- [Official Git Documentation](https://git-scm.com/doc)
- [Pro Git Book](https://git-scm.com/book/en/v2) (free online)

## Installing Git

### macOS

```bash
# Homebrew
brew install git

# Or download from git-scm.com
```

### Linux

```bash
# Debian/Ubuntu
sudo apt install git

# Fedora
sudo dnf install git

# Arch
sudo pacman -S git
```

### Windows

Download the installer from [git-scm.com](https://git-scm.com/download/win) or:

```bash
winget install Git.Git
```

### Verify Installation

```bash
git --version
```

## Basic Git Concepts

### Repository (Repo)

A directory Git is tracking, plus the complete history in `.git/`. `uv init` often creates one for you; `git init` does it by hand.

### Commit

A snapshot of the project at a point in time. Each commit has a hash and a message. Small commits with honest messages are the whole game.

### Branch

A parallel line of history. The default branch is `main`. Older repositories may still use `master`; new work uses `main`.

### Remote

A copy of the repository on a host (GitHub, GitLab, Bitbucket). `origin` is the conventional name for the one you cloned from or first pushed to.

## Essential Git Commands

### Setting Up Git

```bash
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"
```

Use the email that will match your GitHub (or GitLab) account, or commits will look like they came from a stranger who happens to share your name.

### Creating a Repository

```bash
# In the current directory
git init

# Copy an existing repository
git clone https://github.com/username/repository.git
```

### Basic Workflow

```bash
git status

git add filename.py           # stage one file
git add .                     # stage everything in the directory

git commit -m "Descriptive commit message"

git log
git log --oneline
```

Staging is deliberate. `git add -p` is there when you want to commit one hunk and leave the rest.

### Working with Remotes

```bash
git remote add origin https://github.com/username/repo.git

git push origin main
git pull origin main

git remote -v
```

### Branching

Prefer `git switch` for changing branches and `git switch -c` for creating one. `git checkout` still works; it is the older, overloaded command you will see in Stack Overflow answers and 2017 blog posts.

```bash
# Create and switch
git switch -c feature-name

# Switch to an existing branch
git switch feature-name

# List branches
git branch

# Merge feature-name into the branch you are on
git merge feature-name

git branch -d feature-name
```

Equivalent older spelling, for when you are reading someone else’s notes:

```bash
git checkout feature-name
git checkout -b feature-name
```

Same idea. Two jobs (switch branch, restore files) jammed into one verb. Use `switch` and `restore`.

## Typical Git Workflow

A feature branch, a pull request, then back to `main`:

```bash
# 1. Start from an up-to-date main
git switch main
git pull origin main

# 2. Branch for the work
git switch -c add-login-feature

# 3. Edit files
# ... code ...

# 4. See what changed
git status
git diff

# 5. Stage
git add .

# 6. Commit
git commit -m "Add login feature with email validation"

# 7. Publish the branch
git push origin add-login-feature

# 8. Open a pull request on GitHub (web UI)

# 9. After review, merge on GitHub

# 10. Return to main and pick up the merge
git switch main
git pull origin main

# 11. Delete the local feature branch
git branch -d add-login-feature
```

That is the loop. Direct commits to `main` on a shared repo are how “quick fixes” become archaeology.

## .gitignore File

`.gitignore` tells Git which paths not to track. You want it to exclude:

- Virtual environments
- Bytecode and build artifacts
- Editor junk
- Secrets (API keys, `.env`, private keys)
- Coverage and cache directories

### Example .gitignore for Python Projects

```gitignore
# Virtual environments
venv/
.venv/
env/
ENV/

# Python cache
__pycache__/
*.py[cod]
*$py.class
*.so

# Distribution / packaging
dist/
build/
*.egg-info/

# Testing
.pytest_cache/
.coverage
htmlcov/

# IDE
.vscode/
.idea/
*.swp
*.swo

# Environment variables
.env
.env.local

# OS
.DS_Store
Thumbs.db
```

Generate a fuller file at [gitignore.io](https://www.toptal.com/developers/gitignore) (Toptal’s generator). `uv init` already writes a Python-sensible `.gitignore`; add secrets patterns yourself if they are missing. `uv.lock` should be tracked, not ignored.

## Best Practices

**Commit often, in small pieces.** A commit that does one thing is easy to review and easy to revert. A commit named “wip stuff” that touches twelve files is a weekend.

**Write a message that future-you can search:**

```bash
# Good
git commit -m "Add email validation to login form"
git commit -m "Fix off-by-one error in pagination"

# Bad
git commit -m "Fixed stuff"
git commit -m "Changes"
git commit -m "Update"
```

First line: a short summary, about 50 characters. Blank line. Details if the *why* is not obvious from the diff.

**Do not commit secrets.** Passwords, API keys, private keys, database URLs with credentials — none of that belongs in Git. Use environment variables or a local config file that is listed in `.gitignore`. If you commit a secret, rotating it is the fix; deleting the file in a later commit is not.

**Keep `main` green.** Feature branches exist so `main` stays shippable. Broken experimental work lives on a branch until it is not broken.

**Pull before you push** so you merge (or rebase) locally instead of discovering a conflict on the remote:

```bash
git pull origin main
git push origin main
```

On a feature branch, pull the branch, not a surprise rewrite of `main`, unless you are integrating.

**Review before you commit:**

```bash
git status
git diff
git diff --staged
```

`git add .` followed by a commit you did not look at is how `debug.py` and a 20 MB CSV enter history.

## Common Scenarios

### Undoing Changes

Prefer `git restore` for files. `git checkout -- file` is the older spelling of the same idea.

```bash
# Discard unstaged changes to a file
git restore filename.py

# Unstage a file; keep the working-tree changes
git restore --staged filename.py

# Older spelling you will still see:
# git checkout -- filename.py
# git reset HEAD filename.py

# Fix the last commit (message or forgotten files). Do not amend commits you already pushed.
git add forgotten_file.py
git commit --amend

# Undo a commit by adding a new commit that reverses it
git revert commit_hash
```

### Resolving Conflicts

When Git cannot merge automatically, it marks the file:

```
<<<<<<< HEAD
your changes
=======
their changes
>>>>>>> branch-name
```

Edit until the file is the version you want, remove the markers, then:

```bash
git add filename.py
git commit
```

Conflicts are normal. Leaving the markers in the file is the part that is not.

### Stashing Changes

Park uncommitted work without making a fake commit:

```bash
git stash
git stash list
git stash apply     # apply, keep the stash
git stash pop       # apply and drop
```

Useful when `main` moved and you need to switch branches with a dirty tree. Not a substitute for a commit if the work matters.

## GitHub and Remote Hosting

Git is the tool. GitHub, GitLab, and Bitbucket are hosts with collaboration bolted on:

- **GitHub**: [https://github.com](https://github.com) — default for open source, and for a lot of private work
- **GitLab**: [https://gitlab.com](https://gitlab.com) — strong built-in CI/CD
- **Bitbucket**: [https://bitbucket.org](https://bitbucket.org) — Atlassian-shaped shops

They add pull/merge requests, issues, project boards, CI, review UI, and wikis. Chapter 10 uses GitHub Actions; the Git part you learned here does not care which host stores `origin`.

## Quick Reference

```bash
# Setup
git config --global user.name "Name"
git config --global user.email "email@example.com"

# Initialize
git init
git clone <url>

# Basic workflow
git status
git add <file>
git commit -m "message"
git push origin main
git pull origin main

# Branching (modern)
git switch <name>
git switch -c <name>
git merge <name>
git branch -d <name>

# Branching (older spelling)
git checkout <name>
git checkout -b <name>

# Viewing
git log
git log --oneline
git diff
git show <commit>

# Restoring files (modern)
git restore <file>
git restore --staged <file>

# Restoring files (older spelling)
git checkout -- <file>
git reset HEAD <file>

git revert <commit>
git stash
git stash pop
```

## What's Next?

Your code, `pyproject.toml`, and `uv.lock` can live in Git with a workflow other people will recognize. Next is linting: automatic checks for errors and style so review does not start with “you forgot an import.”

[← Previous: Managing Dependencies](./03-dependencies.md) | [Back to README](./README.md) | [Next: Linting →](./05-linting.md)
