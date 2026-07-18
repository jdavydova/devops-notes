# Git: `fetch` vs `pull`

## Overview

`git fetch` and `git pull` are related, but they do different things.

  -------------------------------------------------------------------------
  Command       What it does         Changes your working files?
  ------------- -------------------- --------------------------------------
  `git fetch`   Downloads the latest ❌ No
                changes from the     
                remote repository    

  `git pull`    Downloads changes    ✅ Yes
                and merges (or       
                rebases) them into   
                your current branch  
  -------------------------------------------------------------------------

## `git fetch`

``` bash
git fetch origin
```

This command:

-   Connects to GitHub.
-   Downloads new commits and branch information.
-   Updates remote-tracking branches like `origin/main`.
-   **Does not** modify your current branch or files.

### Example

Before:

``` text
GitHub (main): A --- B --- C
Local (main):  A --- B
```

After `git fetch`:

``` text
origin/main: A --- B --- C
local main:  A --- B
```

Your working directory stays exactly the same.

Inspect new commits:

``` bash
git log HEAD..origin/main --oneline
```

See differences:

``` bash
git diff main origin/main
```

## `git pull`

``` bash
git pull origin main
```

This is roughly equivalent to:

``` bash
git fetch origin
git merge origin/main
```

It first downloads the changes, then immediately merges them into your
current branch.

### Example

Before:

``` text
local main:  A --- B
origin/main: A --- B --- C
```

After:

``` text
local main:  A --- B --- C
```

Your local files may change because the new commits are applied.

## When to use each

### Use `git fetch` when you want to:

-   Check for updates without changing your work.
-   Review incoming commits before merging.
-   Safely inspect remote changes.

### Use `git pull` when you want to:

-   Update your current branch immediately.
-   Merge the latest remote changes into your local branch.

## Recommended workflow

``` bash
git fetch origin
git status
git log --oneline HEAD..origin/main
```

If everything looks good:

``` bash
git merge origin/main
```

Or simply:

``` bash
git pull
```

## Summary

-   **`git fetch`** = Download changes only.
-   **`git pull`** = Download changes and merge them into your current
    branch.
-   For learning Git, using **`git fetch`** first is recommended because
    it lets you inspect changes before updating your local branch.
