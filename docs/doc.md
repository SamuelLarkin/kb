# Configuration Files

* [Difference Between `.bashrc`, `.bash-profile`, and `.profile`](https://www.baeldung.com/linux/bashrc-vs-bash-profile-vs-profile)
* [What are the functional differences between .profile .bash_profile and .bashrc](https://serverfault.com/questions/261802/what-are-the-functional-differences-between-profile-bash-profile-and-bashrc)


`.bash_profile` and `.bashrc` are specific to `bash`, whereas `.profile` is read by many shells in the absence of their own shell-specific config files.
(`.profile` was used by the original Bourne shell.) `.bash_profile` or `.profile` is read by login shells, along with `.bashrc`; subshells read only `.bashrc`.
(Between job control and modern windowing systems, `.bashrc` by itself doesn't get used much. If you use `screen` or `tmux`, screens/windows usually run subshells instead of login shells.)

The idea behind this was that one-time setup was done by `.profile` (or shell-specific version thereof), and per-shell stuff by `.bashrc`.
For example, you generally only want to load environment variables once per session instead of getting them whacked any time you launch a subshell within a session, whereas you always want your aliases (which aren't propagated automatically like environment variables are).

Other notable shell config files:

`/etc/bash_profile` (fallback `/etc/profile`) is read before the user's .profile for system-wide configuration, and likewise `/etc/bashrc` in subshells (no fallback for this one).
Many systems including Ubuntu also use an `/etc/profile.d` directory containing shell scriptlets, which are `.` (`source`)-ed from `/etc/profile`; the fragments here are per-shell, with `*.sh` applying to all Bourne/POSIX compatible shells and other extensions applying to that particular shell.



# Bash Colors
[Script to display all terminal colors](https://askubuntu.com/a/1044802)
```sh
msgcat --color=test
```



# Putty Colors
We need to use `putty-256color` instead of `xterm-256color` or else the `Home` key is not working.

[Putty shows prompt with no color, but Linux SSH can](https://superuser.com/a/1502895)
In Putty, change Settings -> Connection > Data > Terminal-type string to: `putty-256color`.
["Emulate" 256 colors in PuTTY terminal](https://superuser.com/a/436928)
1. Configure Putty

In Settings > Windows > Colours there is a check box for "Allow terminal to use xterm 256-colour mode".
2. Let the app know

You'll probably have to change Settings -> Connection > Data > Terminal-type string to: `xterm-256color`

if your server has a terminfo entry for `putty-256color`, typically in `/usr/share/terminfo/p/putty-256color`, you can set `Putty`'s Terminal-Type to `putty-256color` instead.

The main thing here is to make the server use an available `Terminfo` entry that most closely matches the way `Putty` is configured.

## 24bit
* [Getting 24-bit color working in terminals](https://pisquare.osisoft.com/s/Blog-Detail/a8r1I000000GvXBQA0/console-things-getting-24bit-color-working-in-terminals)




# LSCOLORS Schemes
```sh
for theme in $(vivid themes); do
  echo "Theme: $theme";
  LS_COLORS=$(vivid generate $theme);
  ls;
  echo;
done
vivid generate one-light
LSCOLORS=$(vivid generate one-light)
```



# Tips-and-Tricks
## Sort
Sort according to a set of columns:
```sh
zcat FILE.gz | awk -F'\t' '!_[$4,$5]++'
```

## Filtering
Filtering tsv files based on a subset of columns.
Provided by Marc.
```sh
# Filter-in
awk \
  -F'\t' \
  'NR==FNR{a[$4,$5];next} ($4,$5) in a' \
  uniq.DEVTEST_2022_${BIFILTER}.tsv \
  uniq.TRAIN_2021-2016_${BIFILTER}.tsv \
> TRAIN_indev.tsv
```
```sh
# Filter-out
awk \
  -F'\t' \
  'NR==FNR{a[$4,$5];next} !(($4,$5) in a)' \
  uniq.DEVTEST_2022_${BIFILTER}.tsv \
  uniq.TRAIN_2021-2016_${BIFILTER}.tsv \
> TRAIN_notindev.tsv
```

## JSONL
### Compare
Compare two jsonl files that are not in the same order.
This implies that we need to sort the files on some `key`.
Do a trick a la `Schwartian transform` where we prepend the sort `key`, sort on that `key` and the remove the `key`.
```sh
jqdiff \
  <(zcat train.jsonl.gz \
    | jq --compact-output --raw-output '[.id, .|@text] | @tsv' \
    | sort -k1,1 \
    | cut -f2,2) \
  <(zcat source/train.jsonl.gz \
    | jq --compact-output --raw-output '[.id, .|@text] | @tsv' \
    | sort -k1,1 \
    | cut -f 2,2)
```

Otherwise, use the alias `jqdiff` which essentially does
```sh
vimdiff <(jq --sort-keys . file1.json) <(jq --sort-keys . file2.json)
```

### Parallel Processing
Note that we use `--keep-order`, `--spreadstdin` & `--recend='\n'`.
```sh
function process {
   cat
}
export -f process

zcat input.gz \
    | time \parallel \
       --keep-order \
       --spreadstdin \
       --recend='\n' \
       --env=process \
       'process' \
| gzip \
> output.gz
```


### Counting Elements
Count the number of entries/sentence pairs that have the `.unparsable` key.
```sh
pv Huge.jsonl \
| jq --null-input '[ inputs | select(.unparsable)] | reduce .[] as $item (0; . + 1)'
```



## git
[How to remove a remote branch ref from local (gh-pages)](https://stackoverflow.com/a/64618529)
```sh
git update-ref -d refs/remotes/origin/gh-pages
```


## Broken Symlinks
```sh
find . -type l ! -exec test -e {} \; -print
```


## SLURM
### Script Example
The following example uses multiple nodes.
```sbatch
#!/bin/bash

#SBATCH --job-name=train
#SBATCH --partition=gpu_a100
#SBATCH --account=nrc_ict__gpu_a100

#SBATCH --time=2880

##IMPORTANT: `#SBATCH --ntasks-per-node=2`  MUST match  `#SBATCH --gres=gpu:2`
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=2
#SBATCH --gres=gpu:2

#SBATCH --output=./A100x2.o%j
#SBATCH --error=./A100x2.e%j
#SBATCH --open-mode=append
#SBATCH --mail-user==tes001
#SBATCH --mail-type=NONE

#SBATCH --signal=B:15@30


ulimit -v unlimited

cd /home/tes001/DT/tes001/
source ./SETUP_PT2.source
cd /home/tes001/DT/tes001/LJSpeech-1.1/PT2

srun everyvoice train text-to-spec --devices 2 --nodes 1 config/everyvoice-text-to-spec.yaml
```

### Stats of a job
Get some stats about a job that ran on `Slurm`.
```sh
sacct --long --jobs=jobid
sacct -l -j jobid
```

### Node's Specs
Get node's specs.
```sh
sinfo --Node --responding --long
sinfo -N -r -l
```
| NODELIST | NODES | PARTITION   | STATE      | CPUS   | S:C:T  | MEMORY | TMP_DISK | WEIGHT | AVAIL_FE | REASON |
|----------|-------|-------------|------------|--------|--------|--------|----------|--------|----------|--------|
| cn101    | 1     | TrixieMain* | drained    | 64     | 2:16:2 | 192777 | 0        | 1      | (null)   | update |
| cn102    | 1     | TrixieMain* | mixed      | 64     | 2:16:2 | 192777 | 0        | 1      | (null)   | none   |
| cn110    | 1     | TrixieMain* | mixed      | 64     | 2:16:2 | 192777 | 0        | 1      | (null)   | none   |
| cn118    | 1     | TrixieMain* | idle       | 64     | 2:16:2 | 192777 | 0        | 1      | (null)   | none   |
| cn119    | 1     | TrixieMain* | idle       | 64     | 2:16:2 | 192777 | 0        | 1      | (null)   | none   |
| cn125    | 1     | TrixieMain* | idle       | 64     | 2:16:2 | 192777 | 0        | 1      | (null)   | none   |


### Allocated Hostnames of a Multinode Job
Get a list of hostnames allocated to a multi node job.
```sh
scontrol show hostnames $SLURM_NODELIST
```
```
cn101
cn102
cn103
cn104
```

### SSH to a Worker Node

This is an example command to connect to a worker node on GPSC5
```sh
srun --jobid=JOBID --pty bash -l
```

Example of connecting to a GPU running job on GPSC5.
```sh
srun \
  --jobid 4139955 \
  --gres=gpu:0 \
  --nodes 1 \
  --ntasks 1 \
  --mem-per-cpu=0 \
  --pty \
  --nodelist ib12gpu-001 \
  --oversubscribe \
  /bin/bash -l
```


## BASH debugging
* [Bash debugging - Youtube](https://www.youtube.com/watch?v=9pbpevjuwmI)
* `PS4` `export PS4='${BASH_SOURCE}:${LINENO}: ${FUNCNAME[0]}() - [${SHLVL},${BASH_SUBSHELL},$?] '`
* `bash -x`
* `bashdb`
* `shellcheck`


## `tmux`
### Exporting a Variable
[tmux is exporting an environment variable that is no longer being exported in .bashrc](https://superuser.com/questions/1216492/tmux-is-exporting-an-environment-variable-that-is-no-longer-being-exported-in-b)
```
tmux set-environment -r FZF_TMUX_OPTS
```

### Rename a pane
```
set -g pane-border-status top
set -g pane-border-format " [ ###P #T ] "
```
```
CTRL+b + :
select-pane -T "title"
```

### `tmux-fzf`
* List of bindings
    * <kbd>prefix</kbd> <kbd>F</kbd>  To launch tmux-fzf, press `prefix` + `F` (Shift+F).


## `fzf-git.sh`
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
  >     * (`stty -ixon` will disable it)
  >
  > To workaround the problems, you can use
  > <kbd>CTRL-G</kbd> <kbd>*{key}*</kbd> instead of
  > <kbd>CTRL-G</kbd> <kbd>CTRL-*{KEY}*</kbd>.
* Inside fzf
    * <kbd>TAB</kbd> or <kbd>SHIFT-TAB</kbd> to select multiple objects
    * <kbd>CTRL-/</kbd> to change preview window layout
    * <kbd>CTRL-O</kbd> to open the object in the web browser (in GitHub URL scheme)


## `lvim`
Find commands `:WhichKey`.
This opens a popup and if you type the shortcut key you get submenus.

`<leader>sT` where `<leader>` is `space` opens a popup to do fuzzy finding across files.

`<leader>f` find a file.



## GNU parallel a la Spark
```sh
function desubtokenize {}
export -f desubtokenize

function tokenize_corpus {}
export -f tokenize_corpus

zcat --force train.gz \
| parallel \
  --spreadstdin \
  --recend '\n' \
  --env desubtokenize \
  --env tokenize_corpus \
  "desubtokenize | tokenize_corpus $lang" \
> train.tok.gz
```


## GPSCC
Start a sleeper job
```sh
psub -N sleeper -Q nrc_ict 'sleep 3600'
```


## Weather
[wttr.in - GitHub](https://github.com/chubin/wttr.in): The right way to check the weather
Get the weather:
* `curl wttr.in/CityName`
* `curl v2d.wttr.in/CityName`


## Login Name
Find the full name of a user from its username.
```sh
lslogins | fzf
```
