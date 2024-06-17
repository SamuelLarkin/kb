# `tmux`

## Exporting a Variable

[tmux is exporting an environment variable that is no longer being exported in .bashrc](https://superuser.com/questions/1216492/tmux-is-exporting-an-environment-variable-that-is-no-longer-being-exported-in-b)

```sh
tmux set-environment -gru FZF_TMUX_OPTS
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


## Find a Panes Running a Command
find-window [-iCNrTZ] [-t target-pane] match-string
  (alias: findw)
  Search for a fnmatch(3) pattern or, with -r, regular expression match-string
  in window names, titles, and visible content (but not history).  The flags
  control matching behavior: -C matches only visible window contents, -N matches
  only the window name and -T matches only the window title.  -i makes the search
  ignore case. The default is -CNT.  -Z zooms the pane.

  This command works only if at least one client is attached.

```tmux
CTRL+b + f
<SEARCH STRING>
```
