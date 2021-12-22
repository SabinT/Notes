## Uncommit last commit and keep changes locally

```
git reset HEAD~1 --soft
```

## Merge conflicts in multiple files with ours/theirs

E.g., change all `.meta` files to current version.

```
git checkout --ours -- *.meta
```

Or, choose to keep the version from the branch being pulled/merged

```
git checkout --theirs -- *.meta
```

## Update a branch to match remote without having to stash/checkout/switch/stash

Say you're on branch `feature`, but want to update local `main` to remote `main` without switching branches:

```
git fetch origin main:main
```
