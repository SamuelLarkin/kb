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



## Python

How to profile python's import statements.
Used in finding expensive imports that slow down `--help`.

```sh
python -X importtime myscript.py
```

or:

```sh
PYTHONPROFILEIMPORTTIME=1 myscript.python
```


## miller

### Tabulate BLEU Scores
* Reading a csv dataframe
* Write a nice table using bars
* Print numbers with 2 decimal
* cut: keep column based on a list of regular expressions
* merge-fields: add a `mean` column calculating the mean of columns
* reorder: move the datetime column at the end of the table
* label: renamed the columns
* sort: on `test` to rank the rows
* put: add the row's rank
* sort: reorder `on expt_name`

```sh
  score-tool tabulate --no-title \
  | mlr --icsv --opprint --barred --ofmt %.2f \
    cut -rf expt_name,$suffix,date \
    then merge-fields  -a mean  -r $suffix  -o $suffix  -k \
    then reorder -e -f datetime \
    then label expt_name,test,validation,mean,datetime \
    then sort -nr test \
    then put 'begin {@rank = 1} $rank = @rank; @rank += 1' \
    then sort -nr expt_name
```

### Find HoC Sittings Elapsed Time

```sh
bzcat sentence_word_count_fr.tsv.bz2 | head -n 3
```
```
id      datetime        wc      sentence
House/House/391/Debates/001/HAN001      2006-04-03 11:05:00.000000      49      The 38th Parliament ...
House/House/391/Debates/001/HAN001      2006-04-03 11:05:00.001000      5       Monday, April 3, 2006
```

* Transform `$datestamp` into the number of second since epoch 0
* Find the minimum and maximum datetime for each Sitting
* Figure out the elpase time
* Convert the start and end time back to a datetime
* Discard unwanted fields by specifying which one we want to keep
* Reorder the fields

```sh
bzcat sentence_word_count_fr.tsv.bz2 \
| head -n 10000 \
| mlr --itsv --opprint --barred \
  cut -x -f sentence then put '$datestamp = strptime($datetime, "%Y-%m-%d %T.%f")' \
  then \
  stats1 -g id -f datestamp -a min,max \
  then \
  put '$elapse = $datestamp_max - $datestamp_min'
  then \
  put '$start = strftime($datestamp_min, "%Y-%m-%d %T"); $end = strftime($datestamp_max, "%Y-%m-%d %T")' \
  then \
  cut -f id,start,end,elapse \
  then \
  reorder -f id,start,end,elapse
```
```
+------------------------------------+---------------------+---------------------+--------------------+
| id                                 | start               | end                 | elapse             |
+------------------------------------+---------------------+---------------------+--------------------+
| House/House/391/Debates/001/HAN001 | 2006-04-03 11:05:00 | 2006-04-03 14:05:00 | 10800.12400007248  |
| House/House/391/Debates/002/HAN002 | 2006-04-04 00:00:00 | 2006-04-04 17:05:00 | 61500.206000089645 |
| House/House/391/Debates/003/HAN003 | 2006-04-05 00:00:00 | 2006-04-05 18:25:00 | 66300.73900008202  |
| House/House/391/Debates/004/HAN004 | 2006-04-06 00:00:00 | 2006-04-06 23:15:01 | 83701.97799992561  |
| House/House/391/Debates/005/HAN005 | 2006-04-07 00:00:00 | 2006-04-07 14:30:00 | 52200.70899987221  |
| House/House/391/Debates/006/HAN006 | 2006-04-10 00:00:00 | 2006-04-10 11:35:00 | 41700.10199999809  |
+------------------------------------+---------------------+---------------------+--------------------+
```
