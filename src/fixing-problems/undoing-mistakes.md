# Undoing mistakes with `jj undo`

Let's make a mess on purpose. Here's a small repository with three changes:

```console
$ jj log
@  ozylkotm steve@steveklabnik.com 2024-03-20 09:14:31 3c267bd1
│  add goodbye
○  rxwzpxqt steve@steveklabnik.com 2024-03-20 09:14:20 0570dd0d
│  add a comment
○  zzzzmznr steve@steveklabnik.com 2024-03-20 09:14:20 3e44726c
│  hello world
◆  zzzzzzzz root() 00000000
```

Now let's do something we regret. We meant to abandon some other change, but we
typed the wrong one and threw away "add a comment":

```console
$ jj abandon rxwzpxqt
Abandoned 1 commits:
  rxwzpxqt 0570dd0d add a comment
Rebased 1 descendant commits onto parents of abandoned commits.
Working copy  (@) now at: ozylkotm 404f6b4e add goodbye
Parent commit (@-)      : zzzzmznr 3e44726c hello world
Added 0 files, modified 1 files, removed 0 files
```

It's gone, and `jj` helpfully rebased "add goodbye" onto "hello world" for us,
so our work is now sitting on top of a commit that doesn't have the comment in
it:

```console
$ jj log
@  ozylkotm steve@steveklabnik.com 2024-03-20 09:14:31 404f6b4e
│  add goodbye
○  zzzzmznr steve@steveklabnik.com 2024-03-20 09:14:20 3e44726c
│  hello world
◆  zzzzzzzz root() 00000000
```

In `git`, recovering this change would take a few steps. In `jj`, we can run one
command:

```console
$ jj undo
Undid operation: 897f241ce8c8 (2024-03-20 09:14:31) abandon commit 0570dd0df085
Restored to operation: 4079800aed31 (2024-03-20 09:14:20) snapshot working copy
Working copy  (@) now at: ozylkotm 3c267bd1 add goodbye
Parent commit (@-)      : rxwzpxqt 0570dd0d add a comment
Added 0 files, modified 1 files, removed 0 files
```

And we're back:

```console
$ jj log
@  ozylkotm steve@steveklabnik.com 2024-03-20 09:14:20 3c267bd1
│  add goodbye
○  rxwzpxqt steve@steveklabnik.com 2024-03-20 09:14:20 0570dd0d
│  add a comment
○  zzzzmznr steve@steveklabnik.com 2024-03-20 09:14:20 3e44726c
│  hello world
◆  zzzzzzzz root() 00000000
```

Notice that `rxwzpxqt` came back with the same change ID *and* the same commit
ID, `0570dd0d`. This isn't a fresh commit that happens to have the same
contents. It's the same commit; it was never really destroyed, just made
invisible.

## What `jj undo` actually undoes

Here's the important bit: `jj undo` undoes the last *operation*, not the last
*commit*. This is what makes it different from the closest commands in `git`.

An operation is a recorded transition between repository states. Commands such
as `jj abandon`, `jj describe`, `jj rebase`, and `jj new` create operations.
Read-only commands don't. And — this surprises people — the automatic snapshot
`jj` takes of your working copy can be a separate operation from the command
that triggered it. Each operation records the complete state of the repository
afterwards.

So `jj undo` doesn't reason about what your command meant, or try to construct
an inverse of it. It puts the repository back the way it was. That's why it
works uniformly on every command, including ones that touched a dozen commits
at once, and why the output tells you which operation it undid and which one it
went back to.

## Running it more than once

There's one detail that can be surprising. If we run `jj undo` a second time,
it does *not* undo the first undo:

```console
$ jj undo
Undid operation: 4079800aed31 (2024-03-20 09:14:20) snapshot working copy
Restored to operation: 84248e9a9cb9 (2024-03-20 09:14:20) new empty commit
```

Look at what it undid: the working copy snapshot from *before* the abandon.
Repeated `jj undo` walks further and further back into the past, one operation
at a time. It's a rewind button, not a toggle.

If we do want to go the other way, there's `jj redo`:

```console
$ jj redo
Restored to operation: c8fa9084a76c (2024-03-20 09:14:20) undo: restore to operation 4079800aed31
Working copy  (@) now at: ozylkotm 3c267bd1 add goodbye
Parent commit (@-)      : rxwzpxqt 0570dd0d add a comment
```

`jj redo` moves in the direction of the future, so between the two of them you
can scrub back and forth until you find the state you wanted.

## Undoing something other than the last thing

Rewinding one step at a time is fine when you've just made one mistake. When
you've made four, and you only want to reach past three of them, you want to
name the state you're going back to directly. `jj undo` won't help there — it
takes no arguments, and only ever walks backwards from where you are.

For that you need to see the operations and refer to them by name, and that's
the operation log. Let's look at it.
