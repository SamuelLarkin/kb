# Tips-and-Tricks

## Activate Custom Scripts

When running experiments, we often have a `venv` and a couple of custom scripts and aliases.
This templates is used to enable all those.

```sh
# :vim:filetype=bash:

tool_dir=$(readlink -m $(dirname "${BASH_SOURCE[0]}"))
echo $tool_dir

test -f $tool_dir/../venv/bin/activate && source $tool_dir/../venv/bin/activate ""
[ -f $tool_dir/aliases ] && source $tool_dir/aliases ""
export PATH=$tool_dir${PATH:+:$PATH}

unset tool_dir
```

## Activate Template for Common Software

This is an example of an `activate` script when you compile a tool by hand and you don't install it a common place.

```sh
############ SENTENCEPIECE ############
# Set this variable to override where Python is installed

[ -n "$SENTENCEPIECE_HOME" ] && echo "NOT sourcing SENTENCEPIECE again" >&2 && return

# Home
export SENTENCEPIECE_HOME=$(readlink -m $(dirname "${BASH_SOURCE[0]}"))
export SENTENCEPIECE_HOME=${SENTENCEPIECE_HOME_OVERRIDE:-$SENTENCEPIECE_HOME}

# Binaries
export PATH=$SENTENCEPIECE_HOME/bin:$PATH

# Libraries
export LIBRARY_PATH=$SENTENCEPIECE_HOME/lib64${LIBRARY_PATH:+:$LIBRARY_PATH}
export LD_LIBRARY_PATH=$SENTENCEPIECE_HOME/lib64${LD_LIBRARY_PATH:+:$LD_LIBRARY_PATH}
export LD_RUN_PATH=$SENTENCEPIECE_HOME/lib64${LD_RUN_PATH:+:$LD_RUN_PATH}

# Includes
export CPLUS_INCLUDE_PATH=$SENTENCEPIECE_HOME/include${CPLUS_INCLUDE_PATH:+:$CPLUS_INCLUDE_PATH}

# Package Configuration for ./configure to work when building other packages.
export PKG_CONFIG_PATH=$SENTENCEPIECE_HOME/lib/pkgconfig${PKG_CONFIG_PATH:+:$PKG_CONFIG_PATH}
```

## BASH Debugging

