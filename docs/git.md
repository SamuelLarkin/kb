# Git

## git

[How to remove a remote branch ref from local (gh-pages)](https://stackoverflow.com/a/64618529)

```sh
git update-ref -d refs/remotes/origin/gh-pages
```

How to delete a remote branch.

```sh
git push origin --delete dev/semantic_diff
```


## GitHub

Delete all failed `CI activity` for a given user.
[Delete GitHub workflow runs using the gh cli](https://blog.oddbit.com/post/2022-09-22-delete-workflow-runs/)

```sh
gh run list --status failure --user samuellarkin --json databaseId -q '.[].databaseId' \
| parallel --jobs  1 "gh api repos/$(gh repo view --json nameWithOwner -q .nameWithOwner)/actions/runs/{} -X DELETE"
```


### Push an Approved PR's branch

Once a PR is approved, you can use the following command to merge your `dev/work` branch to `main` given that your branch is at the tip of `main`.
This effectively does a fast forward push from the CLI.

```sh
git push origin origin/dev/work:main
```
