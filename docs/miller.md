# miller

* [Documentation](https://miller.readthedocs.io)
* [GitHub](https://github.com/johnkerl/miller): Miller is like awk, sed, cut, join, and sort for name-indexed data such as CSV, TSV, and tabular JSON.

## Tabulate BLEU Scores

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

## Find HoC Sittings Elapsed Time

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

## Group per Date

```sh
jq --raw-output --compact-output \
   '{"date": (.timestamp|split(" ")[0]), "fr": (.fr|split(" ") | length), "en": (.en|split(" ")|length)}' \
| mlr --ijson \
  then stats1 -a sum,count -f en,fr -g date \
  then cut -x -f en_count \
  then rename en_sum,#en_word,fr_sum,#fr_word,fr_count,#sentence \
| mlr --opprint --barred summary -a count,null_count,distinct_count,mean,min,median,max,stddev
```

```
{"date":"2022-10-24","fr":18,"en":15}
{"date":"2022-10-24","fr":18,"en":18}
{"date":"2022-10-24","fr":29,"en":28}
```

```
date=2022-10-24,#en_word=55951,#fr_word=59892,#sentence=2692
date=2022-10-25,#en_word=73587,#fr_word=79288,#sentence=3660
date=2022-10-26,#en_word=29726,#fr_word=32800,#sentence=1492
```

| field_name | count | null_count | distinct_count | mean | stddev | min | median | max |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| date | 112 | 0 | 112 |  |  | 2022-10-24 | 2023-03-23 | 2023-09-29 |
| #en_word | 112 | 0 | 112 | 64893.669642857145 | 25599.55527604576 | 2812 | 68386 | 123193 |
| #fr_word | 112 | 0 | 112 | 71302.02678571429 | 27863.87682146542 | 3150 | 76163 | 133257 |
| #sentence | 112 | 0 | 112 | 3282.6160714285716 | 1318.6068832416186 | 128 | 3502 | 6373 |

## Group per Object

```sh
jq --raw-output --compact-output \
   '{"sitting": (.doc | {parliament, session, number})} + {"fr": (.fr|split(" ") | length), "en": (.en|split(" ")|length)}' \
| mlr --ijson \
   then stats1 -a sum,count -f en,fr -g sitting \
   then cut -x -f en_count \
   then rename en_sum,#en_word,fr_sum,#fr_word,fr_count,#sentence \
| mlr --opprint --barred summary -a count,null_count,distinct_count,mean,min,median,max,stddev
```

```
{"sitting":{"parliament":44,"session":1,"number":116},"fr":18,"en":15}
{"sitting":{"parliament":44,"session":1,"number":116},"fr":18,"en":18}
{"sitting":{"parliament":44,"session":1,"number":116},"fr":29,"en":28}
```

```
sitting.parliament=44,sitting.session=1,sitting.number=116,#en_word=55951,#fr_word=59892,#sentence=2692
sitting.parliament=44,sitting.session=1,sitting.number=117,#en_word=73587,#fr_word=79288,#sentence=3660
sitting.parliament=44,sitting.session=1,sitting.number=118,#en_word=29726,#fr_word=32800,#sentence=1492
```

| field_name | count | null_count | distinct_count | mean | stddev | min | median | max |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| sitting.parliament | 111 | 0 | 1 | 44 | 0 | 44 | 44 | 44 |
| sitting.session | 111 | 0 | 1 | 1 | 0 | 1 | 1 | 1 |
| sitting.number | 111 | 0 | 111 | 171.44144144144144 | 32.61697402069964 | 116 | 171 | 227 |
| #en_word | 111 | 0 | 111 | 65478.2972972973 | 25569.833917971166 | 23026 | 68386 | 131905 |
| #fr_word | 111 | 0 | 110 | 71944.38738738738 | 27816.245672487912 | 25521 | 76163 | 142068 |
| #sentence | 111 | 0 | 110 | 3312.189189189189 | 1315.06589889158 | 1147 | 3502 | 6588 |
