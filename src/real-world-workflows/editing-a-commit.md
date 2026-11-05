# Editing a commit's diff with `jj diffedit`

`jj split` pulls one change into two, and `jj squash` moves work between
changes. Sometimes we don't want to move anything, though. We just want to fix
what one commit *contains*: perhaps there's a stray debug line three commits
back, or a file that shouldn't have been touched.

With `git`, we might start an interactive rebase, stop at the commit, amend it,
and continue. `jj diffedit` gives us a more direct route. It opens the commit's
diff, lets us edit it, and writes the result back without checking out the
commit.

## Fixing a commit in place

Let's point it at the revision we want to fix:

```console
$ jj diffedit -r 'description(glob:"add feature*")'
```

With the default configuration, we'll get the same TUI we saw with `jj split`
and `jj squash -i`, showing that commit's diff. The left side is the parent, and
the right side is the commit. We can edit the right side until it contains what
we intended, perhaps unticking a line we didn't mean to add or deleting a debug
statement. When we close the editor, `jj` updates the commit and rebases
everything on top of it:

```console
$ jj diffedit -r 'description(glob:"add feature*")'
Rebased 1 descendant commits.
Working copy  (@) now at: pxukppkx 5f724084 (empty) (no description set)
Parent commit (@-)      : nmvqtsqq f1b645ea add feature
```

Our working copy never moved! We didn't need to check out the commit, use
`jj edit`, or run `jj new` afterwards. As with any rewrite, rebasing its
descendants can introduce conflicts that we'll then need to resolve.

## Editing one file with `jj file edit`

The TUI is good for picking lines out of a diff. When we already know which file
is wrong, there's a shorter route. `jj file edit` opens a single file from any
revision in our editor:

```console
$ jj file edit -r 'description(glob:"add feature*")' src/feature.rs
Rebased 1 descendant commits.
Working copy  (@) now at: pxukppkx 9c35793d (empty) (no description set)
Parent commit (@-)      : nmvqtsqq 2f28cbb7 add feature
Added 0 files, modified 1 files, removed 0 files
```

We get the file as it is in that commit, not as it is in our working copy. We
delete the stray debug line, save, and `jj` writes the file back into the
commit and rebases everything above it, including our `@`. If the file is
conflicted in that revision, the conflict markers show up in the editor, and
fixing them there resolves the conflict.

Its sibling, `jj file delete`, takes a file out of a revision altogether. That
suits a log file we committed by accident:

```console
$ jj file delete -r 'description(glob:"add feature*")' debug.log
Rebased 1 descendant commits.
Working copy  (@) now at: pxukppkx 31531a98 (empty) (no description set)
Parent commit (@-)      : nmvqtsqq a3c6c92e add feature
Added 0 files, modified 0 files, removed 1 files
```

Again, our working copy stays where it was.

## What you can and can't do here

`jj diffedit` edits the *content* of one commit against its parent. You're
changing what the patch does, not where it sits. For the neighboring jobs:

* Moving a whole file's contents in from another revision is
  [`jj restore`](../fixing-problems/reverting-changes.md).
* Moving a hunk *between* two commits is `jj squash -i`.
* Splitting one commit into two is `jj split`.

That leaves `jj diffedit` for the case where one commit's diff is almost right
and we want to fix it in place.

## Comparing two revisions instead

We can also use `--from` and `--to` to edit the diff between any two revisions.
`diffedit` will write the result into the `--to` side:

```console
$ jj diffedit --from A --to B
```

The `-r` form is just the common case of this, where `--from` is the parent.
Most of the time `-r` is what you want.

Like every rewrite, editing a commit gives it a new commit ID and rebases its
descendants, so the immutable rule applies: `jj` won't let you `diffedit` a
commit on trunk.
