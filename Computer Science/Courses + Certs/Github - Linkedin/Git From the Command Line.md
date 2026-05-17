---
status: done
tags:
  - terminal
  - issue
  - branch
  - commit
  - pr
date: 2026-01-02
---

> [!tip] Terminal
> The command line interface used to interact with Git

# The first Issue

- issues can be created in on the official GitHub page

# Create a Branch

```bash
# Clone a repo
git clone <https link>

# Look at branches
git branch --all

# make new branch
git branch mybranch

# switch to branch
git checkout mybranch
```

# First Commit

```bash
# add change to commit
git add readme.md
git add . # add everything

# create commit and add message
git commit --m "message"
git commit -m "message"

# push to remote repo first time
git push --set-upstream origin mybranch or git push -u origin mybranch

#after first time on branch
git push
```

# First Pull Request

- create in the GitHub page

# Review Pull Request

- review in GitHub