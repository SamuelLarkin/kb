# rsync

 [rsync copy over only certain types of files using include option](https://stackoverflow.com/a/11111793)
 NOTE you MUST add --include='*/' to let rsync at least visit all directories.
```sh
rsync \
  -Parzu \
  -m \
  --include='*/' \
  --include='*/run_mlm.config' \
  --include='*/trainer.py' \
  --exclude='*' \
  xlm-roberta-large.2500.min_prob.emb \
  xlm-roberta-large.2500.not_sorted.emb \
  xlm-roberta-large.5000.min_prob.emb \
  xlm-roberta-large.5000.not_sorted.emb \
  ../mt/ \
  -n
```

 [How to use Rsync to copy only specific subdirectories (same names in several directories)](https://stackoverflow.com/questions/15687755/how-to-use-rsync-to-copy-only-specific-subdirectories-same-names-in-several-dir)
```sh
rsync \
  -F \
  -Parzu \
  gpsc5:/space/project/portage/models/WMT2023 \
  /gpfs/projects/DT/mtp/models/ \
  --include='*/' \
  --include='*/wmttest2023.splited' \
  --include='wmttest2023.splited/***' \
  --exclude='*' \
  -n
```
