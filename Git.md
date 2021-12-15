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
