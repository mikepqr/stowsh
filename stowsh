#!/usr/bin/env bash

_runcommands() {
    if [[ $DRYRUN == 1 ]] || [[ $VERBOSE -gt 1 ]]; then
        echo "$@"
    fi
    if [[ $DRYRUN != 1 ]]; then
        eval "$@"
    fi
}

echoerr() {
    printf "%s\n" "$*" >&2;
}

deperr() {
    echoerr "stowsh requires $1"
}

isgnu() {
    if $1 --version >/dev/null 2>&1 ; then
        return 1
    else
        return 0
    fi
}

stowsh_setpaths() {
    if command -v grealpath >/dev/null 2>&1; then
        rpcmd="grealpath"
    elif command -v realpath >/dev/null 2>&1; then
        rpcmd="realpath"
    else
        deperr "GNU coreutils"
        return 1
    fi
    if isgnu $rpcmd; then
        deperr "GNU coreutils"
        return 1
    fi
    if command -v gfind >/dev/null 2>&1; then
        findcmd="gfind"
    elif command -v find >/dev/null 2>&1; then
        findcmd="find"
    else
        deperr "GNU findutils"
        return 1
    fi
    if isgnu $findcmd; then
        deperr "GNU findutils"
        return 1
    fi
}

stowsh_install() {
    stowsh_setpaths || return 1
    local pkg=$1
    local target
    target=$( "$rpcmd" "${2-$PWD}" )
    local commands=()

    cd "$pkg" || return 1

    while IFS= read -r -d '' d; do
        commands+=("mkdir -p '$target/$d'")
    done < <($findcmd . -mindepth 1 -type d -print0 | sed "s|./||")

    while IFS= read -r -d '' f; do
        local targetf="$target/$f"
        local thisdir
        thisdir=$(dirname "$targetf")
        local relative
        relative=$($rpcmd "$f" --relative-to="$thisdir" --canonicalize-missing)
        if [[ ! -f "$targetf" ]] ; then
            commands+=("ln -s '$relative' '$targetf'")
        else
            echoerr "$targetf already exists."
            if [[ ! $SKIP -eq 1 ]]; then
                echoerr "Aborting. Rerun with the -s flag to skip errors.";
                return
            fi
        fi
    done < <($findcmd . -type f -print0 -or -type l -print0 | sed "s|./||")

    for cmd in "${commands[@]}"; do
        _runcommands "$cmd"
    done;
}

stowsh_uninstall() {
    stowsh_setpaths || return 1
    local pkg=$1
    local target
    target=$( "$rpcmd" "${2-$PWD}" )
    local commands=()

    cd "$pkg" || return 1

    while IFS= read -r -d '' f; do
        local targetf="$target/$f"
        if [[ $($rpcmd "$targetf") == $($rpcmd "$f") ]] ; then
            commands+=("rm '$targetf'")
        elif [[ -f "$targetf" ]] ; then
            echoerr "$targetf does not point to to $($rpcmd "$f")."
            if [[ ! $SKIP -eq 1 ]]; then
                echoerr "Aborting. Rerun with the -s flag to skip errors.";
                return
            fi
        elif [[ ! -f "$targetf" ]] ; then
            echoerr "$targetf does not exist. Nothing to do."
            if [[ ! $SKIP -eq 1 ]]; then
                echoerr "Aborting. Rerun with the -s flag to skip errors.";
                return
            fi
        fi
    done < <($findcmd . -type f -print0 -or -type l -print0 | sed "s|./||")

    while IFS= read -r -d '' d; do
        commands+=("[[ -d '$target/$d' ]] && $findcmd '$target/$d' -type d -empty -delete")
    done < <($findcmd . -mindepth 1 -type d -print0 | sed "s|./||")

    for cmd in "${commands[@]}"; do
        _runcommands "$cmd"
    done;
}

stowsh_help() {
    echo "Usage: $0 [-D] [-n] [-s] [-v[v]] [-t TARGET] PACKAGES..."
}

if [ "$0" = "$BASH_SOURCE" ]; then
    UNINSTALL=0
    DRYRUN=0
    SKIP=0
    TARGET="$PWD"
    PACKAGES=()
    while [[ "$@" ]]; do
        if [[ $1 =~ ^- ]]; then
            OPTIND=1
            while getopts ":vhDsnt:" opt; do
                case $opt in
                    h)
                        stowsh_help
                        exit 0
                        ;;
                    D)  UNINSTALL=1
                        ;;
                    n)  DRYRUN=1
                        ;;
                    s)  SKIP=1
                        ;;
                    v)  VERBOSE=$((VERBOSE + 1))
                        ;;
                    t)  TARGET="$OPTARG"
                        ;;
                    *)  echo "'$OPTARG' is an invalid option/flag"
                        exit 1
                        ;;
                esac
            done
            shift $((OPTIND-1))
        else
            PACKAGES+=("$1")
            shift
        fi
    done

    if [[ ${#PACKAGES[@]} -eq 0 ]] ; then
        stowsh_help
        exit 1
    fi

    for i in ${!PACKAGES[*]}; do
        pkg=${PACKAGES[$i]}
        if [[ $UNINSTALL -eq 1 ]]; then
            if [[ $VERBOSE -gt 0 ]] ; then echoerr "Uninstalling $pkg from $TARGET" ; fi
            stowsh_uninstall "$pkg" "$TARGET"
        else
            if [[ $VERBOSE -gt 0 ]] ; then echoerr "Installing $pkg to $TARGET" ; fi
            stowsh_install "$pkg" "$TARGET"
        fi
    done
fi
