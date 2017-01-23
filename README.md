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
Usage: shtow [-D] package [target]
```

`target` is the destination directory (`$HOME` by default). The `-D` switch
uninstalls a package.

When installing a package `shtow` will not overwrite existing files. When
uninstalling it will not delete files unless they are links pointing to the
expected place in the package. Any other files are skipped.

If a package contains directories, and the corresponding directories do not
exist in the target, `shtow` will create them. When uninstalling a package
these directories will only be deleted if they are empty.

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
implemented as a Perl module, which means Perl is a dependency. Also I am not
smart enough to install Perl modules.

## Author

[Mike Lee Williams](http://mike.place). Issues and PRs welcome.
