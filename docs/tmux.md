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
