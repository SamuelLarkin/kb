# fzf

## `tmux-fzf`

[tmux-fzf](https://github.com/sainnhe/tmux-fzf): Use fzf to manage your tmux work environment!

* List of bindings
  * <kbd>prefix</kbd> <kbd>F</kbd>  To launch tmux-fzf, press `prefix` + `F` (Shift+F).

## `fzf-git.sh`

[fzf-git.sh](https://github.com/junegunn/fzf-git.sh): bash and zsh key bindings for Git objects, powered by fzf.

* List of bindings
  * <kbd>CTRL-G</kbd> <kbd>CTRL-F</kbd> for **F**iles
  * <kbd>CTRL-G</kbd> <kbd>CTRL-B</kbd> for **B**ranches
  * <kbd>CTRL-G</kbd> <kbd>CTRL-T</kbd> for **T**ags
  * <kbd>CTRL-G</kbd> <kbd>CTRL-R</kbd> for **R**emotes
  * <kbd>CTRL-G</kbd> <kbd>CTRL-H</kbd> for commit **H**ashes
  * <kbd>CTRL-G</kbd> <kbd>CTRL-S</kbd> for **S**tashes
  * <kbd>CTRL-G</kbd> <kbd>CTRL-E</kbd> for **E**ach ref (`git for-each-ref`)
  > :warning: You may have issues with these bindings in the following cases:
  >
  > * <kbd>CTRL-G</kbd> <kbd>CTRL-B</kbd> will not work if
  >   <kbd>CTRL-B</kbd> is used as the tmux prefix
  > * <kbd>CTRL-G</kbd> <kbd>CTRL-S</kbd> will not work if flow control is enabled,
  >   <kbd>CTRL-S</kbd> will freeze the terminal instead
  >   * (`stty -ixon` will disable it)
  >
  > To workaround the problems, you can use
  > <kbd>CTRL-G</kbd> <kbd>*{key}*</kbd> instead of
  > <kbd>CTRL-G</kbd> <kbd>CTRL-*{KEY}*</kbd>.
* Inside fzf
  * <kbd>TAB</kbd> or <kbd>SHIFT-TAB</kbd> to select multiple objects
  * <kbd>CTRL-/</kbd> to change preview window layout
  * <kbd>CTRL-O</kbd> to open the object in the web browser (in GitHub URL scheme)
