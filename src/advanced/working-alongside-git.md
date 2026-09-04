# Working alongside `git`

Every repository in this book is colocated: a `.jj` directory and a real `.git`
directory over one working copy. This is the default because it lets the rest
of our toolchain keep working. `gh`, our editor, CI, and plain `git` all see an
ordinary `git` repository. Let's look at the rules that keep the two views in
sync.

## Everything syncs on every command

`jj` imports from `git` and exports to it at the start and end of nearly every
command. We don't need to run `jj git import` or `jj git export` by hand in a
colocated repository; it happens for us. That means `git log` shows the commits
we made with `jj`:

```console
$ git log --oneline
c4b8fac hello world
```

And a `jj` *bookmark* is a `git` branch. Move a bookmark in `jj` and the branch
moves when the command exports it. Create a branch with `git` and a bookmark of
the same name appears the next time `jj` imports it. They are two names for one
ref.

## `git HEAD` points at a *parent* of `@`

`git` has a current branch and a `HEAD`. `jj` has neither. To bridge the gap,
`jj` keeps `git`'s `HEAD` detached, pointing at a *parent* of our working-copy
commit `@`. Most of the time this is invisible. It stops being invisible after
`jj edit` on a commit that already has content:

```console
$ jj edit 'description(glob:"add feature*")'
$ git status --short
 M src/feature.rs
```

`git` reports `src/feature.rs` as modified. It is not uncommitted work. `@` is a
finished commit whose files are checked out, `HEAD` points at its parent, and
`git` is honestly reporting the difference between the two. `jj` already has the
change recorded.

When we're done editing that commit, we can move `@` back on top:

```console
$ jj new
```

Now `@` is a fresh empty change, `HEAD` sits at its parent, and `git status` is
clean again without moving any bookmark or touching pushed history. The same
thing happens with `jj commit`, which closes `@` and opens a new empty one for
us. If we leave a non-empty change in `@`, `git`-facing tools will report the
working tree as dirty. Finishing with `jj new` or `jj commit` makes it clean.

## Let `jj` do the mutating

Both tools write to the same object store. If they write at once, their import
and export steps can race. There are two rules I follow:

* Don't run `git` and `jj` at the same time against one repository. Run one,
  then the other.
* Prefer `jj` for anything that changes history, such as committing, merging,
  or rebasing.
  `jj` ignores `git`'s staging area and can't model a half-finished `git rebase`
  or an unresolved `git` index conflict. A `git` mutation that *completes*
  cleanly can usually be imported afterwards, but never hand `jj` an operation
  `git` left in progress.

Reading with `git` is safe. We can use `git log`, `git show`, `git status`, or
`git diff` whenever they are useful.

## What happens if you mutate with `git` anyway

This doesn't mean that every `git` mutation corrupts the repository. Completed
operations can often be imported successfully, but they make the interaction
more complicated. Let's see what actually happens.

A `git` commit that *finishes* is imported on the next `jj` command. `git` keeps
`HEAD` detached; attach it to a branch first if you want the commit to have a
bookmark when `jj` imports it:

```console
$ git checkout -b gitwork
$ echo change > f.txt && git add f.txt && git commit -m "made by raw git"
$ jj log
Reset the working copy parent to the new Git HEAD.
Done importing changes from the underlying Git repo.
```

`jj` picked up the commit as an ordinary change, with `gitwork` as a bookmark,
and put a fresh empty `@` on top. Excellent! In this case, everything came
across cleanly.

Now the sharper edge. A `git reset --hard` that moves the branch backwards is
imported too:

```console
$ git reset --hard <earlier-commit>
$ jj log
Reset the working copy parent to the new Git HEAD.
Done importing changes from the underlying Git repo.
```

The Git branch and `@` now follow the earlier commit, but commits that `jj`
already knew about remain as anonymous heads. We can find them with `jj log`,
or explicitly with `jj log -r 'heads(all())'` if our log revset hides them, and
point the bookmark back at the change we want:

```console
$ jj bookmark set gitwork -r <discarded-change>
```

