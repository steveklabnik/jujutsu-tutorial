# Working with Gerrit

The Gerrit-based workflow prefers small commits that get reviewed individually,
as "changelists" (CLs). `jj` is well suited for this workflow: each `jj` change
can be a single CL which you can update over time.

## Change IDs

Similar to `jj`'s change IDs, Gerrit associates commits with a common "change"
using a `Change-Id` header in the commits. Gerrit repositories typically use a
Git commit hook to add change IDs to commits, you can configure `jj` to do the
same with this configuration:

```toml
[templates]
commit_trailers = '''
if(self.author().email() == "YOUR_EMAIL_HERE" &&
  !trailers.contains_key("Change-Id"),
  format_gerrit_change_id_trailer(self)
)
```


## Push workflow

In a traditional Gerrit workflow, you push commits to the review server using
`git push origin <rev>:refs/for/main`.

Repositories using `depot_tools` will likely teach you about the `git cl upload`
workflow instead: which by default will only push from a named, checked-out
branch, and will squash all commits into a single change upstream
(overrideable with `--no-squash`).


If working with `jj`, it is convenient for a single `jj` change to be a single
Gerrit change. `jj` also likes to keep the repository in "detached HEAD" mode,
so `git cl upload` needs a `git checkout <branchname>` first to work. As such,
`git cl upload` is not the best way to interact with a `jj` repository.

Regardless of whether or not you are using `depot_tools`, `jj` works best with
Gerrit when you build on top of the `refs/for/main` primitive.


The following alias makes uploading to Gerrit with `jj` convenient:

```toml
[aliases]
cl-up = ["util", "exec", "--", "bash", "-c", """
set -euo pipefail
INPUT=${1:-"@-"}
HASH=$(jj log -r "${INPUT}" -T commit_id --no-graph)
HASHINFO=$(git log -n 1 ${HASH} --oneline --color=always)
echo "Pushing from commit ${HASHINFO}"
git push origin "${HASH}":refs/for/main
""", ""]
```

`jj cl-up <rev>` will push `<rev>` (`@-` if not specified) and its ancestors to
`origin`, creating new CLs if necessary, updating existing ones otherwise.

Its output will look something like this:

```console
$ jj
@  wtzvnrmz manishearth@google.com 2025-06-18 18:29:08 f694f9f1
│  (empty) (no description set)
○  zqlxpsus manishearth@google.com 2025-06-18 18:29:08 git_head() 556b213a
│  [temporal] Add Add/Subtract to all Temporal types
│ ○  rnnrzqpt manishearth@google.com 2025-06-18 18:29:08 8d16e5e5
├─╯  [temporal] Add remaining .with() methods
○  slwvqnrr manishearth@google.com 2025-06-18 18:28:52 e68229d4
│  [temporal] Add `wrapped_rust()` helper for writing generic code
│ ○  vtszmlsx manishearth@google.com 2025-06-18 18:25:38 c14b9e56
├─╯  [temporal] Add PlainTime.{compare, with}
○  xrvwxply manishearth@google.com 2025-06-18 18:24:28 f4ac4ff7
│  [temporal] Add IsPartialTemporalObject
│ ○  tlmqwnpn manishearth@google.com 2025-06-18 17:02:40 df3c2fa5
├─╯  Fix comment about enable_rust
◆  ytnpqlum manishearth@google.com 2025-06-18 16:38:37 main@origin 1f5d17d2
│  [temporal] Add GetTemporalRelativeToOption, use it
~

$ jj cl-up
Pushing from commit 556b213a122 [temporal] Add Add/Subtract to all Temporal types
Enumerating objects: 16, done.
Counting objects: 100% (16/16), done.
Delta compression using up to 14 threads
Compressing objects: 100% (11/11), done.
Writing objects: 100% (11/11), 2.77 KiB | 61.00 KiB/s, done.
Total 11 (delta 9), reused 0 (delta 0), pack-reused 0 (from 0)
remote: Resolving deltas: 100% (9/9)
remote: Waiting for private key checker: 3/3 objects left
remote: Processing changes: refs: 1, new: 1, updated: 1, done    
remote: 
remote: SUCCESS
remote: 
remote:   https://chromium-review.googlesource.com/c/v8/v8/+/6654560 [temporal] Add `wrapped_rust()` helper for writing generic code [NEW]
remote:   https://chromium-review.googlesource.com/c/v8/v8/+/6653151 [temporal] Add Add/Subtract to all Temporal types
remote: 
To sso://chromium/v8/v8.git
 * [new reference]           556b213a122d5a0dad5bdd5dd0a1485a303e9ea1 -> refs/for/main
```

Here, it pushed commits `zqlxpsus` and `slwvqnrr`, creating/updating changes for
them. Change `xrvwxply` was also in the history but not pushed since it was
already up to date upstream. Other changes were not pushed since they were not ancestors of `@`.

