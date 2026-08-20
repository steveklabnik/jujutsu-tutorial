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
[ "$(cat v.txt)" -lt 4 ]
```

Then we can run it over the range we want to search:

```console
$ jj bisect run --range 'mutable() & ~empty()' -- ~/bin/check.sh
Bisecting: 3 revisions left to test after this (roughly 2 steps)
Now evaluating: upqnzxnm d8afdf06 c5
Working copy  (@) now at: kzlyrqkk e7a2a494 (empty) (no description set)
Parent commit (@-)      : upqnzxnm d8afdf06 c5
The revision is bad.

Bisecting: 1 revisions left to test after this (roughly 1 steps)
Now evaluating: lrmywvmw 4e2d0df0 c4
Working copy  (@) now at: oqonuxym d9edf35c (empty) (no description set)
Parent commit (@-)      : lrmywvmw 4e2d0df0 c4
The revision is bad.

Search complete. To discard any revisions created during search, run:
  jj op restore e31971608abc
The first bad revision is: lrmywvmw 4e2d0df0 c4
```

At each step, `jj` checks out a revision, runs our command, and narrows the
range. The answer at the end is the first revision where the command started
failing.

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
  jj op restore e31971608abc
```

Bisecting checks out revisions as it searches, so it leaves working-copy
commits behind. We can use the operation log from earlier in this section to
clean them up. One command puts the repository back exactly as it was before
the search, and `jj` gives us the operation ID rather than making us find it.
