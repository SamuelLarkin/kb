# `tmux`
## Exporting a Variable
[tmux is exporting an environment variable that is no longer being exported in .bashrc](https://superuser.com/questions/1216492/tmux-is-exporting-an-environment-variable-that-is-no-longer-being-exported-in-b)
```
tmux set-environment -r FZF_TMUX_OPTS
```

## Rename a pane
```
set -g pane-border-status top
set -g pane-border-format " [ ###P #T ] "
```
```
CTRL+b + :
select-pane -T "title"
```
