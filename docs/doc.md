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



## BASH debugging
* [Bash debugging - Youtube](https://www.youtube.com/watch?v=9pbpevjuwmI)
* `PS4` `export PS4='${BASH_SOURCE}:${LINENO}: ${FUNCNAME[0]}() - [${SHLVL},${BASH_SUBSHELL},$?] '`
* `bash -x`
* `bashdb`
* `shellcheck`



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
