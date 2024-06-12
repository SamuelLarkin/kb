# JSON/JSONL

## Compare

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

## Parallel Processing

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


## Counting Elements

Count the number of entries/sentence pairs that have the `.unparsable` key.

```sh
pv Huge.jsonl \
| jq --null-input '[ inputs | select(.unparsable)] | reduce .[] as $item (0; . + 1)'
```


## Group by X and Merge

Context: after generating `*.scores.json` using `sacrebleu  --width=14 reference --metrics bleu chrf ter  < translation > scores.json`.
Can we extract BLEU scores from all our experiments and tabulate the result using `mlr`?

```sh
find -type f -name \*scores.json \
| xargs dirname \
| parallel 'jq "{\"expt_name\": \"{//}\",  \"{/}\": (.[] | select(.name == \"BLEU\") | .score)}" {}/*scores.json' \
| jq --slurp --sort-keys 'group_by(.expt_name) | [.[] | add]' \
| mlr --json --opprint --barred cat \
| less
```



## Aggregate a Field

Given a list of objects where some of them have the same `id` but with a field with different values, aggregate that field for each object.
This happens when you extracted data from `mysql`.
`mysql` doesn't allow subqueries to return multiple rows with multiple columns thus you have to do the same work using `JOIN`.
[Merge Arrays of JSON](https://stackoverflow.com/a/42012281)

```sh
echo -e '{"id":1, "b":[{"c":1}]}{"id":1, "b":[{"c":2}]}'
```
```
{
  "id": 1,
  "b": [
    {
      "c": 1
    }
  ]
}
{
  "id": 1,
  "b": [
    {
      "c": 2
    }
  ]
}
```

* group entries by `id`
* for each group
  * take the first element and aggregate all of the `b` in a list
  * return that first element that has been augmented with a list of `b`



```sh
echo -e '{"id":1, "b":[{"c":1}]}{"id":1, "b":[{"c":2}]}' \
| jq --slurp 'group_by(.id) | .[] | (.[0].b=([.[].b]|flatten)) | .[0]'
```
```
{
  "id": 1,
  "b": [
    {
      "c": 1
    },
    {
      "c": 2
    }
  ]
}
```

## Zip Multiple files

[Merge arrays](https://github.com/jqlang/jq/issues/680)
The key here is the `transpose`.

```sh
zcat translation.fr.json.gz \
| jq \
    --slurp \
    --argfile src <(jq -R '{"src":.}' source.en) \
    --argfile ref <(jq -R '{"ref":.}' reference.fr) \
    '[., $src, $ref] | transpose | map(add) | .[]'
```


## Flat Files to Structured json

When you have multiple flat files that you want to combine into a structured json.

*lingua_eng_spa/Tilde-worldbank-1-eng-spa.spa.gz*
```
SPA     0.9998978843092705
SPA     0.9991979235059277
```

*lingua_all_languages/Tilde-worldbank-1-eng-spa.spa.gz*
```
SPA     0.9999975457963204
SPA     0.9847735076254288
```

*Tilde-worldbank-1-eng-spa.spa.gz*
```
"Igualmente, hacemos notar la importancia de abordar el problema del hambre y la malnutrición”.
"La vida es muy difícil.
```

*Tilde-worldbank-1-eng-spa.eng.gz*
```
" We also note the importance of addressing hunger and malnutrition.”
"[Life] is extremely difficult.
```

```sh
paste \
  <(zcat lingua_eng_spa/Tilde-worldbank-1-eng-spa.spa.gz) \
  <(zcat lingua_all_languages/Tilde-worldbank-1-eng-spa.spa.gz) \
  <(zcat Tilde-worldbank-1-eng-spa.spa.gz) \
  <(zcat Tilde-worldbank-1-eng-spa.eng.gz) \
| mlr --tsv --ojson --implicit-csv-header \
  label eng_spa.lid,eng_spa.confidence,all.lid,all.confidence,spa,eng \
| jq '.[]'
```

```json
{
  "eng_spa": {
    "lid": "SPA",
    "confidence": 0.9998978843092705
  },
  "all": {
    "lid": "SPA",
    "confidence": 0.9999975457963204
  },
  "spa": "\"Igualmente, hacemos notar la importancia de abordar el problema del hambre y la malnutrición”.",
  "eng": "\" We also note the importance of addressing hunger and malnutrition.”"
}
{
  "eng_spa": {
    "lid": "SPA",
    "confidence": 0.9991979235059277
  },
  "all": {
    "lid": "SPA",
    "confidence": 0.9847735076254288
  },
  "spa": "\"La vida es muy difícil.",
  "eng": "\"[Life] is extremely difficult."
}
```
