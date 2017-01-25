# stowsh

A shell script to install and uninstall dotfiles using symlinks.
[`stow`](https://www.gnu.org/software/stow/) written in bash, in other words.

## Quickstart

Create a directory of dotfiles:
```bash
$ ls -a ~/.dotfiles/bash
./  ../  .bash_aliases  .bash_profile  .bashrc
```
`stowsh` it:
```
$ stowsh ~/.dotfiles/bash
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

A package is a directory containing related configuration files. `stowsh`
symlinks its contents to the corresponding locations in `$HOME`, creating
subdirectories as necessary.

You can gather a set of packages in a dotfiles repository. For an example of
this, and an [install script that uses
`stowsh`](https://github.com/williamsmj/dotfiles/blob/master/install.sh), see
[my dotfiles](https://github.com/williamsmj/dotfiles).

```bash
$ stowsh -h
Usage: stowsh [-D] [-n] [-f] [-v] PACKAGE [TARGET]
```

`TARGET` is the destination directory (`$HOME` by default).

 - `-D` uninstall a package
 - `-n` dry-run (print what would happen, but don't do anything)
 - `-v` verbose
 - `-s` skip (skip install or uninstall errors rather than abort)

When installing a package `stowsh` will never overwrite existing files. When
unsintalling a package `stowsh` will never delete files that are not symlinks
to the expected place in the package. 

By default `stowsh` will abort without making _any_ changes if either of these
errors occurs. This is done to avoid being left with a broken half installed
configuration. The `-s` flag can be used to force stowsh to install/uninstall
as much as possible.

One important consequence of this behaviour is that if you install package,
then delete a file from the package, then use `stowsh` to uninstall the
package, your target directory will be left with a broken symlink to the
package.

## Subdirectories

`stowsh` does not create links to directories.

If a package contains subdirectories, and the corresponding directories do not
exist in the target, `stowsh` will create real directories (not links). So:

```
$ cd ~
$ tree -a .dotfiles/bin
.dotfiles/bin
└── .config
    ├── a.conf
    ├── b.conf
    ├── c.conf
    └── d.conf
$ stowsh .dotfiles/bin
$ tree -a ~/bin
~/bin
├── a.conf -> ../.dotfiles/bin/bin/a.conf
├── b.conf -> ../.dotfiles/bin/bin/b.conf
├── c.conf -> ../.dotfiles/bin/bin/c.conf
└── d.conf -> ../.dotfiles/bin/bin/d.conf
```

Note `~/bin` is a real directory, not a link.

This choice allows two packages to install files in the same subdirectory, and
files that don't belong in your dotfiles repo (e.g. caches) to be added to
those subdirectories without also being added to your dotfiles repo or its
`.gitignore` file.

When uninstalling a package, subdirectories will only be deleted if they are
empty.

## Installation

Put [stowsh](https://raw.githubusercontent.com/williamsmj/stowsh/master/stowsh) in
your path.

## Bootstrapping

`source` the script and use the `stowsh_install` and `stowsh_uninstall` functions
directly. These functions both take one required argument, the package path,
and one optional argument, the target path (which defaults to `$HOME` if not
given). e.g.

```
$ source <(curl https://raw.githubusercontent.com/williamsmj/stowsh/master/stowsh)
$ stowsh_install ~/.dotfiles/bash
```

## What's wrong with GNU stow?

`stowsh` is a short, cross-platform bash script without dependencies. `stow` is
implemented as a Perl module. Neither of us are smart enough to install Perl
modules, and we'd rather not have Perl as a dependency.

## Authors

[Mike Lee Williams](https://github.com/williamsmj/) and [Micha
Gorelick](https://github.com/mynameisfiber/). Issues and PRs welcome.
