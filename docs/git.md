# Git

## Remove Remote Branch from Local

[How to remove a remote branch ref from local (gh-pages)](https://stackoverflow.com/a/64618529)

```sh
git update-ref -d refs/remotes/origin/gh-pages
```

## How to Delete a Remote Branch

```sh
git push origin --delete dev/semantic_diff
```

## Realign a Branch with `origin`

When you want to make your branch `BRANCH` the same as `origin/BRANCH` no matter what.

```sh
git swith BRANCH
git reset --hard origin/BRANC
```

## Temporarily Disabled `delta` to Get a Patch

To get a patch out of `git diff`, by the command to `less` which changes the pager from `delta` to `less`.

```sh
git diff main -- .pre-commit-config.yaml | less
```

# GitHub

## Failed CI activity

Delete all failed `CI activity` for a given user.
[Delete GitHub workflow runs using the gh cli](https://blog.oddbit.com/post/2022-09-22-delete-workflow-runs/)

```sh
gh run list --status failure --user samuellarkin --json databaseId -q '.[].databaseId' \
| parallel --jobs  1 "gh api repos/$(gh repo view --json nameWithOwner -q .nameWithOwner)/actions/runs/{} -X DELETE"
```

## Push an Approved PR's branch

Once a PR is approved, you can use the following command to merge your `dev/work` branch to `main` given that your branch is at the tip of `main`.
This effectively does a fast forward push from the CLI.

```sh
git push origin origin/dev/work:main
```

## Find Which PR a Commit Belongs to

When you want to know a commit what added during which Pull Request:

```sh
git clone --mirror git@github.com:EveryVoiceTTS/EveryVoice.git EveryVoice-mirror
cd EveryVoice-mirror/
sed -i 's|refs/pull/|refs/heads/pull/|' packed-refs
git log --all --graph --decorate --oneline   # or your favourite compact log
