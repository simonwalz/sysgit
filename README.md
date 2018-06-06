# sysgit - git based debian provisioning toolkit

Let's manage and deploy debian systems with the help of a system-wide
git repository.

sysgit provides some helpfull commands and git hooks for this task.

## Initialize

### System variants:

We provide some basic system variants:

* **base** - Just the init script and the git hooks
* **minimal** - Base system with sysgit repository as submodule
* **full** - Anything helpfull

### Initialize from an empty repository

First, init the system-wide git repo:
```
git init /
```

Merge our system variant into it. Choose a variant, see above.
```
cd /
git remote add upstream https://github.com/simonwalz/sysgit-variants.git
git fetch upstream
# git merge upstream/<VARIANT>
# i.e.
git merge upstream/minimal
```

And run the init script. This installs the git hooks and initialize the submodules:
```sh
/.sysgit/init.sh
```


(optional) Add your remote:
```
git remote add origin GIT_URL
git checkout -b master --track origin/master
git push
```


### Initialize from existing repository

First, init the system-wide git repo:
```sh
git init .
```

Add your remote:
```sh
git remote add origin GIT_URL
git checkout -b master --track origin/master
```

Use either:
```sh
git reset --mixed origin/master 
# or
git reset --hard origin/master
```
to restore your files. `--mixed` only write non-existing files.  `--hard` overwrites all files.


And run the init script:
```sh
/.sysgit/init.sh
```

### Port an other system-wide git not managed via sysgit:

The git merge command fails if we try to merge an other repo.

Thus use the commands in "Initialize from an empty repository" and
 use the following commands to rebase your existing commits:

```sh
git remote add old GIT_URL
git fetch old
# git rebase --onto upstream/<VARIANT> --root old/master
# i.e.
git rebase --onto upstream/minimal --root old/master

# force push:
git push -f
```

Or cherry-pick the commits you what.

## Working

The following tools are automatically installed in **minimal** and **full** variants via a submodule and linked into path (by `/.sysgit/init.sh`). You can use these tools without the system variants; you only need to add all executables to the path.

### Use git

See [git documentation](https://git-scm.com/docs/). Use `git add FILE`, `git commit -m "Message"` and `git push`.

### Command: `git sys-status`

Show git status WITHOUT unchanged debian system-files.

Usage: See [git status](https://git-scm.com/docs/git-status).

This tool runs `git sys-ignore --update` before showing the status.

### Command: `git sys-ignore-gen` (internal use)

Update the git exclude file (`/.git/info/exclude`).
This is automatically executed before showing `git sys-status`.

### Command: `git sys-ignore`

Configure which files shall be ignored.

#### Usage: List pattern:

```sh
git sys-ignore
```

#### Usage: Add pattern:
```sh
git sys-ignore PATTERN
```

#### Example:
```sh
git sys-ignore 110 120 130
```

To remove a pattern, edit /etc/sysgit/ignore-config


### Command: `git sys-ignore-untracked`

Index all untracked files. If you change one of these files, it will
show up in `git sys-status`.

The index is saved to `/etc/sysgit/local-config.ignore-chk`

### Command: `git sys-diff`

Show differences of a file to version of the debian package:

#### Usage:
```sh
git sys-diff debian-system-file
```


### Command: `git sys-update`: update variant

Get commits from one (or multiple) foreign repository (of remote `upstream`):

```sh
git sys-update [<VARIANT> [<VARIANT2> ...]]
```

Example
```sh
git sys-update minimal ansible
```

You can obmit the variant parameters if you saved it in the past with
```sh
git sys-update --save <VARIANT> [<VARIANT2> ...]
```


Background: The command executes the following git commands:
```sh
cd /
git fetch upstream
git merge upstream/<VARIANT>
```

