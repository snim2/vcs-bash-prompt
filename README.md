vcs-bash-prompt
===============

My custom bash prompt using Version Control Systems (VCS) such as: git, hg, ecc...


Introduction
------------

When I work with VCS at bash command line, on my favorite GNU/Linux distribution, I find useful a custom prompt that display information about the VCS used in a project.

When I am in any directory of a project's directory tree the custom prompt display me:
* which VCS is being used
* which branch I am on

Many thanks to this [Tom Ryder's post](http://blog.sanctum.geek.nz/bash-prompts/).


Code
----

At the end of my `.bashrc` file I have added this four functions:

```
prompt_git() {
    $(git rev-parse --is-inside-git-dir 2>/dev/null ) \
        && return 1
    $(git rev-parse --is-inside-work-tree 2>/dev/null ) \
        || return 1
    git status &>/dev/null
    branch=$(git symbolic-ref --quiet HEAD 2>/dev/null ) \
        || branch=$(git rev-parse --short HEAD 2>/dev/null ) \
        || branch='unknown'
    branch=${branch##*/}
    git diff --quiet --ignore-submodules --cached \
        || state=${state}+
    git diff-files --quiet --ignore-submodules -- \
        || state=${state}!
    $(git rev-parse --verify refs/stash &>/dev/null ) \
        && state=${state}^
    [ -n "$(git ls-files --others --exclude-standard )" ] \
        && state=${state}?
    printf '(git:%s)' "${branch:-unknown}${state}"
}
prompt_hg() {
    hg branch &>/dev/null || return 1
    BRANCH="$(hg branch 2>/dev/null)"
    [[ -n "$(hg status 2>/dev/null)" ]] && STATUS="!"
    printf '(hg:%s)' "${BRANCH:-unknown}${STATUS}"
}
prompt_svn() {
    svn info &>/dev/null || return 1
    URL="$(svn info 2>/dev/null | \
        awk -F': ' '$1 == "URL" {print $2}')"
    ROOT="$(svn info 2>/dev/null | \
        awk -F': ' '$1 == "Repository Root" {print $2}')"
    BRANCH=${URL/$ROOT}
    BRANCH=${BRANCH#/}
    BRANCH=${BRANCH#branches/}
    BRANCH=${BRANCH%%/*}
    [[ -n "$(svn status 2>/dev/null)" ]] && STATUS="!"
    printf '(svn:%s)' "${BRANCH:-unknown}${STATUS}"
}
prompt_vcs() {
    prompt_hg || prompt_git || prompt_svn 
}
```

Because these functions call your VCS program (git, hg, svn) you have to add your VCS to your `PATH` variable.

Then I have looked for `PS1` in my `.bashrc` and I have changed the setting to the following one where I have introduced the call to the `prompt_vcs` function:
```
PS1="${debian_chroot:+($debian_chroot)}\u@\h:\!:\w$(prompt_vcs)\$ "
```

That is all.


Adding some color
-----------------

Probably your `.bashrc` lets you choose a colored prompt. In that case simply introduce the call to the `prompt_vcs` function to both the `PS1` setting lines. The example below includes some additional information, such as the number of the last job which was run and its return code:

```
if [ "$color_prompt" = yes ]; then
    color_reset=$(tput sgr0)
    color_bold=$(tput bold)
    color_white=$(tput setaf 7)
    color_jobs=$(tput setaf 7)
    color_user=$(tput setaf 2)
    color_vcs=$(tput setaf 3)
    color_dir=$(tput setaf 4)
    color_tick=$(tput setaf 2)
    color_cross=$(tput setaf 1)
    sep=$(tput setaf 7)\:
    PS1="${debian_chroot:+($debian_chroot)}${color_bold}${color_user}\[\u\]${color_jobs}${sep}\[\!$\]${sep}(\`if [[ \$? == 0 ]]; then echo \"${color_tick}\[\342\234\223\]\"; else echo \"${color_cross}\[\342\234\227\]\"; fi\`${color_white})${sep}${color_dir}\[\w\$]{sep}${color_vcs}\[$(prompt_vcs)\]${color_reset}\$ "
else
    PS1="${debian_chroot:+($debian_chroot)}\[\u\]:\[\!\]:\[\w\]\[$(prompt_vcs)\]\$ "
fi
```

In the following screenshot you can see my custom prompt when I work on `JugEvents3` with hg and when I work on `SemVer` with git:

![Screenshot](/screenshot.png)
