#  Git
---


## Table of Contents
1. [Core Concepts (Q1–Q20)](#1-core-concepts)
2. [Branching & Merging (Q21–Q38)](#2-branching--merging)
3. [Remote Repositories (Q39–Q52)](#3-remote-repositories)
4. [Advanced Git (Q53–Q72)](#4-advanced-git)
5. [Git Workflows (Q73–Q85)](#5-git-workflows)
6. [Git in DevOps & CI/CD (Q86–Q100)](#6-git-in-devops--cicd)

---

## 1. Core Concepts

**Q1. What is Git?**
> Git is a distributed version control system (DVCS). It tracks changes in code over time, allows multiple developers to work together, and lets you go back to any previous version.
> - Created by Linus Torvalds in 2005
> - Every developer has a full copy of the repository (distributed)
> - Fast, reliable, and supports non-linear development (branching)

---

**Q2. What is the difference between Git and GitHub/GitLab/Bitbucket?**
> - **Git** — the version control tool (runs locally, command-line)
> - **GitHub** — cloud platform hosting Git repos + collaboration features (PRs, Actions)
> - **GitLab** — similar to GitHub + built-in CI/CD
> - **Bitbucket** — Atlassian's platform, integrates well with Jira
> Git is the engine. GitHub/GitLab/Bitbucket are the platforms built on top.

---

**Q3. What is a Git Repository?**
> A repository (repo) is a directory tracked by Git. It contains:
> - All files of the project
> - The entire history of changes
> - All branches and tags
> - `.git/` folder stores all Git data

---

**Q4. What are the 3 states of a file in Git?**
> - **Modified:** File has been changed but not staged yet (working directory)
> - **Staged:** File is marked to be included in the next commit (staging area/index)
> - **Committed:** Changes are safely stored in Git history (local repository)
> ```
> Working Directory → git add → Staging Area → git commit → Repository
> ```

---

**Q5. What is `git init`?**
```bash
git init                    # initialize new repo in current directory
git init my-project         # create new directory and initialize
```
> Creates a `.git/` folder which is the Git database for the project.

---

**Q6. What is `git clone`?**
```bash
git clone https://github.com/org/repo.git         # clone repo
git clone https://github.com/org/repo.git mydir   # clone into mydir
git clone --depth 1 https://github.com/org/repo   # shallow clone (latest only)
git clone -b main https://github.com/org/repo     # clone specific branch
```
> Creates a full local copy of a remote repository, including all history.

---

**Q7. What is `git status`?**
```bash
git status          # show working tree status
git status -s       # short format
```
> Shows:
> - Files modified but not staged
> - Files staged and ready to commit
> - Untracked files (new files Git doesn't know about)
> - Current branch

---

**Q8. What is `git add`?**
```bash
git add file.txt            # stage specific file
git add .                   # stage ALL changes in current directory
git add *.js                # stage all JS files
git add -p                  # interactive staging (choose hunks)
git add -A                  # stage all changes (modified + deleted + new)
```

---

**Q9. What is `git commit`?**
```bash
git commit -m "feat: add user authentication"   # commit with message
git commit -am "fix: update connection string"  # stage + commit tracked files
git commit --amend -m "updated message"         # fix last commit message
git commit --amend --no-edit                    # add staged files to last commit
```

---

**Q10. What makes a good commit message?**
> Follow **Conventional Commits** format:
> ```
> <type>(<scope>): <description>
>
> [optional body]
> [optional footer]
> ```
> Types: `feat`, `fix`, `docs`, `style`, `refactor`, `test`, `chore`, `perf`, `ci`
> Examples:
> ```
> feat(auth): add OAuth2 login with Google
> fix(api): handle null response from payment service
> docs: update deployment guide for K8s
> ci: add trivy image scanning to pipeline
> ```
> Rules: imperative mood, max 72 chars, explain WHY not HOW.

---

**Q11. What is `.gitignore`?**
> Tells Git which files/directories to ignore (not track):
```gitignore
# Dependencies
node_modules/
vendor/

# Build output
dist/
build/
*.class
*.jar

# Environment
.env
.env.local
*.env

# IDE
.idea/
.vscode/
*.swp

# Logs
*.log
logs/

# OS
.DS_Store
Thumbs.db
```
> Never commit: secrets, build artifacts, dependencies, personal config.

---

**Q12. What is `git log`?**
```bash
git log                        # full history
git log --oneline              # compact: hash + message
git log --oneline --graph      # with branch graph
git log --oneline -10          # last 10 commits
git log --author="Alice"       # by author
git log --since="2 weeks ago"  # by date
git log --grep="bug fix"       # search commit messages
git log -p                     # show diff for each commit
git log filename.txt           # history of specific file
git log branch1..branch2       # commits in branch2 not in branch1
```

---

**Q13. What is `git diff`?**
```bash
git diff                    # changes in working dir (not staged)
git diff --staged           # changes staged for commit
git diff HEAD               # all changes since last commit
git diff branch1 branch2   # differences between branches
git diff abc123 def456      # differences between commits
git diff HEAD~3 HEAD        # last 3 commits
git diff --stat             # files changed (summary)
```

---

**Q14. What is a Git commit hash?**
> Every commit has a unique SHA-1 hash (40 characters): `a3b4c5d6e7f8...`. Can be used as short form (first 7 chars): `a3b4c5d`.
> Used to reference specific commits in commands like `git checkout`, `git revert`, `git diff`.

---

**Q15. What is `HEAD` in Git?**
> `HEAD` is a pointer to the current commit you're working from. Usually points to the tip of your current branch.
> - `HEAD` = current commit
> - `HEAD~1` = one commit before current
> - `HEAD~3` = three commits before current
> - `HEAD^` = parent of current commit (same as `HEAD~1`)

---

**Q16. What is `git show`?**
```bash
git show                    # show last commit with diff
git show abc123             # show specific commit
git show HEAD~2             # show 2 commits ago
git show HEAD:file.txt      # show file content at HEAD
git show branch:file.txt    # show file on specific branch
```

---

**Q17. What is `git blame`?**
```bash
git blame file.txt          # who changed each line and when
git blame -L 10,20 file.txt # lines 10-20 only
```
> Shows commit hash, author, date, and content for each line. Useful for finding who introduced a bug.

---

**Q18. What is a `.git` directory?**
> Contains all Git data:
> ```
> .git/
> ├── HEAD          # current branch reference
> ├── config        # repo configuration
> ├── objects/      # all commits, trees, blobs (compressed)
> ├── refs/         # branch and tag pointers
> │   ├── heads/    # local branches
> │   └── remotes/  # remote branches
> ├── index         # staging area
> └── COMMIT_EDITMSG # last commit message
> ```

---

**Q19. What is `git stash`?**
```bash
git stash                       # save uncommitted changes temporarily
git stash push -m "WIP: login"  # stash with description
git stash list                  # show all stashes
git stash pop                   # apply latest stash and remove it
git stash apply stash@{2}       # apply specific stash (keep it)
git stash drop stash@{0}        # delete specific stash
git stash clear                 # delete all stashes
git stash branch feature/login  # create branch from stash
```
> Use when: switching branches but have uncommitted changes you don't want to commit yet.

---

**Q20. What is `git config`?**
```bash
# Global config
git config --global user.name "Alice"
git config --global user.email "alice@example.com"
git config --global core.editor "vim"
git config --global init.defaultBranch "main"
git config --global pull.rebase true

# View config
git config --list
git config user.name

# Local repo config (overrides global)
git config user.email "work@company.com"

# Config stored in: ~/.gitconfig (global), .git/config (local)
```

---

## 2. Branching & Merging

**Q21. What is a Branch in Git?**
> A branch is a lightweight pointer to a commit. Creating a branch creates a new line of development without affecting other branches.
```bash
git branch                      # list local branches
git branch -a                   # list all branches (local + remote)
git branch feature/login        # create branch
git checkout feature/login      # switch to branch
git checkout -b feature/login   # create AND switch (shortcut)
git switch -c feature/login     # modern way to create and switch
git branch -d feature/login     # delete branch (safe - only if merged)
git branch -D feature/login     # force delete
git branch -m old-name new-name # rename branch
```

---

**Q22. What is `git merge`?**
> Merge combines changes from one branch into another.
```bash
git checkout main
git merge feature/login         # merge feature into main
git merge --no-ff feature/login # always create merge commit
git merge --squash feature/login # squash all commits into one
git merge --abort               # cancel merge (when conflict)
```
> Types of merge:
> - **Fast-forward:** No divergence — just moves pointer forward (linear history)
> - **3-way merge:** Branches diverged — creates a merge commit

---

**Q23. What is `git rebase`?**
> Rebase moves or replays commits from one branch onto another, creating a linear history.
```bash
git checkout feature/login
git rebase main                 # rebase feature onto main
git rebase -i HEAD~5            # interactive rebase (last 5 commits)
git rebase --abort              # cancel rebase
git rebase --continue           # continue after fixing conflict
```
> Result: Linear, clean history (no merge commits).

---

**Q24. What is the difference between Merge and Rebase?**
> | Feature | Merge | Rebase |
> |---|---|---|
> | History | Preserves all commits + merge commit | Linear, cleaner |
> | Safety | Safe on public branches | Never rebase public/shared branches! |
> | Conflict | Resolve once | Resolve for each replayed commit |
> | Use case | Feature branches to main | Keeping feature branch up to date |
> **Golden rule:** Never rebase commits that have been pushed to a shared branch.

---

**Q25. What is a Merge Conflict and how do you resolve it?**
> A conflict happens when two branches change the same part of the same file.
```bash
# Git marks conflicts like this:
<<<<<<< HEAD (main branch)
const PORT = 3000;
=======
const PORT = 8080;
>>>>>>> feature/config-update

# Resolution:
# 1. Edit the file to keep what you want
const PORT = 8080;  # or whatever is correct

# 2. Stage the resolved file
git add config.js

# 3. Complete the merge
git commit -m "Merge: resolve port conflict"
```
```bash
# Tools for conflict resolution
git mergetool           # use configured visual tool
git diff --conflict     # show conflicts
git checkout --ours file.txt    # take our version
git checkout --theirs file.txt  # take their version
```

---

**Q26. What is `git cherry-pick`?**
> Apply a specific commit from one branch to another:
```bash
git cherry-pick abc123          # apply one commit
git cherry-pick abc123 def456   # apply multiple commits
git cherry-pick abc123..def456  # apply range
git cherry-pick --no-commit abc123  # apply without committing
git cherry-pick --abort         # cancel
```
> Use case: Bug fix is on develop branch, need to apply it to production branch without merging everything.

---

**Q27. What is `git tag`?**
```bash
# Lightweight tag
git tag v1.0.0

# Annotated tag (recommended - stores tagger, date, message)
git tag -a v1.0.0 -m "Release version 1.0.0"
git tag -a v1.0.0 abc123 -m "Tag specific commit"

# List tags
git tag
git tag -l "v1.*"

# Push tags
git push origin v1.0.0         # push specific tag
git push origin --tags          # push all tags

# Delete tag
git tag -d v1.0.0
git push origin --delete v1.0.0
```
> Use annotated tags for releases — they're stored as full objects with metadata.

---

**Q28. What is fast-forward merge?**
> When the target branch hasn't diverged from the source, Git just moves the pointer forward — no new merge commit.
```
Before:
main:    A → B → C
feature:          C → D → E

After merge (fast-forward):
main:    A → B → C → D → E
```
> Use `--no-ff` to always create a merge commit (preserves branch history).

---

**Q29. What is `git reset`?**
```bash
git reset --soft HEAD~1    # undo commit, keep changes STAGED
git reset --mixed HEAD~1   # undo commit, keep changes UNSTAGED (default)
git reset --hard HEAD~1    # undo commit, DISCARD all changes (dangerous!)

git reset HEAD file.txt    # unstage a file (keep changes in working dir)
git reset --hard origin/main  # reset local branch to match remote
```
> ⚠️ Never use `--hard` on shared commits — rewrites history!

---

**Q30. What is `git revert`?**
```bash
git revert abc123           # create new commit that undoes abc123
git revert HEAD             # revert the last commit
git revert HEAD~3..HEAD     # revert last 3 commits
git revert --no-commit abc123  # apply revert without committing
```
> **Safe** to use on public branches. Creates a NEW commit that undoes changes. Doesn't rewrite history.

---

**Q31. What is the difference between `git reset` and `git revert`?**
> | Feature | git reset | git revert |
> |---|---|---|
> | History | Rewrites history | Adds new commit |
> | Safety | Dangerous on shared branches | Safe on shared branches |
> | Use for | Local, unpublished commits | Public commits |
> **Rule:** Use `revert` for pushed commits, `reset` for local-only commits.

---

**Q32. What is `git bisect`?**
> Binary search through commits to find which commit introduced a bug:
```bash
git bisect start
git bisect bad                  # current commit has the bug
git bisect good v1.0.0          # this version was good

# Git checks out middle commit
# Test it, then tell Git:
git bisect good                 # this commit is fine
# or
git bisect bad                  # this commit has bug

# Git narrows down - repeat until found
git bisect reset                # done, return to original HEAD
```

---

**Q33. What is `git reflog`?**
```bash
git reflog                  # show history of HEAD movements
git reflog show branch-name # history for specific branch
```
> Reflog records every time HEAD moves — branch switch, commit, reset, rebase. Your safety net!
```bash
# Recover lost commits after reset
git reflog                  # find the commit hash before the reset
git reset --hard abc123     # restore to that point
# Or recover deleted branch:
git checkout -b recovered-branch abc123
```

---

**Q34. What is an interactive rebase?**
```bash
git rebase -i HEAD~5        # interactive rebase last 5 commits
```
> Opens an editor where you can:
> - `pick` — keep commit as-is
> - `reword` — keep commit, edit message
> - `edit` — pause to amend commit
> - `squash` — combine with previous commit
> - `fixup` — combine with previous, discard message
> - `drop` — delete commit
> Use to clean up messy history before merging.

---

**Q35. What is `git squash`?**
> Squashing combines multiple commits into one clean commit:
```bash
# Via interactive rebase
git rebase -i HEAD~4
# Change all but first 'pick' to 'squash'

# Via merge
git merge --squash feature/login
git commit -m "feat: add complete login feature"
```
> Keeps main branch history clean — one feature = one commit.

---

**Q36. What is `git worktree`?**
```bash
git worktree add ../hotfix hotfix-branch   # checkout branch in separate dir
git worktree list                           # list worktrees
git worktree remove ../hotfix              # remove worktree
```
> Allows checking out multiple branches simultaneously in different directories. Useful for working on hotfix while keeping current feature branch untouched.

---

**Q37. What is detached HEAD state?**
> When HEAD points directly to a commit instead of a branch pointer:
```bash
git checkout abc123    # detached HEAD - on specific commit not branch
git log --oneline      # shows "(HEAD detached at abc123)"
```
> In detached HEAD: commits are made but not on any branch — can be lost!
> Fix: `git checkout -b new-branch` to create a branch, or `git checkout main` to go back.

---

**Q38. What is `git clean`?**
```bash
git clean -n            # dry run - show what would be removed
git clean -f            # remove untracked files
git clean -fd           # remove untracked files and directories
git clean -fX           # remove ignored files only
git clean -fdx          # remove all untracked + ignored files
```

---

## 3. Remote Repositories

**Q39. What is a remote in Git?**
```bash
git remote -v                               # list remotes
git remote add origin https://github.com/org/repo.git  # add remote
git remote remove origin                    # remove remote
git remote rename origin upstream           # rename remote
git remote set-url origin new-url          # change remote URL
```
> `origin` is the conventional name for the main remote. You can have multiple remotes (e.g., `origin` = your fork, `upstream` = original repo).

---

**Q40. What is `git fetch` vs `git pull`?**
> - `git fetch` — Downloads changes from remote but does NOT merge into local branch. Safe.
> - `git pull` — Downloads AND merges (fetch + merge). Can cause conflicts.
```bash
git fetch origin              # fetch all branches
git fetch origin main         # fetch specific branch
git pull                      # pull current branch
git pull origin main          # pull specific branch
git pull --rebase             # pull and rebase instead of merge
```
> Best practice: Use `git fetch` + review + `git merge` instead of `git pull` blindly.

---

**Q41. What is `git push`?**
```bash
git push origin main                    # push to remote
git push origin feature/login           # push feature branch
git push -u origin feature/login        # set upstream tracking
git push --force-with-lease origin main # safer force push
git push origin --delete feature/login  # delete remote branch
git push origin v1.0.0                 # push tag
git push origin --tags                  # push all tags
```

---

**Q42. What is the difference between `git push --force` and `git push --force-with-lease`?**
> - `--force` — Overwrites remote regardless. Dangerous — can delete others' work.
> - `--force-with-lease` — Only overwrites if nobody else pushed since your last fetch. Safer.
> Use `--force-with-lease` when you need to force push (e.g., after rebase).

---

**Q43. What is upstream tracking?**
```bash
git push -u origin feature/login   # set upstream
# Now you can just: git push / git pull (no need to specify remote/branch)

git branch -vv                     # show tracking info
```

---

**Q44. What is a Pull Request (PR) / Merge Request (MR)?**
> A PR (GitHub) or MR (GitLab) is a request to merge a feature branch into the main branch. It's not a Git feature — it's a platform feature. Enables:
> - Code review by teammates
> - Automated CI/CD checks
> - Discussion and comments
> - Required approvals before merge
> - Audit trail of changes

---

**Q45. What is `git remote prune`?**
```bash
git remote prune origin             # remove stale remote tracking branches
git fetch --prune                   # fetch + prune in one command
git fetch -p                        # shortcut
```
> When remote branches are deleted (after PR merge), local tracking references become stale. Prune removes them.

---

**Q46. How do you fork a repository and contribute?**
```bash
# 1. Fork on GitHub (click Fork button)
# 2. Clone your fork
git clone https://github.com/YOU/repo.git

# 3. Add upstream (original repo)
git remote add upstream https://github.com/ORIGINAL/repo.git

# 4. Create feature branch
git checkout -b feature/my-fix

# 5. Make changes, commit
git add . && git commit -m "fix: my fix"

# 6. Keep your fork up to date
git fetch upstream
git rebase upstream/main

# 7. Push to YOUR fork
git push origin feature/my-fix

# 8. Create Pull Request on GitHub
```

---

**Q47. What is `git archive`?**
```bash
git archive HEAD --format=zip -o project.zip    # zip latest code
git archive v1.0.0 --format=tar.gz -o v1.0.0.tar.gz  # zip tagged version
```
> Export source code without `.git` directory. Good for releases.

---

**Q48. How do you sync a forked repo with the original?**
```bash
git remote add upstream https://github.com/ORIGINAL/repo.git
git fetch upstream
git checkout main
git merge upstream/main     # or rebase
git push origin main
```

---

**Q49. What is `git submodule`?**
> A submodule lets you include another Git repo inside your repo:
```bash
git submodule add https://github.com/lib/awesome-lib.git libs/awesome
git submodule update --init           # init and clone submodules
git submodule update --init --recursive  # including nested
git submodule foreach git pull        # update all submodules
```
> Use case: Including a shared library repo in multiple projects.

---

**Q50. What is `git bundle`?**
```bash
git bundle create repo.bundle --all   # pack entire repo into file
git bundle create latest.bundle main  # bundle only main branch
git clone repo.bundle local-repo      # clone from bundle
git pull repo.bundle main             # pull from bundle
```
> Transfer Git repos over networks without internet access (USB drives, email).

---

**Q51. What is `git shortlog`?**
```bash
git shortlog                    # commits grouped by author
git shortlog -sn                # just summary: count + author name
git shortlog -sn --since="1 month ago"  # last month's contributions
```
> Great for generating changelog or seeing team contribution stats.

---

**Q52. How do you clone a specific tag or commit?**
```bash
# Clone specific tag
git clone --branch v1.0.0 --depth 1 https://github.com/org/repo.git

# Clone then checkout specific commit
git clone https://github.com/org/repo.git
cd repo
git checkout abc123
```

---

## 4. Advanced Git

**Q53. What is `git rerere`?**
> "Reuse Recorded Resolution" — Git remembers how you resolved a conflict and automatically applies it next time.
```bash
git config --global rerere.enabled true
# Now Git records conflict resolutions automatically
```
> Useful when rebasing frequently — same conflict comes up multiple times.

---

**Q54. What are Git hooks?**
> Scripts that Git runs automatically at certain events. Located in `.git/hooks/`.
```bash
# Client-side hooks
pre-commit      # runs before commit (lint, tests)
commit-msg      # validate commit message format
pre-push        # runs before push (integration tests)
post-checkout   # runs after checkout

# Server-side hooks
pre-receive     # runs before receiving push (validate)
post-receive    # runs after receiving push (CI trigger, deploy)
```
```bash
# Example pre-commit hook
#!/bin/bash
npm test
if [ $? -ne 0 ]; then
    echo "Tests failed - commit rejected"
    exit 1
fi
```
> Tools: **Husky** (for Node.js projects) manages hooks in package.json.

---

**Q55. What is `git lfs` (Large File Storage)?**
```bash
git lfs install
git lfs track "*.psd"              # track large files
git lfs track "*.mp4" "*.png"
git lfs ls-files                   # list LFS-tracked files
```
> Git LFS stores large files (videos, binaries, datasets) outside the repo, replacing them with lightweight pointers. Prevents repo from bloating with large files.

---

**Q56. What is `git notes`?**
```bash
git notes add -m "Reviewed by: Alice, Bob" abc123
git notes show abc123
git notes list
```
> Attach metadata to commits without changing commit hash. Not commonly used in practice.

---

**Q57. What is `git grep`?**
```bash
git grep "TODO"                 # search in working directory
git grep "TODO" HEAD            # search in committed files
git grep "TODO" v1.0.0         # search in specific tag
git grep -n "password"         # show line numbers
git grep -l "TODO"             # just filenames
git grep -c "TODO"             # count per file
```
> Faster than `grep -r` because it searches tracked files only.

---

**Q58. What is `git filter-branch` and when would you use it?**
> Rewrites Git history (change commits, remove files from all history). **Deprecated** — use `git filter-repo` instead.
```bash
# Remove accidentally committed file from ALL history
git filter-repo --path sensitive-file.txt --invert-paths

# Remove sensitive data from history
git filter-repo --replace-text replacements.txt
```
> ⚠️ Rewrites all commit hashes — coordinate with team, force push needed.

---

**Q59. How do you find who introduced a bug (git bisect automation)?**
```bash
git bisect start
git bisect bad HEAD
git bisect good v1.0.0

# Automate with test script
git bisect run ./test.sh
# Git automatically finds the first bad commit!

git bisect reset
```

---

**Q60. What is `git worktree` use case?**
> Real-world use: You're working on a feature, urgent hotfix needed:
```bash
# Without worktree: stash changes, switch branch, fix, switch back, pop stash
# With worktree: no context switching!
git worktree add ../hotfix-1.0 hotfix/critical-fix
cd ../hotfix-1.0
# Fix the bug here
git commit -m "fix: critical security patch"
cd ../myfeature
# Continue working on feature!
git worktree remove ../hotfix-1.0
```

---

**Q61. What is `git sparse-checkout`?**
```bash
git clone --sparse https://github.com/org/huge-mono-repo.git
cd huge-mono-repo
git sparse-checkout set services/my-service/
```
> Check out only a subset of a large monorepo. Speeds up clones and operations on huge repositories.

---

**Q62. What is a shallow clone?**
```bash
git clone --depth 1 https://github.com/org/repo.git  # latest commit only
git clone --depth 10 https://github.com/org/repo.git  # last 10 commits
```
> Downloads only recent history. Much faster for CI/CD where you don't need full history. Trade-off: can't go further back than depth.

---

**Q63. How does Git store objects internally?**
> Git has 4 object types:
> - **blob** — file content
> - **tree** — directory listing (maps names to blobs/trees)
> - **commit** — snapshot pointer (points to tree + parent commit + metadata)
> - **tag** — annotated tag object
```bash
git cat-file -t abc123     # type of object
git cat-file -p abc123     # content of object
```

---

**Q64. What is the difference between `git fetch` and `git remote update`?**
> - `git fetch origin` — fetch from `origin` remote
> - `git remote update` — fetch from ALL remotes
> Both download without merging. `remote update` is useful when you have multiple remotes.

---

**Q65. What is `git format-patch` and `git am`?**
```bash
# Export commits as patch files (email/file sharing)
git format-patch -1 HEAD            # last commit as patch
git format-patch main..feature      # all commits between branches

# Apply patches
git am < 0001-fix-bug.patch
git am *.patch
```
> Used in open-source projects where contributors email patches instead of PRs.

---

**Q66. How do you find large files in Git history?**
```bash
# Find large objects in history
git rev-list --objects --all | \
  git cat-file --batch-check='%(objecttype) %(objectname) %(objectsize) %(rest)' | \
  sort -k3 -rn | head -20

# Better tool
git-sizer --verbose
```

---

**Q67. What is `git instaweb`?**
```bash
git instaweb --httpd=webrick    # start web interface
git instaweb --stop
```
> Quick web interface to browse repo locally. Rarely used — GitHub/GitLab are better.

---

**Q68. What is `git describe`?**
```bash
git describe                    # describe current commit relative to tags
# Output: v1.2.0-3-gabc1234
# v1.2.0 = last tag, 3 = commits since tag, gabc1234 = commit hash
git describe --tags
git describe --always           # always output (even without tags)
```
> Great for versioning — generates version strings automatically from tags.

---

**Q69. What is `git blame -C`?**
```bash
git blame -C file.txt           # detect copied/moved code
git blame -C -C file.txt        # more aggressive copy detection
```
> Shows original source file even for code copied from another file. Better than regular blame for tracking code lineage.

---

**Q70. What is Git's object model?**
> ```
> commit "feat: add login"
>   ├── tree (root directory)
>   │   ├── blob src/login.js (file content)
>   │   ├── blob src/auth.js
>   │   └── tree tests/ (subdirectory)
>   │       └── blob tests/login.test.js
>   └── parent: previous commit hash
> ```
> Everything is content-addressed (SHA-1 hash of content = filename in .git/objects).

---

**Q71. What is `git maintenance`?**
```bash
git maintenance start         # start background maintenance tasks
git maintenance run           # run maintenance once
git gc                        # garbage collection (compress objects, clean refs)
git gc --aggressive           # more thorough GC
git prune                     # remove unreachable objects
```
> Keeps repo performing well. `git gc` compresses many small object files into pack files.

---

**Q72. What is `git credential.helper`?**
```bash
git config --global credential.helper store     # store in plain text (not secure)
git config --global credential.helper cache     # cache in memory
git config --global credential.helper manager   # use OS credential manager (recommended)
git config --global credential.helper osxkeychain  # Mac keychain
```
> Avoids entering username/password every time. Use SSH keys or personal access tokens instead of passwords.

---

## 5. Git Workflows

**Q73. What is Git Flow?**
> A branching strategy with strict rules:
```
main (production)
│
├── develop (integration branch)
│   ├── feature/login
│   ├── feature/payment
│   └── feature/dashboard
│
├── release/1.0.0 (stabilization)
└── hotfix/critical-bug (urgent fix)
```
> Flow:
> - Feature branches → develop
> - develop → release branch (testing)
> - release → main + develop
> - hotfix → main + develop
> Best for: Software with versioned releases.

---

**Q74. What is GitHub Flow?**
> Simpler workflow for continuous deployment:
```
main (always deployable)
├── feature/login
├── bugfix/auth-error
└── experiment/new-ui

Flow:
1. Create branch from main
2. Make changes + commit
3. Open PR
4. Code review + CI passes
5. Deploy from branch (optional)
6. Merge to main
7. Deploy main to production
```
> Best for: Web applications with continuous deployment.

---

**Q75. What is Trunk-Based Development?**
> All developers commit directly to `main` (trunk) or very short-lived branches:
```
main ← feature (merged within 1-2 days max)
```
> Key practices:
> - Feature flags to hide incomplete features
> - Small, frequent commits
> - Robust CI/CD
> - Pair programming or short PRs
> Best for: High-performing teams with strong CI/CD.

---

**Q76. What is GitLab Flow?**
> Combines GitHub Flow with environment branches:
```
feature → main → pre-production → production
```
> Or with release branches:
```
feature → main
              └→ release/1.0 (deployed to prod)
              └→ release/2.0
```

---

**Q77. What is a branching strategy and why does it matter?**
> A branching strategy defines rules for:
> - How developers create branches (naming, from where)
> - Where features, fixes, and releases are developed
> - How code flows from dev to production
> - Who can merge to which branches (branch protection)
> Choose based on: team size, deployment frequency, release model.

---

**Q78. How do you enforce branch protection rules?**
> On GitHub/GitLab, protect main/production branches:
> - Require Pull Requests (no direct push)
> - Required number of reviewers (1-2)
> - Required status checks (CI must pass)
> - Dismiss stale approvals on new commits
> - No force pushes
> - No deletions
> This ensures code review and tests before anything reaches production.

---

**Q79. What is a squash merge strategy?**
> All commits in a feature branch are squashed into a single commit when merging:
```
feature: commit1, commit2, commit3, commit4
                       ↓ squash merge
main: single clean commit "feat: complete login feature"
```
> Pros: Clean history on main. Cons: Lose individual commit history.

---

**Q80. What is `--no-ff` and when to use it?**
```bash
git merge --no-ff feature/login
```
> Forces creation of a merge commit even when fast-forward is possible. Preserves the fact that a feature branch existed. Makes it easy to revert entire feature with `git revert -m 1 <merge-commit>`.

---

**Q81. What is semantic versioning (SemVer) in context of Git tags?**
> `MAJOR.MINOR.PATCH` — `v2.1.3`:
> - **MAJOR:** Breaking changes (v1.x → v2.0.0)
> - **MINOR:** New features, backward compatible (v2.0.0 → v2.1.0)
> - **PATCH:** Bug fixes only (v2.1.0 → v2.1.1)
```bash
git tag -a v2.1.3 -m "Fix authentication timeout bug"
git push origin v2.1.3
```

---

**Q82. What is a monorepo vs polyrepo?**
> - **Monorepo:** All services/projects in one Git repo
>   - Examples: Google, Meta, Nx
>   - Tools: Nx, Turborepo, Bazel
>   - Pros: Easy code sharing, atomic commits across services
>   - Cons: Slow CI, complex tooling needed
>
> - **Polyrepo:** Each service has its own Git repo
>   - Pros: Independent CI/CD, team autonomy
>   - Cons: Code sharing harder, dependency management complex

---

**Q83. What is Conventional Commits and why use it?**
> A specification for commit message format that enables:
> - Automatic changelog generation
> - Automatic semantic version bumping
> - Structured history
```
feat!: remove deprecated API endpoints   ← MAJOR (breaking)
feat: add dark mode toggle               ← MINOR
fix: prevent crash on empty cart         ← PATCH
docs: add API documentation
ci: fix broken GitHub Actions workflow
chore: update dependencies
```
> Tools: `commitlint`, `semantic-release`, `standard-version`

---

**Q84. What is `semantic-release`?**
> Fully automated versioning and release publishing:
> 1. Analyzes commit messages since last release
> 2. Determines next version (major/minor/patch)
> 3. Creates Git tag
> 4. Generates CHANGELOG.md
> 5. Publishes release to GitHub/npm
> All based on Conventional Commits format.

---

**Q85. What is a Git alias?**
```bash
git config --global alias.st status
git config --global alias.co checkout
git config --global alias.br branch
git config --global alias.lg "log --oneline --graph --all"
git config --global alias.unstage "reset HEAD --"
git config --global alias.last "log -1 HEAD"

# Usage
git st          # = git status
git lg          # nice log graph
git last        # show last commit
```

---

## 6. Git in DevOps & CI/CD

**Q86. How does Git integrate with CI/CD pipelines?**
> ```
> Developer pushes code to Git
>   → Webhook fires to CI/CD (Jenkins, GitHub Actions, GitLab CI)
>   → Pipeline triggered:
>       1. Clone repository
>       2. Build application
>       3. Run tests
>       4. Build Docker image
>       5. Push to registry
>       6. Deploy to environment
>   → Notify on Slack/email
> ```

---

**Q87. What is a Git webhook?**
> A webhook is an HTTP callback that Git platforms (GitHub, GitLab) send to a URL when events happen:
> - Push to branch
> - Pull request opened/merged
> - Tag created
> - Issue commented
> Jenkins, CircleCI, ArgoCD all listen for webhooks to trigger pipelines.

---

**Q88. How do you use Git in GitHub Actions?**
```yaml
# .github/workflows/ci.yml
name: CI Pipeline

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
    - name: Checkout code
      uses: actions/checkout@v4
      with:
        fetch-depth: 0      # full history for semantic-release

    - name: Get commit info
      run: |
        echo "Branch: ${{ github.ref_name }}"
        echo "Commit: ${{ github.sha }}"
        echo "Author: ${{ github.actor }}"

    - name: Build and test
      run: |
        npm install
        npm test
```

---

**Q89. How do you trigger deployment based on Git tags?**
```yaml
# Deploy only on version tags
on:
  push:
    tags:
      - 'v*.*.*'      # v1.0.0, v2.1.3, etc.

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4
    - name: Get version
      run: echo "VERSION=${GITHUB_REF#refs/tags/}" >> $GITHUB_ENV
    - name: Deploy
      run: ./deploy.sh ${{ env.VERSION }}
```

---

**Q90. What is GitOps?**
> GitOps uses Git as the single source of truth for infrastructure and deployments. The Git repo defines the desired state, and tools (ArgoCD, Flux) continuously sync the actual state to match.
> ```
> Developer changes K8s YAML in Git
>   → PR created → Code reviewed → Merged
>   → ArgoCD detects change
>   → ArgoCD syncs cluster to match Git state
>   → Cluster updated
> ```
> Key principles: Declarative, versioned, automatically applied, continuously reconciled.

---

**Q91. How do you handle secrets in Git repositories?**
> ❌ **Never commit secrets to Git!**
> Prevention:
> - `.gitignore` for `.env` files
> - `git-secrets` tool — scans commits for secrets
> - `detect-secrets` — pre-commit hook
> - `gitleaks` — scan repos for leaked secrets
>
> If secrets are committed:
```bash
# Remove from history immediately
git filter-repo --replace-text replacements.txt
git push --force

# Rotate the exposed credentials immediately!
# Assume it's compromised even if you delete it
```

---

**Q92. What is `git crypt` and `git-secret`?**
> Tools to encrypt sensitive files that need to be in Git:
```bash
# git-crypt
git-crypt init
git-crypt add-gpg-user alice@example.com
# Mark files to encrypt in .gitattributes:
# *.key filter=git-crypt diff=git-crypt
# secrets/* filter=git-crypt diff=git-crypt

git-crypt lock    # encrypt files
git-crypt unlock  # decrypt files
```
> Alternative: Store secrets outside Git (Vault, AWS Secrets Manager) and reference them in config.

---

**Q93. What are protected branches in GitLab/GitHub?**
> Rules that prevent direct pushes to important branches:
> - **No direct push to main** — requires PR/MR
> - **Required reviewers** — minimum 2 approvals
> - **Required CI checks** — tests must pass
> - **No force push** — prevents history rewriting
> - **No deletion** — accidental branch deletion prevention
> ```
> main (protected) ← only via approved PR
> └── feature/* (anyone can push directly)
> ```

---

**Q94. What is CODEOWNERS file?**
```bash
# .github/CODEOWNERS
# Automatically request review from owners when their files are changed

*               @org/all-team          # everyone owns all files
/docs/          @alice                 # alice owns docs
*.tf            @org/devops-team       # devops team owns terraform
/frontend/      @org/frontend-team     # frontend team owns frontend
/api/auth/      @bob @carol            # bob and carol own auth
```

---

**Q95. How do you revert a bad merge to main?**
```bash
# Find the merge commit
git log --oneline --merges

# Revert the merge commit (creates new revert commit)
git revert -m 1 abc123   # -m 1 = keep main parent
# -m 1 means "revert to the first parent" (main branch)

git push origin main

# Note: The feature branch's commits are now "poisoned"
# If you want to re-merge the feature later, you need to revert the revert first!
git revert def456    # revert the revert
```

---

**Q96. How do you set up Git for a new team project?**
```bash
# 1. Initialize repo
git init
echo "# Project Name" > README.md
echo "node_modules/\n.env\ndist/" > .gitignore

# 2. First commit
git add . && git commit -m "chore: initial project setup"

# 3. Add remote and push
git remote add origin https://github.com/org/project.git
git push -u origin main

# 4. Set up branch protection (via GitHub UI or API)

# 5. Set up CODEOWNERS

# 6. Add PR template (.github/PULL_REQUEST_TEMPLATE.md)

# 7. Add CI/CD (.github/workflows/ci.yml)
```

---

**Q97. What is a Git commit signing?**
```bash
# Configure GPG signing
git config --global user.signingkey YOUR_GPG_KEY_ID
git config --global commit.gpgsign true

# Signed commit
git commit -S -m "feat: verified commit"
git log --show-signature    # verify signatures
```
> Signed commits prove the commit was made by the person who owns the GPG key. GitHub shows "Verified" badge.

---

**Q98. How do you handle a large file accidentally committed?**
```bash
# Remove from last commit only (if not pushed)
git rm --cached large-file.bin
echo "large-file.bin" >> .gitignore
git commit --amend

# Remove from entire history (if pushed)
git filter-repo --path large-file.bin --invert-paths
git push --force

# Or use BFG Repo Cleaner (faster for large repos)
bfg --delete-files large-file.bin
git gc && git push --force
```

---

**Q99. What is `git log --follow`?**
```bash
git log --follow -p file.txt    # follow file through renames
git log --follow old-name.js    # see history even before rename
```
> Regular `git log file.txt` stops at renames. `--follow` tracks the file through renames.

---

**Q100. Real-world scenario: Complete Git workflow for a team using GitHub Flow.**
```bash
# Developer workflow
git checkout main
git pull origin main
git checkout -b feature/JIRA-123-user-avatar

# Work on feature
git add .
git commit -m "feat(profile): add user avatar upload"
git commit -m "test(profile): add avatar upload tests"
git commit -m "fix(profile): handle unsupported image formats"

# Push branch
git push -u origin feature/JIRA-123-user-avatar

# Create PR on GitHub
# → CI runs (tests, lint, security scan)
# → Code review by 2 teammates
# → All checks pass
# → Squash and merge to main

# After merge - cleanup
git checkout main
git pull origin main
git branch -d feature/JIRA-123-user-avatar
git push origin --delete feature/JIRA-123-user-avatar

# Auto-deploy to production via CD pipeline triggered on main push
```

---
---