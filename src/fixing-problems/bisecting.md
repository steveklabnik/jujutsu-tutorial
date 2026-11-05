# Finding the commit that broke it with `jj bisect`

Everything else in this section has been about undoing something we did. This
chapter is about finding out which past change caused a problem in the first
place.

The idea is the same as `git bisect`: we know something worked before and
doesn't work now, so we can use a binary search to find where that changed.
`jj bisect run` does the whole search for us, given a command that can tell a
good revision from a bad one.

Our command needs to exit `0` on a good revision and use most non-zero statuses
for a bad one, which is the convention test runners already follow. There are
two reserved statuses: `125` skips a revision that can't be tested, and `127`
aborts the search. Let's put the command in a script somewhere *outside* the
repository. `jj` will be checking out old revisions, so a script in the working
copy could vanish along with everything else:

```sh
#!/bin/sh
[ ! -e v.txt ] || [ "$(cat v.txt)" -lt 4 ]
```

The first test treats a missing `v.txt` as good. We'll see why in a moment.
Then we can run it over the range we want to search:

```console
$ jj bisect run --range 'mutable() & ~empty()' -- ~/bin/check.sh
Pre-bisection check: ensuring this revision is bad:
zzxyuqmp 49cc9a8b c6
Working copy  (@) now at: qwnuvmxx b1598543 (empty) (no description set)
Parent commit (@-)      : zzxyuqmp 49cc9a8b c6
Pre-bisection check: ensuring this revision is good:
zzzzzzzz 00000000 (empty) (no description set)
Working copy  (@) now at: xlzxqzyk 665382aa (empty) (no description set)
Parent commit (@-)      : zzzzzzzz 00000000 (empty) (no description set)
Added 0 files, modified 0 files, removed 1 files
Bisecting: 5 revisions left to test after this (roughly 3 steps)
Now evaluating: yvqzopnt 8d249e6a c3
Working copy  (@) now at: vyvtyopt 9149b5f2 (empty) (no description set)
Parent commit (@-)      : yvqzopnt 8d249e6a c3
Added 1 files, modified 0 files, removed 0 files
The revision is good.

Bisecting: 2 revisions left to test after this (roughly 2 steps)
Now evaluating: vmqnvymy 446a5e73 c4
Working copy  (@) now at: qvvkwpks f7dc4d7b (empty) (no description set)
Parent commit (@-)      : vmqnvymy 446a5e73 c4
Added 0 files, modified 1 files, removed 0 files
The revision is bad.

Search complete. To discard any revisions created during search, run:
  jj op restore 7f5d71cdc5f8
The first bad revision is: vmqnvymy 446a5e73 c4
```

At each step, `jj` checks out a revision, runs our command, and narrows the
range. The answer at the end is the first revision where the command started
failing.

Before any of that, though, `jj` checks the two ends. The newest revision in
the range has to be bad, and the revision just before the range has to be
good. Otherwise there's no change from good to bad to find. Here, the revision
before the range is the root commit, which has no `v.txt` at all. A script that
only ran `cat v.txt` would fail there, and `jj` would stop before searching:

```text
Cannot bisect: this revision was expected to be good, but was bad (exit status: 2) instead.
```

That's why our script counts a missing file as good. If we already trust both
ends, `--trust-endpoints` skips the check.

## The range

`--range` is a revset, which gives us a lot of flexibility. `git bisect` asks us
to mark one good commit and one bad commit and then walks between them. Here, we
can describe the search space directly:

* `mutable()` — everything `jj` permits you to rewrite. This can include pushed
  commits on tracked remote bookmarks.
* `trunk()..@` — the work on your branch.
* `main@origin..main` — what you're about to push.

Excluding empty commits with `& ~empty()` is often worth it, since testing a
commit that changes nothing tells you nothing.

## Cleaning up after it

Notice what the output offers at the end:

```text
To discard any revisions created during search, run:
  jj op restore 7f5d71cdc5f8
```

Bisecting checks out revisions as it searches, so it leaves working-copy
commits behind. We can use the operation log from earlier in this section to
clean them up. One command puts the repository back exactly as it was before
the search, and `jj` gives us the operation ID rather than making us find it.