`jj undo` and `jj op restore` can also reverse imported Git movements. Inspect
`jj op log` first: importing Git's `HEAD` and refs can produce separate
operations, so the operation before the reset is a more reliable target than
assuming one `jj undo` is enough. Data that `jj` never snapshotted is a different
matter: `git reset --hard` can still destroy uncommitted working-tree changes.

The failure mode the `jj` docs warn about is subtler than lost data: interleaving
`git` and `jj` mutations can leave a bookmark whose `git` position and `jj`
position disagree. `jj` then shows the Git-side position separately as
`name@git`. The conflict itself never loses data, but untangling it is a chore,
and an editor's automatic `git fetch` can trigger it without you typing
anything. The cure is the same: prefer `jj` for the mutation, and reach for
`jj undo` when `git` got there first.

## Two cases to avoid

Most completed `git` mutations import cleanly, as above. Two cases don't,
because `jj` snapshots the working copy without understanding the state that
an unfinished Git operation keeps in `.git`.

First, never hand `jj` a `git` operation that's still in progress. A
`git rebase`, `git merge`, or `git cherry-pick` that hits a conflict stops and
waits, leaving conflict markers in your files and its own state under `.git`.
`jj` knows none of that. The next `jj` command snapshots those files as they sit
— and the raw markers become plain text inside `@`, *not* a `jj` conflict:

```console
$ git rebase B          # conflicts, and stops half-done
$ jj st
Working copy changes:
M f.txt
$ jj file show -r @ f.txt
<<<<<<< HEAD
BBB
=======
AAA
>>>>>>> 1ce7f83 (A change)
```

Those `<<<<<<<` lines are committed content now, `jj` sees an ordinary edit
rather than a conflict, and `git` still thinks the rebase is running. Finish the
`git` operation or `git rebase --abort` it *before* you run a single `jj`
command. Otherwise, `jj` will record the conflict markers as ordinary file
contents rather than as a conflict.

Second, the `git` index is invisible to `jj`. `jj` has no staging area and
ignores `git`'s. `git add`, and especially `git add -p` to stage part of a file,
do nothing `jj` respects — `jj` snapshots the whole working tree no matter
what's staged. Use
[`jj split`](../real-world-workflows/splitting-changes.md) or `jj squash -i` to
pick apart a change instead.

For the record, one thing that *isn't* a hazard: `jj` keeps `.jj` out of `git`
with its own `.gitignore` inside that directory, so a stray `git add -A` can't
swallow `jj`'s internal state.

## Adopting an existing `git` repository

Everything in this book started life as a `jj` repository, but the other
direction works too: run `jj git init` inside an existing `git` checkout, and
`jj` adopts it in place:

```console
$ jj git init
Done importing changes from the underlying Git repo.
Initialized repo in "."
$ jj log
@  syomluky steve@steveklabnik.com 2024-03-24 10:12:08 c23e3885
│  (empty) (no description set)
○  ypzkvlqw steve@steveklabnik.com 2024-03-24 10:11:52 main 9283911f
│  existing history
◆  zzzzzzzz root() 00000000
```

The history comes across, branches become bookmarks, and a fresh empty `@`
lands on top of wherever `HEAD` was. Nothing about the `git` repository is
changed except the new `.jj` directory next to it, so if `jj` turns out not to
be for you, deleting `.jj` puts everything back.

One gotcha to check first: `git` has two formats for storing refs, and `jj`
0.45 can only read the older `files` format. On a repository using the newer
`reftable` format, `jj git init` appears to succeed but imports *nothing* — no
bookmarks, and a working copy that claims every file is newly added. If you
see that, don't panic, and don't commit anything: delete the freshly created
`.jj` directory, migrate the refs, and initialize again:

```console
$ git rev-parse --show-ref-format
reftable
$ git refs migrate --ref-format=files
```

Most repositories are `files` and adopt cleanly.

## If you'd rather fence `git` off

Colocation trades a small cost — the import/export on every command, and the odd
confusing intermediate state above — for zero-friction interop. If you'd rather
`git` couldn't interfere at all, you can hide it entirely. That's the subject of
[Non-colocated repositories](non-colocated-repos.md).
