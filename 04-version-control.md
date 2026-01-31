# Chapter 4: Version Control

[← Previous: Managing Dependencies](./03-dependencies.md) | [Back to README](./README.md) | [Next: Linting →](./05-linting.md)

## What is Version Control?

Version control is a system that records changes to files over time so you can recall specific versions later. Think of it as a time machine for your code - you can see what changed, when it changed, who changed it, and why.

Without version control, you might resort to creating copies of your project folder:
```
my_project/
my_project_backup/
my_project_final/
my_project_final_v2/
my_project_actually_final/
```

This quickly becomes unmanageable. Version control provides a better way.

## Benefits of Version Control

### 1. History and Audit Trail
Every change is recorded with:
- What changed (the diff)
- When it changed (timestamp)
- Who changed it (author)
- Why it changed (commit message)

### 2. Collaboration
Multiple people can work on the same codebase without stepping on each other's toes. The version control system helps merge changes and resolve conflicts.

### 3. Backup and Recovery
Your code is backed up at every commit. If something breaks, you can easily revert to a previous working state.

### 4. Branching and Experimentation
You can create branches to experiment with new features without affecting the main codebase. If the experiment works, merge it in. If not, delete the branch.

### 5. Code Review
Changes can be reviewed before being integrated into the main codebase, improving code quality and knowledge sharing.

## Introduction to Git

Git is the most popular version control system in the world. It's:
- **Distributed**: Every developer has a full copy of the repository history
- **Fast**: Most operations are local and don't require network access
- **Free and Open Source**: Created by Linus Torvalds in 2005

