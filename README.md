# stowsh

[![Build Status](https://travis-ci.org/williamsmj/stowsh.svg?branch=master)](https://travis-ci.org/williamsmj/stowsh)

A shell script to install and uninstall dotfiles using symlinks.
[`stow`](https://www.gnu.org/software/stow/) written in bash, in other words.

## Installation

Put [stowsh](https://raw.githubusercontent.com/williamsmj/stowsh/master/stowsh)
in your path.

## Dependencies

`stowsh` strives for minimal dependencies, but it currently depends on the GNU
coreutils implementation of `realpath` and GNU findutils.

These are already installed on most Linux systems. On macOS you can install
them with `brew install findutils coreutils`. 

Unfortunately older Debian-based systems (including Ubuntu Trusty 14.04) do not
include the GNU implementation of `realpath` in their coreutils package. See
[below](#stowsh-on-ubuntu-trusty-1404) for a solution.

I would welcome PRs to [remove these dependencies and replace them with POSIX
shell commands](https://github.com/williamsmj/stowsh/issues/14)!

## Quickstart

Create a package (a directory of dotfiles, `~/.dotfiles` in this example):
```bash
$ cd
$ tree -a
.
└── .dotfiles
    ├── .bash_profile
    └── .vimrc
```
Install the package (create symlinks in the current directory) using `stowsh`
```
$ stowsh ~/.dotfiles
$ tree -a
.
├── .bash_profile -> .dotfiles/.bash_profile
├── .dotfiles
│   ├── .bash_profile
│   └── .vimrc
└── .vimrc -> .dotfiles/.vimrc
```
Uninstall the package (delete symlinks) using `stowsh`
```
$ stowsh -D ~/.dotfiles
```

## Details and options

A package is a directory containing related configuration files. `stowsh`
symlinks the package's contents to the corresponding locations in the current
directory, creating subdirectories as necessary.

```bash
$ stowsh -h
Usage: stowsh [-D] [-n] [-s] [-v[v]] [-t TARGET] PACKAGES...
```

`TARGET` is the destination directory (current directory by default).

 - `-D` uninstall a package
 - `-n` dry-run (print what would happen, but don't do anything)
 - `-v` verbose (`-vv` is even more verbose)
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
$ stowsh .dotfiles/pkg1 .dotfiles/pkg2
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
$ stowsh -D .dotfiles/pkg1 .dotfiles/pkg2
$ tree -a -I '.dotfiles'
.
└── .conf
    └── conf.cache
```

## stowsh on Ubuntu Trusty 14.04

Older Debian-based systems (including Ubuntu Trusty 14.04) do not include the
GNU implementation of `realpath` in their coreutils package.

Assuming you aren't able to switch operating system (e.g. you're using Travis
CI), you need to install a newer version of coreutils. coreutils contains many
fundamental utilities (cut, true, kill, etc.), and it's probably not a good
idea to upgrade the system version.

The simplest solution is to install coreutils with a prefix:

```bash
wget http://ftp.gnu.org/gnu/coreutils/coreutils-8.28.tar.xz
tar xf coreutils-8.28.tar.xz
cd coreutils-8.28
./configure --program-prefix=g
make && sudo make install
```

Built and installed like this `cut` (or any other coreutils command) will still
be your operating system's regular `cut`, and `gcut` will be the new coreutils
version you just installed.

stowsh will look for `grealpath` and use it in preference to `realpath` if
found.

## What's wrong with GNU stow?

`stowsh` is a short, cross-platform bash script without dependencies. `stow` is
implemented as a Perl module. I'm not smart enough to install Perl modules, and
I'd rather not have Perl as a dependency.

## Author

[Mike Lee Williams](https://github.com/williamsmj/). Issues and PRs welcome.