- [Bash debugging - Youtube](https://www.youtube.com/watch?v=9pbpevjuwmI)
- `PS4` `export PS4='${BASH_SOURCE}:${LINENO}: ${FUNCNAME[0]}() - [${SHLVL},${BASH_SUBSHELL},$?] '`
- `bash -x`
- `bashdb`
- `shellcheck`

## Broken Symlinks

```sh
find . -type l ! -exec test -e {} \; -print
```

Here's an example to fix symlinks that need to have a level added to them.

```sh
function fix_link {
  local -r link=${1:?LINK?}
  ln -fns ../$(readlink $link) $link
}
```

## Disk Usage

### Tools

- [diskus: A minimal, fast alternative to 'du -sh'](https://github.com/sharkdp/diskus)
- [dua: View disk space usage and delete unwanted data, fast.](https://github.com/Byron/dua-cli)
- [duc: Duc is a collection of tools for inspecting and visualizing disk usage](https://duc.zevv.nl/)
- [dust: A more intuitive version of du in rust](https://github.com/bootandy/dust)
- [dutree: a tool to analyze file system usage written in Rust](https://github.com/nachoparker/dutree)
- [gdu: Fast disk usage analyzer with console interface written in Go](https://github.com/dundee/gdu)
- [godu: Simple golang utility helping to discover large files/folders.](https://github.com/viktomas/godu)
- [pdu: Highly parallelized, blazing fast directory tree analyzer](https://github.com/KSXGitHub/parallel-disk-usage)
- [tin-summer: Find build artifacts that are taking up disk space](https://github.com/vmchale/tin-summer)

### View disk usage by filetype

```sh
dust -t
```

```
 3.0K   ┌── (others)      │                                             █ │   0%
 2.0K   ├── .BLEU         │                                             █ │   0%
 2.0K   ├── .CHRF         │                                             █ │   0%
 2.0K   ├── .TER          │                                             █ │   0%
  64K   ├── .en           │                                             █ │   0%
  64K   ├── .sh           │                                             █ │   0%
  64K   ├── .train        │                                             █ │   0%
  64K   ├── .xz           │                                             █ │   0%
 128K   ├── .sharditer    │                                             █ │   0%
 130K   ├── .0            │                                             █ │   0%
 192K   ├── .log          │                                             █ │   0%
 832K   ├── .00000        │                                             █ │   0%
 1.3M   ├── (no extension)│                                             █ │   0%
 2.1M   ├── .fr           │                                             █ │   0%
 2.1M   ├── .gz           │                                             █ │   0%
 2.6M   ├── .out          │                                             █ │   0%
 3.3M   ├── .json         │                                             █ │   0%
 4.9M   ├── .word         │                                             █ │   0%
 799M   ├── .00001        │                                    ██████████ │  20%
 3.1G   ├── .pkl          │         █████████████████████████████████████ │  80%
 3.9G ┌─┴ (total)         │██████████████████████████████████████████████ │ 100%
```

## Filtering

### Filtering tsv Files on Column

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

### Filter-out Testset From Train

```sh
grep --text --line-regexp --invert-match --fixed-strings --file=$testset_filename
```

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

## Grep for Emojis

[POSIX and Unicode character categories](https://github.com/Genivia/ugrep?tab=readme-ov-file#posix-and-unicode-character-categories)

```sh
ugrep '\p{So}' input_file
```

## Keep Sentence Pairs for Sockeye's Length Limit

Here's an example that keep sentence pairs that have less than 5 tokens.

```sh
paste <(zcat OPUS-multiun-v1-eng-spa.spa.gz) <(zcat OPUS-multiun-v1-eng-spa.eng.gz) \
| awk \
    -F'\t' \
    'BEGIN {OFS = FS}  (split($1, a, " +")<5 && split($2, b, " +")< 5) {print $1, $2}'
```

## Login Name

Find the full name of a user from its username.

```sh
lslogins | fzf
```

## Refresh Bash's Cache

[How do I clear Bash's cache of paths to executables?](https://unix.stackexchange.com/a/5610)
`bash` does cache the full path to a command.
You can verify that the command you are trying to execute is hashed with the type command:

```sh
type svnsync
svnsync is hashed (/usr/local/bin/svnsync)
```

To clear the entire cache:

`hash -r`

Or just one entry:

`hash -d svnsync`

For additional information, consult help hash and man bash.

## Seeded `shuf`

```sh
function get_seeded_random {
  local -r seed="$1"
  openssl enc -aes-256-ctr -pass pass:"$seed" -nosalt </dev/zero 2>/dev/null
}

shuf -i1-100 --random-source=<(get_seeded_random 42)
```

## Sort

Sort according to a set of columns:

```sh
zcat FILE.gz | awk -F'\t' '!_[$4,$5]++'
```

## Time

To get an time output that is easier to parse.

```sh
command time --portability python my_script.py
```

```
real 3588.63
user 3333.36
sys 519.36
```

## Watch Files/Directories then Take Action

If you want to take action when a directory gets created, you can use `watchexec`.
Note, there used to be a `--emit-events-to environment` but it is phasing-out.

```sh
watchexec \
  --emit-events-to file \
  --watch compare-mt.output/ \
  --fs-events=create \
  --filter='checkpoint-?00' \
  --filter='checkpoint-*000' \
  "parallel --dry-run --plus 'sbatch translate.slurm {##create:} en2fr' :::: \$WATCHEXEC_EVENTS_FILE"
```

## Weather

[wttr.in - GitHub](https://github.com/chubin/wttr.in): The right way to check the weather
Get the weather:

- `curl wttr.in/CityName`
- `curl v2d.wttr.in/CityName`

## Where is the process running

If there is a running process like `vim` that you would like to properly stop, you need to find in which `tmux` window it is running.
To help figure this out, given the PID, you can ask `lsof` for its `CWD`.

```sh
lsof -a -d cwd -p PID
```

```
COMMAND  PID    USER   FD   TYPE DEVICE SIZE/OFF      NODE NAME
vim     7688 larkins  cwd    DIR   0,47     4096 154447160 /gpfs/projects/DT/mtp/models/HoC-Senate/corpora/spm/v2
```

## conda

Build environment spec from explicit specs in history.

```sh
conda env export --from-history --prefix "$prefix" > "$prefix.from-history.yaml"
```

## setfacl

```sh
   setfacl -m g:blz_acl_mtp:rwx git
   getfacl --access git | setfacl -d -M- git
```

Change the default for the common group.
Apply only to directories.

```sh
find corpora/ models/ pkgs/ venv/ -type d \
| xargs setfacl -d -m g:ai4dcluster-dt-mtp-project:rwx
```
