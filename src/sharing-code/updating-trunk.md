# Updating trunk from upstream

Everything so far has assumed the repository stands still while we work. In a
real project, other people will merge things into `main`, and our work will
eventually need to catch up. Let's see how to do that.

## Getting the new commits

`jj git fetch` pulls new commits and bookmark positions down from the remote:

```console
$ jj git fetch
bookmark: main@origin [updated] tracked
```

Notice what it *didn't* do: nothing in our working copy moved, and no merge
happened. Where `git pull` combines fetching with an integration step, `jj`
keeps those two jobs separate. We can fetch first, inspect the result, and then
choose how to move our work.

Let's look at where things sit afterwards:

```console
$ jj log
@  nxoupqvp steve@steveklabnik.com 2024-03-22 10:12:04 60d3e181
│  more work
○  tqmvurqz steve@steveklabnik.com 2024-03-22 10:12:04 my-feature d04bd54d
│  my feature
│ ◆  ztzvtupk steve@steveklabnik.com 2024-03-22 10:14:31 main 2fd78044
├─╯  upstream commit
◆  qoypmutx steve@steveklabnik.com 2024-03-22 09:58:12 23a248a4
│  initial
~
```

There's the drift, drawn out. `main` moved forward, our two changes are still
sitting on the old commit, and the two lines have forked apart.

The local `main` bookmark moved on its own, because it *tracks* `main@origin`
and we hadn't touched it locally. This is the normal case, and it's why you
rarely think about the local copy of trunk at all.

Also notice the diamonds: `main` and the initial commit are immutable. The
default *immutable heads* are
`trunk() | tags() | untracked_remote_bookmarks()`, and those heads plus all
their ancestors are immutable. A tracked remote bookmark for a pull request is
deliberately not an immutable head, so pushed review branches can still be
rewritten.

## Moving your work onto it

Now let's catch up. `jj rebase -b @ -o trunk()` takes the whole branch our
working copy is on and puts it on trunk:

```console
$ jj rebase -b @ -o 'trunk()'
Rebased 2 commits to destination.
Working copy  (@) now at: nxoupqvp 355a63e0 more work
Parent commit (@-)      : tqmvurqz 71ade80c my-feature | my feature
Added 1 files, modified 0 files, removed 0 files
```

```console
$ jj log
@  nxoupqvp steve@steveklabnik.com 2024-03-22 10:16:02 355a63e0
│  more work
○  tqmvurqz steve@steveklabnik.com 2024-03-22 10:16:02 my-feature 71ade80c
│  my feature
◆  ztzvtupk steve@steveklabnik.com 2024-03-22 10:14:31 main 2fd78044
│  upstream commit
~
```

One line again. Both changes kept their change IDs — `nxoupqvp` and `tqmvurqz`
are the same changes they were before — and got new commit IDs, because their
contents now sit on a different parent. The bookmark came along for the ride.

`jj rebase` takes two halves: which revisions move, and where they land.

There are three flags we can use to choose what moves:

* `-b` (branch) moves everything connected to the given revision that isn't
  already an ancestor of the destination. This is the one you want for "catch my
  work up to trunk".
* `-s` (source) moves that revision and its descendants.
* `-r` (revision) moves exactly one revision, and reparents its descendants onto
  its old parent.

`-o` (onto) chooses where they land, leaving the destination's existing
descendants where they are. Its two siblings splice into a stack instead:
`-A` inserts after the target and replants the target's descendants on top of
the moved work, and `-B` inserts before it.

`trunk()` is a revset function resolving to the main bookmark. `jj git clone`
sets it up for you — you'll see `revset-aliases."trunk()" = "main@origin"` in the
repository's config — so it points at whatever the project actually calls its
trunk, and you can write `trunk()` in scripts and aliases without caring.

If the rebase produces conflicts, they get recorded in the rebased commits and
you carry on, exactly as in the conflicts chapter. Nothing stops halfway, and
there's no `--continue` to remember.

## Seeing what's yours

The revset `trunk()..@` means "everything from trunk up to my working copy",
which is a precise way of saying "my work":

```console
$ jj log -r 'trunk()..@'
@  nxoupqvp steve@steveklabnik.com 2024-03-22 10:16:02 355a63e0
│  more work
○  tqmvurqz steve@steveklabnik.com 2024-03-22 10:16:02 my-feature 71ade80c
│  my feature
~
```

That one earns a place in your config as an alias. We'll do that in the
customization section.

## When trunk has diverged

Sometimes we commit directly onto `main` locally, and upstream moves too. Now
both ends of the bookmark have moved, and `jj` won't guess which one we meant:

```console
$ jj git fetch
bookmark: main@origin [updated] tracked

$ jj log
@  oqtwrqnv steve@steveklabnik.com 2024-03-22 10:20:11 main?? main@git 914cd522
│  (empty) local commit on main
│ ◆  quvlnmky steve@steveklabnik.com 2024-03-22 10:20:37 main?? main@origin edd53bde
├─╯  upstream commit 2
```

The `??` marks a *conflicted bookmark*: one name, two candidate positions. It's
not a file conflict, and it doesn't block you — but a bookmark pointing at two
commits can't be pushed anywhere sensible.

`jj bookmark list` lays out the disagreement:

```console
$ jj bookmark list
main (conflicted):
  - ztzvtupk 2fd78044 upstream commit 1
  + oqtwrqnv 914cd522 (empty) local commit on main
  + quvlnmky edd53bde upstream commit 2
  @git (behind by 1 commits): oqtwrqnv 914cd522 (empty) local commit on main
  @origin (behind by 1 commits): quvlnmky edd53bde upstream commit 2
my-feature: tqmvurqz 71ade80c my feature
Hint: Some bookmarks have conflicts. Use `jj bookmark set <name> -r <rev>` to resolve.
```

The `-` line is the common ancestor the two sides started from, and the `+`
lines are the two positions. Fix it the same way you'd fix the drift: put your
local commit on top of the remote one.

```console
$ jj rebase -r oqtwrqnv -o main@origin
Rebased 1 commits to destination.
Working copy  (@) now at: oqtwrqnv 07effb17 main* | (empty) local commit on main
Parent commit (@-)      : quvlnmky edd53bde main@origin | upstream commit 2
```

```console
$ jj bookmark list
main: oqtwrqnv 07effb17 (empty) local commit on main
  @origin (behind by 1 commits): quvlnmky edd53bde upstream commit 2
```

The `??` is gone. The bookmark followed the commit it was pointing at, and once
that commit sat on top of `main@origin` there was nothing left to disagree
about. The `*` in `main*` means the local bookmark is ahead of the remote — the
push you'd expect to make next.

If you'd rather throw the local side away than keep it,
`jj bookmark set main -r main@origin` names the winner directly. Your commit
isn't lost; it's still in the log, just no longer wearing the bookmark.

## The habit

I like to fetch and rebase onto trunk regularly. The rebase is cheap, it never
interrupts us, and any conflicts are recorded rather than blocking the command.
That keeps catching up a small part of the ordinary workflow.
