# JSON/JSONL

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

- group entries by `id`
- for each group
  - take the first element and aggregate all of the `b` in a list
  - return that first element that has been augmented with a list of `b`

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

## Convert to Array

Given

```json
[{ "seg": ["A", "B"] }, { "seg": "C" }]
```

The second object is NOT an array but you need it to be an array to process all elements the same way, you can make sure all segments are arrays by doing:

```sh
jq '.[] | .seg | (if type == "object" then [.] else . end) | .[]'
```

## Counting Elements

Count the number of entries/sentence pairs that have the `.unparsable` key.

```sh
pv Huge.jsonl \
| jq --null-input '[ inputs | select(.unparsable)] | reduce .[] as $item (0; . + 1)'
```

## Filename

If you need the file name in your script, you can call the function `input_filename`.

```sh
jq '[input_filename, input_line_number]' translation/nmt/test.en2fr.scores.json
```

```
[
  "translation/nmt/test.en2fr.scores.json",
  53
]
```

## Filter-out SubObjects

Given

```xml
<?xml version='1.0' encoding='utf-8'?>
<dataset id="wmttest2024">
  <collection id="general">
    <doc origlang="en" id="test-en-news_beverly_press.3585" domain="news">
      <src lang="en">
        <p>
          <seg id="1">Siso's depictions of land, water center new gallery exhibition</seg>
        </p>
      </src>
      <ref lang="es" translator="refA">
        <p>
          <seg id="1">Representaciones de la tierra y el agua de Siso centran una nueva exposición</seg>
        </p>
      </ref>
    </doc>
    <doc origlang="en" id="test-en-news_brisbanetimes.com.au.228963" domain="NOT_news">
      <src lang="en">
        <p>
          <seg id="1">Adapt the old, accommodate the new to solve issue</seg>
        </p>
      </src>
      <ref lang="es" translator="refA">
        <p>
          <seg id="1">Adapta lo viejo, incorpora lo nuevo para resolver el problema</seg>
        </p>
      </ref>
    </doc>
  </collection>
</dataset>
```

Remove documents that are NOT of `news` domain keeping the document's structure.

```sh
~/.local/bin/yq 'del(.dataset.collection.doc[] | select(.["+@domain"] != "news"))' wmttest2024.en-es.xml
```

```xml
<?xml version='1.0' encoding='utf-8'?>
<dataset id="wmttest2024">
  <collection id="general">
    <doc origlang="en" id="test-en-news_beverly_press.3585" domain="news">
      <src lang="en">
        <p>
          <seg id="1">Siso's depictions of land, water center new gallery exhibition</seg>
        </p>
      </src>
      <ref lang="es" translator="refA">
        <p>
          <seg id="1">Representaciones de la tierra y el agua de Siso centran una nueva exposición</seg>
        </p>
      </ref>
    </doc>
  </collection>
</dataset>
```

## Flat Files to Structured json

When you have multiple flat files that you want to combine into a structured json.

_lingua_eng_spa/Tilde-worldbank-1-eng-spa.spa.gz_

```
SPA     0.9998978843092705
SPA     0.9991979235059277
```

_lingua_all_languages/Tilde-worldbank-1-eng-spa.spa.gz_

```
SPA     0.9999975457963204
SPA     0.9847735076254288
```

_Tilde-worldbank-1-eng-spa.spa.gz_

```
"Igualmente, hacemos notar la importancia de abordar el problema del hambre y la malnutrición”.
"La vida es muy difícil.
```

_Tilde-worldbank-1-eng-spa.eng.gz_

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

## Value in a Set

Select entries that have a field value within a set.

```sh
jq 'select(.BlockID == (1742974, 1742975))' data/hoc-log-20241218-blocks.json
```

## XML to json

Using [yq](https://github.com/mikefarah/yq/), we can convert a xml document into a json file.

```sh
yq -p xml -o json < input.xml > output.json
```

## Zip Multiple files

[Merge arrays](https://github.com/jqlang/jq/issues/680)
The key here is the `transpose`.

```sh
zcat translation.fr.json.gz \
| jq \
    --slurp \
    --argfile src <(jq --raw-input '{"src":.}' source.en) \
    --argfile ref <(jq --raw-input '{"ref":.}' reference.fr) \
    '[., $src, $ref] | transpose | map(add) | .[]'
```

## script.jq

To make a `jq` script executable:

```sh
#!/usr/bin/env -S jq -Mf --slurp
```
