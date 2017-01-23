# shtow

A shell script to install and uninstall dotfiles using symlinks.
[`stow`](https://www.gnu.org/software/stow/) written in bash, in other words.

## Quickstart

Create a directory of dotfiles:
```bash
$ ls -a ~/.dotfiles/bash
./  ../  .bash_aliases  .bash_profile  .bashrc
```
`shtow` it:
```
$ shtow ~/.dotfiles/bash
```
Symlinks to those files are created in `$HOME`:
```
$ ls -altr ~/.bash* | tail -4
lrwxr-xr-x    1 mike staff    22 Jan 22 09:11 .bashrc -> .dotfiles/bash/.bashrc
lrwxr-xr-x    1 mike staff    28 Jan 22 09:11 .bash_profile -> .dotfiles/bash/.bash_profile
-rw-------    1 mike staff 85604 Jan 22 09:11 .bash_history
lrwxr-xr-x    1 mike staff    28 Jan 22 09:11 .bash_aliases -> .dotfiles/bash/.bash_aliases
```

## More usage

A package is a directory containing related configuration files. `shtow`
symlinks its contents to the corresponding locations in `$HOME`, creating
subdirectories as necessary.

You can gather a set of packages in a dotfiles repository. For an example of
this, and an [install script that uses
`shtow`](https://github.com/williamsmj/dotfiles/blob/master/install.sh), see
[my dotfiles](https://github.com/williamsmj/dotfiles).

```bash
$ shtow -h
Usage: shtow [-D] [-n] [-f] [-v] PACKAGE [TARGET]
```

`TARGET` is the destination directory (`$HOME` by default).

 - `-D` uninstall a package
 - `-n` dry-run (print what would happen, but don't do anything)
 - `-v` verbose
 - `-f` force (overwrite existing files, or delete files that do not link to `package`)

When installing a package `shtow` will never overwrite existing files. Unless
the `-f` switch is set, `shtow` will abort without making _any_ changes if
existing files would get in the way. This is done to avoid being left with a
broken half installed configuration.

When unsintalling a package `shtow` will never delete files that are not
symlinks to the expected place in the package. Again, unless the `-f` switch is
set, `shtow` will abort without making _any_ changes if such files are found.

## Subdirectories

`shtow` does not create links to directories.

If a package contains directories, and the corresponding directories do not
exist in the target, `shtow` will create real directories (not links). 

This choice allows two packages to install files in the same subdirectory, and
for files that don't belong in your dotfiles (e.g. caches) to live in the same
directory without being added to your dotfiles `.gitignore`.

When uninstalling a package subdirectories will only be deleted if they are
empty.

## Installation

Put [shtow](https://raw.githubusercontent.com/williamsmj/shtow/master/shtow) in
your path.

## Bootstrapping

`source` the script and use the `shtow_install` and `shtow_uninstall` functions
directly. These functions both take one required argument, the package path,
and one optional argument, the target path (which defaults to `$HOME` if not
given). e.g.

```
$ source <(curl https://raw.githubusercontent.com/williamsmj/shtow/master/shtow)
$ shtow_install ~/.dotfiles/bash
```

## What's wrong with GNU stow?

`shtow` is a short, cross-platform bash script without dependencies. `stow` is
implemented as a Perl module. Neither of us are smart enough to install Perl
modules, and we'd rather not have Perl as a dependency.

## Authors

[Mike Lee Williams](https://github.com/williamsmj/) and [Micha
Gorelick](https://github.com/mynameisfiber/). Issues and PRs welcome.
