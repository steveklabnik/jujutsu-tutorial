# Splitting a change with `jj split`

`jj squash` takes two changes and makes them one. Sooner or later you'll want
to go the other way: you sat down to fix a bug, noticed some unrelated mess,
tidied it up while you were in there, and now one change is doing two jobs. The
reviewer is going to ask you to separate them, and they're right.

That's `jj split`.

Here's the situation. We meant to add a function, but we wrote some
documentation at the same time:

```console
$ jj st
Working copy changes:
A DOCS.md
A main.rs
Working copy  (@) : ynplyknw dce433ae two unrelated things at once
Parent commit (@-): mllnrzow 4d1201b4 initial
```

Two files, two unrelated jobs, one change. Let's pull the docs out:

```console
$ jj split DOCS.md -m "add some docs"
Selected changes : ynplyknw cd1826ed add some docs
Remaining changes: mxvnxuyl 618910f4 two unrelated things at once
Working copy  (@) now at: mxvnxuyl 618910f4 two unrelated things at once
Parent commit (@-)      : ynplyknw cd1826ed add some docs
```

We named the paths we wanted to peel off, and `-m` gave the new change its
description. Now there are two:

```console
$ jj log --limit 3
@  mxvnxuyl steve@steveklabnik.com 2024-03-20 11:44:17 618910f4
│  two unrelated things at once
○  ynplyknw steve@steveklabnik.com 2024-03-20 11:44:17 cd1826ed
│  add some docs
○  mllnrzow steve@steveklabnik.com 2024-03-20 11:43:36 4d1201b4
│  initial
```

The docs went into the first change, and everything else stayed in the second.
The second still has the old description, "two unrelated things at once", which
is now a lie — so `jj describe` it into something honest. `jj` can split your
change for you but it can't know what the halves should be called.

One detail worth noticing: the *selected* part kept the original change ID,
`ynplyknw`, and the remainder got a fresh one. The part you pull out is treated
as the continuation of the original change.

## Splitting interactively

Naming paths works when the split falls on file boundaries. Often it doesn't —
two changes tangled in the same file. Leave off the paths:

```console
$ jj split
```

and you get the same TUI we saw with `jj squash -i`, where you pick individual
hunks and lines. Everything you select goes into the first change, everything
you don't goes into the second. `jj split -i` does the same thing while still
letting you narrow to some paths first.

## Splitting something other than the working copy

Everything so far has split `@`. Use `-r` for any other change:

```console
$ jj split -r mxvnxuyl extra.txt -m "just the extra file"
Rebased 1 descendant commits.
Selected changes : mxvnxuyl 14798ea1 just the extra file
Remaining changes: nkrkylxx 6f0f5589 the middle commit
Working copy  (@) now at: uwmkvzlt f2e56704 a child commit
Parent commit (@-)      : nkrkylxx 6f0f5589 the middle commit
```

That change was in the middle of a stack with work on top of it, and you can
see `jj` rebased the descendants without being asked. Same story as always: it
just works, and your working copy didn't move.

The automatic rebase is especially useful when the change is in the middle of
a stack. With `git`, we would usually start an interactive rebase, stop at the
commit, separate its contents, and then continue the rebase. Here we can name
any commit we're still allowed to rewrite, and `jj split` takes care of its
descendants for us.

## Two children instead of a parent and a child

By default the halves end up stacked, one on top of the other. If they're
genuinely independent, `-p` makes them siblings instead:

```console
$ jj split -p DOCS.md -m "add some docs"
```

Now you have two changes side by side, both on the original parent — an
anonymous branch, of the sort we made back in the branching chapter. Handy when
you split one change into two pull requests that don't depend on each other.

## Where this fits

Between `jj squash` and `jj split` you can push work in either direction: merge
changes together, break them apart, and move pieces between them. Add `jj absorb`
from a couple of chapters ago, which shoves each hunk into whichever ancestor
already touched those lines, and rearranging history stops feeling like surgery
and starts feeling like editing.