### Key Resources
- [Git on GitHub](https://github.com/git/git)
- [Official Git Documentation](https://git-scm.com/doc)
- [Pro Git Book](https://git-scm.com/book/en/v2) (free online)

## Installing Git

### macOS
```bash
# Using Homebrew
brew install git

# Or download from git-scm.com
```

### Linux
```bash
# Debian/Ubuntu
sudo apt-get install git

# Fedora
sudo dnf install git

# Arch
sudo pacman -S git
```

### Windows
Download the installer from [git-scm.com](https://git-scm.com/download/win) or use:
```bash
# Using winget
winget install Git.Git
```

### Verify Installation
```bash
git --version
```

## Basic Git Concepts

### Repository (Repo)
A repository is a directory that Git tracks. It contains all your files and the complete history of changes.

### Commit
A commit is a snapshot of your project at a specific point in time. Each commit has a unique identifier (hash) and a message describing the changes.

### Branch
A branch is a parallel version of your repository. The default branch is usually called `main` or `master`.

### Remote
A remote is a version of your repository hosted on the internet or network (like on GitHub, GitLab, or Bitbucket).

## Essential Git Commands

### Setting Up Git

First, configure your identity:
```bash
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"
```

### Creating a Repository

```bash
# Initialize a new repository in the current directory
git init

# Or clone an existing repository
git clone https://github.com/username/repository.git
```

### Basic Workflow

```bash
# Check the status of your repository
git status

# Add files to staging area
git add filename.py           # Add a specific file
git add .                     # Add all changed files

# Commit your changes
git commit -m "Descriptive commit message"

# View commit history
git log
git log --oneline            # Compact view
```

### Working with Remotes

```bash
# Add a remote repository
git remote add origin https://github.com/username/repo.git

# Push changes to remote
git push origin main

# Pull changes from remote
git pull origin main

# View remotes
git remote -v
```

### Branching

```bash
# Create a new branch
git branch feature-name

# Switch to a branch
git checkout feature-name

# Create and switch in one command
git checkout -b feature-name

# Or using the newer syntax
git switch -c feature-name

# List branches
git branch

# Merge a branch into current branch
git merge feature-name

# Delete a branch
git branch -d feature-name
```

## Typical Git Workflow

Here's a common workflow for working on a feature:

```bash
# 1. Make sure you're on the main branch and it's up to date
git checkout main
git pull origin main

# 2. Create a new branch for your feature
git checkout -b add-login-feature

# 3. Make changes to your files
# ... edit code ...

# 4. Check what changed
git status
git diff

# 5. Stage your changes
git add .

# 6. Commit your changes
git commit -m "Add login feature with email validation"

# 7. Push your branch to the remote
git push origin add-login-feature

# 8. Create a pull request on GitHub (done in the web interface)

# 9. After review and approval, merge via GitHub

# 10. Switch back to main and update
git checkout main
git pull origin main

# 11. Delete the feature branch
git branch -d add-login-feature
```

## .gitignore File

A `.gitignore` file tells Git which files or directories to ignore. This is crucial for excluding:
- Virtual environments
- Compiled code
- IDE configuration files
- Sensitive information (API keys, passwords)
- Build artifacts

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

You can generate comprehensive .gitignore files at [gitignore.io](https://www.toptal.com/developers/gitignore).

## Best Practices

### 1. Commit Often, Commit Early
Make small, focused commits. It's easier to understand and revert small changes than large ones.

### 2. Write Good Commit Messages
```bash
# Good
git commit -m "Add email validation to login form"
git commit -m "Fix off-by-one error in pagination"

# Bad
git commit -m "Fixed stuff"
git commit -m "Changes"
git commit -m "Update"
```

**Format for commit messages:**
- First line: Short summary (50 characters or less)
- Blank line
- Detailed explanation if needed

### 3. Don't Commit Sensitive Information
Never commit:
- Passwords
- API keys
- Private keys
- Database credentials

Use environment variables or configuration files (listed in .gitignore) instead.

### 4. Keep the Main Branch Stable
The main branch should always be in a working state. Use feature branches for development.

### 5. Pull Before You Push
Always pull the latest changes before pushing to avoid conflicts:
```bash
git pull origin main
git push origin main
```

### 6. Review Before Committing
Always review your changes before committing:
```bash
git status
git diff
```

## Common Scenarios

### Undoing Changes

```bash
# Discard changes to a file (before staging)
git checkout -- filename.py

# Unstage a file (keep the changes)
git reset HEAD filename.py

# Amend the last commit (add forgotten files or fix message)
git add forgotten_file.py
git commit --amend

# Revert a commit (creates a new commit that undoes changes)
git revert commit_hash
```

### Resolving Conflicts

When Git can't automatically merge changes, you'll need to resolve conflicts manually:

```bash
# 1. Git will mark conflicts in your files
# 2. Open the file and look for conflict markers:
<<<<<<< HEAD
your changes
=======
their changes
>>>>>>> branch-name

# 3. Edit the file to resolve the conflict
# 4. Stage the resolved file
git add filename.py

# 5. Complete the merge
git commit
```

### Stashing Changes

Save your work temporarily without committing:
```bash
# Stash current changes
git stash

# List stashes
git stash list

# Apply the most recent stash
git stash apply

# Apply and remove the most recent stash
git stash pop
```

## GitHub and Remote Hosting

While Git is the version control system, platforms like GitHub, GitLab, and Bitbucket provide hosting and collaboration features:

- **GitHub**: [https://github.com](https://github.com) - Most popular, great for open source
- **GitLab**: [https://gitlab.com](https://gitlab.com) - Built-in CI/CD
- **Bitbucket**: [https://bitbucket.org](https://bitbucket.org) - Integrates with Atlassian tools

These platforms add:
- Pull requests / Merge requests
- Issue tracking
- Project management
- CI/CD integration
- Code review tools
- Wikis and documentation

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

# Branching
git branch <name>
git checkout <name>
git checkout -b <name>
git merge <name>

# Viewing
git log
git diff
git show <commit>

# Undoing
git checkout -- <file>
git reset HEAD <file>
git revert <commit>
```

## What's Next?

Now that you can track your code changes with Git, let's look at how to improve code quality with linting - automatically checking your code for errors and style issues.

[← Previous: Managing Dependencies](./03-dependencies.md) | [Back to README](./README.md) | [Next: Linting →](./05-linting.md)
