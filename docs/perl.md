# Perl

## Print Unicode Codepoint

Here, we have a list of character counts in a corpus file `train.characters`.
We would like to see the character's unicode codepoint followed by its count.
WARNING: You need to use `< file` or else this script fails.

```
ᑮ 346
ᑯ 850
ᑰ 110
ᑲ 888
ᑳ 638
ᑵ 36
```

```sh
perl -CI -Mutf8 -ple 's/([^ ]+)/sprintf("U+%04X", ord($1))/e' < train.char_count
```

- `-CIO` enables UTF-8 on input (`-CI`) and output (`-CO`)
- `-Mutf8` tells Perl the source is UTF-8
- `split` // splits the line into characters
- `ord($c)` gets the code point, `printf` formats it as `U+XXXX`

```
U+146F 850
U+1470 110
U+1472 888
U+1473 638
U+1475 36
```

## Remove Repeating Chinese Characters

```sh
perl -ple 'BEGIN{use utf8;use open qw(:std :utf8);} s/(\p{Han})\1{1,}/\1/gm' < translation.zho.word
```
