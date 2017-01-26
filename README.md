# stowsh

A shell script to install and uninstall dotfiles using symlinks.
[`stow`](https://www.gnu.org/software/stow/) written in bash, in other words.

## Installation

Put [stowsh](https://raw.githubusercontent.com/williamsmj/stowsh/master/stowsh)
in your path.

Note: `find` and `realpath` must be GNU versions. This is likely already true
if you're on a Linux system, but you may need to `brew install findutils
coreutils` if you're on macOS (and follow the instructions to change your
`$PATH`). [There is an open issue to remove this
dependency](https://github.com/williamsmj/stowsh/issues/13).

## Quickstart

Create a package (a directory of dotfiles):
```bash
$ tree -a
.
├── .bash_history
└── .dotfiles
    ├── .bash_profile
    └── .vimrc
```
Install the package (create symlinks) using `stowsh`
```
$ stowsh ~/.dotfiles
$ tree -a
.
├── .bash_history
├── .bash_profile -> .dotfiles/.bash_profile
├── .dotfiles
│   ├── .bash_profile
│   └── .vimrc
└── .vimrc -> .dotfiles/.vimrc
```
Uninstall the package (delete symlinks) using `stowsh`
```
$ stowsh -D .dotfiles/pkg
```

## Details and options

A package is a directory containing related configuration files. `stowsh`
symlinks the package's contents to the corresponding locations in the current
directory, creating subdirectories as necessary.

```bash
$ stowsh -h
Usage: stowsh [-D] [-n] [-s] [-v] PACKAGE [TARGET]
```

`TARGET` is the destination directory (current directory by default).

 - `-D` uninstall a package
 - `-n` dry-run (print what would happen, but don't do anything)
 - `-v` verbose
 - `-s` skip (skip errors rather than abort)

When installing a package `stowsh` will never overwrite existing files. When
unsintalling a package `stowsh` will never delete files that are not symlinks
to the expected place in the package. 

By default `stowsh` will abort without making _any_ changes if either of these
errors occurs. This is done to avoid being left with a broken half installed
configuration. The `-s` flag can be used to force stowsh to skip these errors
and install/uninstall as much as possible.

One important consequence of the fact that `stowsh` does not delete files that
aren't symlinks to the right place is (deep breath!): if you install a package,
delete a file from the package, then uninstall the package, your target
directory will be left with a broken symlink to the deleted file.

## Use as a dotfiles manager

You can put your dotfiles in one package and install that with `stowsh`
directly.

Or you may prefer to have multiple orthogonal packages that get installed by a
script that uses `stowsh`. This allows you to install packages only if certain
conditions are met. Here's an [example install script that uses
stowsh](https://github.com/williamsmj/dotfiles/blob/master/install.sh).

If your needs go beyond this, there are many [fully-featured dotfiles
managers](https://dotfiles.github.io/).

## Subdirectories

`stowsh` will only create links to files and links. It does not create links to
directories.

If a package contains directories, and the corresponding directories do not
exist in the target, `stowsh` will create real directories (not links).

This choice allows two packages to install files in the same subdirectory, and
files that don't belong in your dotfiles repo (e.g. caches) to be added to
those subdirectories without also being added to your dotfiles repo or its
`.gitignore` file.

```bash
$ tree -a
.
├── .conf
│   └── conf.cache
└── .dotfiles
    ├── pkg1
    │   ├── .conf
    │   │   ├── a.conf
    │   │   ├── b.conf
    │   │   └── c.conf
    │   └── bin
    │       └── script1
    └── pkg2
        └── bin
            └── script2
$ stowsh .dotfiles/pkg1
$ stowsh .dotfiles/pkg2
$ tree -a -I '.dotfiles'  # exclude ./.dotfiles from tree listing
.
├── .conf
│   ├── a.conf -> ../.dotfiles/pkg1/.conf/a.conf
│   ├── b.conf -> ../.dotfiles/pkg1/.conf/b.conf
│   ├── c.conf -> ../.dotfiles/pkg1/.conf/c.conf
│   └── conf.cache
└── bin
    ├── script1 -> ../.dotfiles/pkg1/bin/script1
    └── script2 -> ../.dotfiles/pkg2/bin/script2
```

Things to note here: 

 - `~/bin` was created by `stowsh`. It's a real directory, not a link.
 - both `pkg1` and `pkg2` install files into `~/bin`
 - the directory `.conf` existed before `pkg1` was installed
 - `conf.cache` is a real file that exists alongside the symlinks installed by
   `stowsh`

When uninstalling a package, subdirectories will only be deleted if they are
empty. So:

```bash
$ stowsh -D .dotfiles/pkg1
$ stowsh -D .dotfiles/pkg2
$ tree -a -I '.dotfiles'
.
└── .conf
    └── conf.cache
```

## What's wrong with GNU stow?

`stowsh` is a short, cross-platform bash script without dependencies. `stow` is
implemented as a Perl module. Neither of us are smart enough to install Perl
modules, and we'd rather not have Perl as a dependency.

## Authors

[Mike Lee Williams](https://github.com/williamsmj/) and [Micha
Gorelick](https://github.com/mynameisfiber/). Issues and PRs welcome.
