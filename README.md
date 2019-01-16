# sysgit - git based Debian provisioning toolkit

Let's manage and deploy Debian systems with the help of a system-wide
git repository.

sysgit provides some helpful commands and git hooks for this task.
They are tested on *Debian* (real hosts and VM), *Proxmox* and *Raspbian*.

## Initialize

### System variants:

We provide some basic system variants:

* **base** - Just the init script and the git hooks (see [repo](https://github.com/simonwalz/sysgit-variants/tree/base))
* **minimal** - Base system with sysgit repository as submodule (see [repo](https://github.com/simonwalz/sysgit-variants/tree/minimal))
* **full** - Anything helpful (see [repo](https://github.com/simonwalz/sysgit-variants/tree/full))

### Initialize from an empty repository

First, init the system-wide git repository:
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
git push -u origin master
```


### Initialize from existing repository

First, init the system-wide git repository:
```sh
git init /
```

Add your remote:
```sh
git remote add origin GIT_URL
git fetch origin
git branch --track "master" "origin/master"
```

Use either:
```sh
git reset --mixed && git checkout-index -a
# or
git reset --hard && git checkout-index -a
```
to restore your files. `--mixed` only write non-existing files.  `--hard` overwrites all files.


And run the init script:
```sh
/.sysgit/init.sh
```

### Port an other system-wide git not managed via sysgit:

The git merge command fails if we try to merge an other repository.

Thus use the commands in "Initialize from an empty repository" and
 use the following command insteed of the merge command:

```sh
# git rebase --onto upstream/<VARIANT> --root master
# i.e.
git rebase --onto upstream/minimal --root master

# force push:
git push -f
```

Or cherry-pick the commits you what.

## Working

The following tools are automatically installed in **minimal** and **full** variants via a submodule and linked into path (by `/.sysgit/init.sh`). You can use these tools without the system variants; you only need to add all executables to the path.

### Use git

See [git documentation](https://git-scm.com/docs/). Use `git add FILE`, `git commit -m "Message"` and `git push`.

### Command: `git sys-status`

Show git status WITHOUT unchanged Debian system-files.

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

Show differences of a file to version of the Debian package:

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

You can omit the variant parameters if you saved it in the past with
```sh
git sys-update --save <VARIANT> [<VARIANT2> ...]
```


Background: The command executes the following git commands:
```sh
cd /
git fetch upstream
git merge upstream/<VARIANT>
```

## Internal

### Command: `apt-user-installed` (internal)

List all user installed packages (or write them to `/etc/sysgit/apt-packages`)

In minimal system variant, this script is automatically called by a apt hook.

#### Options:

* `--list` - Show list of installed packages
* `--save` - Save list to `/etc/sysgit/apt-packages`
* `--install` - Install Packages listed in `/etc/sysgit/apt-packages`

