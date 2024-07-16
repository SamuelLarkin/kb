# Perl

## Remove Repeating Chinese Characters

```sh
perl -ple 'BEGIN{use utf8;use open qw(:std :utf8);} s/(\p{Han})\1{1,}/\1/gm' < translation.zho.word
```
