# Workspaces

Two chapters ago, we worked on several branches at once without leaving our
working copy, by editing commits directly. That covers most of what people use
`git` branch switching for. Sometimes, though, we really do want two directories
on disk at the same time.

For example, we may want a long build or test run in one tree while we keep
editing in another. We may also want two versions of the code open side by side,
or a scratch checkout for a tool that does a lot of work whenever files change.

`git` calls these worktrees. `jj` calls them workspaces.

## Making one

```console
$ jj workspace add ../feature-b
Created Git worktree for the new workspace.
Created workspace in "../feature-b"
Working copy  (@) now at: omvuuyon b1c8e7a0 (empty) (no description set)
Parent commit (@-)      : posvwlwz 6c45c26f initial
Added 1 files, modified 0 files, removed 0 files
```

There's now a second working directory at `../feature-b`, backed by the same
repository. It got its own fresh empty change, `omvuuyon`. This is the important
part: *each workspace has its own `@`*.

The first line of output tells us something else. Our repository is colocated,
so `jj` colocated the new workspace too, by creating a `git` worktree for it.
`git status` and `gh` work in `../feature-b` just as they do in the original
directory. This needs `git` 2.42 or newer. If we'd rather have a plain `jj`
workspace with no `.git` in it, `--no-colocate` skips the worktree.

```console
$ jj workspace list
default: . vtmvqlrp 825c9b05 feature a
feature-b: ../feature-b omvuuyon b1c8e7a0 (empty) (no description set)
```

The workspace we started in is called `default`. Our new one took its name from
the directory. We can use `--name` to pick a different name, and `-r` to start
at a revision other than the default one.

The log shows both, marked with `@` suffixed by the workspace name:

```console
$ jj log
@  vtmvqlrp steve@steveklabnik.com 2024-03-23 11:04:16 default@ 825c9b05
│  feature a
│ ○  omvuuyon steve@steveklabnik.com 2024-03-23 11:04:16 feature-b@ b1c8e7a0
├─╯  (empty) (no description set)
○  posvwlwz steve@steveklabnik.com 2024-03-23 11:03:52 6c45c26f
│  initial
◆  zzzzzzzz root() 00000000
```

`default@` is our working copy; `feature-b@` is the other workspace's. Two
anonymous branches, two directories, one repository.

Now let's `cd ../feature-b`. It behaves like any other `jj` repository, with the
same history, bookmarks, and operation log. Work we do here is visible from the
other workspace immediately, because there's only one repository underneath.

```console
$ jj st
Working copy changes:
A b.txt
Working copy  (@) : omvuuyon 66ff7d1e feature b
Parent commit (@-): posvwlwz 6c45c26f initial
```

## What it isn't

`git worktree` makes you assign a branch to each tree, and refuses to check out
a branch that's already checked out somewhere else. None of that applies here,
even though there's a `git` worktree underneath. Like the main directory, it
sits on a detached `HEAD` at the parent of its `@`. A workspace just points its
`@` at a revision. There's no branch to reserve and nothing to be exclusive
about.

There is one rule to remember: we shouldn't edit the *same* commit from two
workspaces. `jj` doesn't forbid it, but both directories would then hold files
claiming to be the same commit, and whichever one snapshots last would win.

## Stale working copies

Workspaces share history, so a rewrite in one can affect another workspace's
`@`. Let's say we rebase or abandon a commit that another workspace is sitting
on. That workspace will fall behind:

```console
$ jj st
Error: The working copy is stale (not updated since operation 8f799e48797c).
Hint: Run `jj workspace update-stale` to update it.
See https://docs.jj-vcs.dev/latest/working-copy/#stale-working-copy for more information.
```

Nothing has been damaged. The repository knows exactly where that commit went;
the files on disk just haven't caught up yet. We can update them as the hint
suggests:

```console
$ jj workspace update-stale
Working copy  (@) now at: omvuuyon 08f28542 renamed from elsewhere
Parent commit (@-)      : vtmvqlrp 825c9b05 feature a
Added 1 files, modified 0 files, removed 0 files
Updated working copy to fresh commit 08f28542e3a9
```

The change ID is unchanged — `omvuuyon` is the same change it always was — and
any uncommitted edits in that directory are still there. It's the same change,
in its new position.

Not every rewrite makes a workspace stale. If we change a description from
somewhere else, we haven't touched any files, so the next command in the other
workspace absorbs it without comment. Staleness is about the files on disk
disagreeing with the commit they represent. If we'd rather update them
automatically, we can set `snapshot.auto-update-stale = true` in our config.

## Undoing across workspaces

There's one shared operation log, so the most recent operation might have come
from the other directory. `jj undo` won't reach across to it:

```console
$ jj undo
Error: Refusing to undo operation ba337735fb7c because it was performed in workspace feature-b
Hint: Use `--allow-cross-workspace` to undo it anyway, or use `jj op revert` to revert a specific operation
```

This protects us from undoing work in a directory we aren't looking at. If
that's what we want, `--allow-cross-workspace` does it, and `jj redo` takes the
same flag.

## Cleaning up

Deleting the directory isn't enough, because the repository still has the
workspace registered. `jj workspace remove` does the whole job:

```console
$ jj workspace remove feature-b
Removed Git worktree for "/home/steve/src/feature-b".
Removed workspace directory "/home/steve/src/feature-b".
```

It snapshots the workspace first, so any edits we hadn't committed end up in
its `@` rather than being lost. The commits that workspace made are ordinary
commits and stay in the repository; only the `feature-b@` marker goes away.

If we want to keep the files, `jj workspace forget feature-b` unregisters the
workspace and removes its `git` worktree, but leaves the directory alone.

One more command worth knowing, mostly for scripts:

```console
$ jj workspace root
/home/steve/src/my-project
```

That's the top of the current workspace, wherever in the tree you happen to be
standing.

## When to reach for one

Workspaces cost a full checkout on disk and a little bookkeeping. If we only
need to move between pieces of work, editing commits in one directory is
simpler. I reach for a workspace when I genuinely need two trees at once, such
as something long-running in one while I keep working in the other.
