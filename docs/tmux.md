# `tmux`

## Exporting a Variable

[tmux is exporting an environment variable that is no longer being exported in .bashrc](https://superuser.com/questions/1216492/tmux-is-exporting-an-environment-variable-that-is-no-longer-being-exported-in-b)

```sh
tmux set-environment -gru FZF_TMUX_OPTS
```

```
set-environment [-Fhgru] [-t target-session] name [value]
    (alias: setenv)
  Set or unset an environment variable.
  If -g is used, the change is made in the global environment; otherwise, it is applied to the session environment for target-session.
  If -F is present, then value is expanded as a format.
  The -u flag unsets a variable.
  -r indicates the variable is to be removed from the environment before starting a new process.
  -h marks the variable as hidden.
```

## Find a Panes Running a Command

```
find-window [-iCNrTZ] [-t target-pane] match-string
    (alias: findw)
  Search for a fnmatch(3) pattern or, with -r, regular expression match-string in window names, titles, and visible content (but not history).
  The flags control matching behavior: -C matches only visible window contents, -N matches only the window name and -T matches only the window title.
  -i makes the search ignore case.
  The default is -CNT.
  -Z zooms the pane.
```

This command works only if at least one client is attached.

```tmux
CTRL+b + f
<SEARCH STRING>
```

## Rename a pane

```tmux
set -g pane-border-status top
set -g pane-border-format " [ ###P #T ] "
```

```tmux
CTRL+b + :
select-pane -T "title"
```

## Sharing a Window Across Sessions

To share a window between two sessions: `tmux link-window -s <src-window> -t <dst-window>`
