# Git

## git

[How to remove a remote branch ref from local (gh-pages)](https://stackoverflow.com/a/64618529)

```sh
git update-ref -d refs/remotes/origin/gh-pages
```


## GitHub

Delete all failed `CI activity` for a given user.
[Delete GitHub workflow runs using the gh cli](https://blog.oddbit.com/post/2022-09-22-delete-workflow-runs/)

```sh
gh run list --status failure --user samuellarkin --json databaseId -q '.[].databaseId' \
| parallel --jobs  1 "gh api repos/$(gh repo view --json nameWithOwner -q .nameWithOwner)/actions/runs/{} -X DELETE"
```
