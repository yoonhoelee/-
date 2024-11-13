# Git Commands Reference Guide

## 🔄 Basic Operations

### Status and Information
```bash
# Check status of working directory
git status

# Show commit history
git log

# Show commit history with graph visualization
git log --graph --oneline --all

# Show changes in working directory
git diff

# Show changes in staged files
git diff --staged
```

### Branch Operations
```bash
# List all branches
git branch

# List all branches including remote
git branch -a

# Create new branch
git checkout -b branch-name

# Switch to existing branch
git checkout branch-name

# Delete local branch
git branch -d branch-name

# Force delete local branch
git branch -D branch-name
```

## 🔨 Working with Changes

### Abandoning Changes
```bash
# Discard all local changes in working directory
git reset --hard

# Reset to remote branch state
git reset --hard origin/branch-name

# Remove untracked files and directories
git clean -fd

# Discard changes for specific file
git checkout -- filename
```

### Stashing Changes
```bash
# Stash current changes
git stash

# Stash with a description
git stash save "description"

# List stashes
git stash list

# Apply most recent stash
git stash apply

# Apply specific stash
git stash apply stash@{n}

# Drop most recent stash
git stash drop

# Pop most recent stash (apply and drop)
git stash pop
```

## 🔃 Syncing with Remote

### Fetching and Pulling
```bash
# Fetch remote changes
git fetch origin

# Fetch all remotes
git fetch --all

# Pull changes from remote
git pull

# Pull with rebase instead of merge
git pull --rebase
```

### Pushing Changes
```bash
# Push to remote
git push origin branch-name

# Force push to remote (use with caution!)
git push -f origin branch-name

# Push and set upstream
git push -u origin branch-name
```

## 🛠️ Advanced Operations

### Commit Management
```bash
# Amend last commit
git commit --amend

# Amend last commit without editing message
git commit --amend --no-edit

# Interactive rebase for last n commits
git rebase -i HEAD~n

# Squash last n commits
git reset --soft HEAD~n && git commit
```

### Remote Operations
```bash
# Add remote
git remote add origin url

# Show remote information
git remote -v

# Remove remote
git remote remove origin

# Change remote URL
git remote set-url origin new-url
```

### History Rewriting
```bash
# Reset to specific commit
git reset --hard commit-hash

# Revert a commit
git revert commit-hash

# Cherry-pick a commit from another branch
git cherry-pick commit-hash
```

## 🚨 Emergency Situations

### Recovery Commands
```bash
# Recover deleted commits (within grace period)
git reflog
git checkout -b recovery-branch commit-hash

# Recover deleted branch
git reflog
git checkout -b recovered-branch commit-hash

# Undo a git reset --hard
git reflog
git reset --hard HEAD@{n}
```

## 🔍 Useful Aliases
Add these to your `.gitconfig`:

```bash
[alias]
    # Short status
    st = status -s
    
    # Pretty log
    lg = log --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit
    
    # Undo last commit
    undo = reset HEAD~1 --mixed
    
    # Show branches sorted by last modified
    branches = for-each-ref --sort=-committerdate refs/heads/ --format='%(committerdate:short) %(refname:short)'
```

## 🌟 Pro Tips

1. **Before Force Push**
   ```bash
   # Create backup branch
   git checkout -b backup-branch
   git checkout original-branch
   git push -f
   ```

2. **Clean Working Directory**
   ```bash
   # Reset and clean in one command
   git reset --hard && git clean -fd
   ```

3. **Update Feature Branch**
   ```bash
   # Update feature branch with main
   git checkout feature-branch
   git rebase main
   ```

4. **Temporarily Switch Branch**
   ```bash
   # Quick switch and return
   git checkout -
   ```

## ⚠️ Safety Warnings

1. Never force push to main/master branches
2. Always create backups before destructive operations
3. Be careful with `git clean` - it removes untracked files permanently
4. Double-check branch names before force pushing
5. Communicate with team members before force pushing to shared branches
