+++
categories = ["posts"]
date = "2015-07-29T22:47:14+01:00"
title = "git"
+++

A commit...

- has only one purpose
- has a limited scope
- has an adequate commit message
- is complete
- is consistent
- gets reviewed
- is traceable
- can be reverted

## Contribution Workflow

Clone a project on Github:
```bash
# Clone <project> on github
git clone https://github.com/<yourname>/<project>
git remote add upstream https://github.com/<originalauthor>/<project>
```

Create a feature branch:
```bash
git checkout -b <name>
```

List all branches:
```bash
git branch -l
```

Keep feature branch up to date:
```bash
git fetch upstream master
git rebase upstream/master
```

Commit signed-off:
```bash
git commit -s -m '...'
```

Force push:
```bash
git push -f --set-upstream origin <feature-branch>
```

Undo last n commits:
```bash
git reset --soft HEAD~<n>
```

Add files to last commit or change the last commit message:
```bash
(git add .)
git commit --amend
```

Rebase last n commits:
```bash
git rebase -i HEAD~<n>
```

## Other Tricks

Checkout a commit before a given date:
```bash
git checkout `git rev-list -n 1 --before="2015-03-19 10:00" master`
```

Show how many lines changed between two commits:
```bash
git diff --stat <commit1> <commit2>
```

Show how many lines changed between the parent of `HEAD` and `HEAD`:
```bash
git diff --stat HEAD^ HEAD
```

Stash only some files:
```bash
git add <files you want to keep>
git stash --keep-index
```

Create and apply a patch of the topmost n commits from a given hash:
```bash
git format-patch -<n> <SHA1>
patch -p1 < <patch file>
```

Force git to overwrite local files on pull:
```bash
git fetch --all
git reset --hard origin/master
```

Show the total number of commits:
```bash
git rev-list HEAD --count
```

Show the number of commits by author:
```bash
git shortlog -sn --all
```

Remove mistakenly added files which are in `.gitignore`:
```bash
git rm --cached `git ls-files -i --exclude-from=.gitignore`
```

Remove a tag:
```bash
git tag -d <tag>
git push origin :refs/tags/<tag>
```
