# Untangling divergent changes

We've leaned hard on one promise throughout this book: a change ID names one
change, and it survives every rewrite. There is a situation where that promise
bends. It's called a *divergent change*, and it's worth meeting in a repository
we don't care about before it happens in one we do, because the first time you
see it, it looks alarming.

Here's the short version: a change is divergent when *two visible commits*
claim the same change ID. `jj` marks them `(divergent)` in the log, and refuses
to let the bare change ID pick between them.

## Making one on purpose

Let's set up a little repository with a two-change stack and a bookmark on
top:

```console
$ jj log
@  znquvmmz steve@steveklabnik.com 2024-03-20 11:02:31 feature 6f590d66
│  add an evaluator
○  zluwzunl steve@steveklabnik.com 2024-03-20 11:02:14 511b69ca
│  add a parser
◆  zzzzzzzz root() 00000000
```

Now we'll amend the parser change — `jj edit zluwzunl`, fix a file, and come
back up with `jj new znquvmmz`. We know what happens: both changes get new
commit IDs, the evaluator gets rebased, and the old versions go into hiding.
`jj evolog` can show them to us:

```console
$ jj evolog -r znquvmmz --no-graph
znquvmmz steve@steveklabnik.com 2024-03-20 11:03:44 feature cfffda13
add an evaluator
-- operation 7a0508060358 snapshot working copy
znquvmmz/1 steve@steveklabnik.com 2024-03-20 11:02:31 6f590d66 (hidden)
add an evaluator
-- operation a18d13686e71 describe commit 8c764b87308646d95ab0884ffb35e5d3ee20dbfd
```

So far, so normal. Now the mistake. A while later, we're rooting around in the
evolog for something we lost, we find that old commit `6f590d66`, and — maybe
thinking it's the current version — we point our bookmark at it:

```console
$ jj bookmark set feature -r 6f590d66 --allow-backwards
Moved 1 bookmarks to znquvmmz/1 6f590d66 feature* | (divergent) add an evaluator
```

There's the word already. A bookmark keeps a commit visible, and so it keeps
that commit's *ancestors* visible too. We've just resurrected the entire old
version of the stack:

```console
$ jj log
@  tkvuwxwx steve@steveklabnik.com 2024-03-20 11:03:44 c5324b95
│  (empty) (no description set)
○  znquvmmz/0 steve@steveklabnik.com 2024-03-20 11:03:44 cfffda13 (divergent)
│  add an evaluator
○  zluwzunl/0 steve@steveklabnik.com 2024-03-20 11:03:44 c2e9dfd1 (divergent)
│  add a parser
│ ○  znquvmmz/1 steve@steveklabnik.com 2024-03-20 11:02:31 feature 6f590d66 (divergent)
│ │  add an evaluator
│ ○  zluwzunl/1 steve@steveklabnik.com 2024-03-20 11:02:14 511b69ca (divergent)
├─╯  add a parser
◆  zzzzzzzz root() 00000000
```

Both versions of both changes, all four marked `(divergent)`. Note that a
whole *stack* went divergent from one bookmark move. When people hit this in
real life, it's rarely one commit — it's twenty, and it looks like the
repository exploded. It didn't. It's the same two lines of history you can see
right here, just longer.

Moving a bookmark isn't the only way in. Fetching can do it too, if the remote
has commits descended from a version you've since rewritten — a common one is
someone (or CI) building on your branch right before you force-push over it.
Two machines, or two workspaces, rewriting the same change also gets you here.
However you arrive, the shape is the same: an old line and a new line, both
visible.

## Reading the display

Those `/0` and `/1` suffixes are how `jj` names the versions of a change.
You've seen them in `jj evolog` output already, counting back through hidden
predecessors. Now that two versions are *visible*, the suffixes are how we
tell them apart, and the bare change ID stops working:

```console
$ jj show znquvmmz
Error: Change ID `znquvmmz` is divergent
Hint: Use change offset to select single revision: znquvmmz/0, znquvmmz/1
Hint: Use `change_id(znquvmmz)` to select all revisions
Hint: To abandon unneeded revisions, run `jj abandon <commit_id>`
```

This error is usually the moment people discover divergence exists, so it's
nice that the hints tell you everything: pick a side with an offset, or take
all sides with `change_id()`.

One caution about the offsets: they're assigned by sorting the visible
commits, and they get *reassigned* as commits appear and disappear. For
looking around, `znquvmmz/1` is fine. For a multi-step cleanup, use commit
IDs — `6f590d66` means the same commit tomorrow, and `/1` may not.

`jj log -r 'divergent()'` shows every divergent commit in the repository,
which is the quickest way to see the full extent of the problem.

## Cleaning it up

Our repository has a new line we want and an old line we don't. The plan: make
sure nothing we care about is attached to the old line, then abandon the old
line, all of it.

First, the bookmark. It's sitting on the old tip, which is both why the old
line is visible and not where we want it. Let's put it back on the current
version:

```console
$ jj bookmark set feature -r cfffda13 --allow-backwards
Moved 1 bookmarks to znquvmmz/0 cfffda13 feature* | (divergent) add an evaluator
```

Now let's check the old line is really abandoned-safe: no bookmarks left on
it, and nothing built on top of its tip. We use commit IDs for the range,
since the change IDs are exactly what's ambiguous right now:

```console
$ jj log -r '511b69ca::6f590d66 & (bookmarks() | remote_bookmarks())'
$ jj log -r 'children(6f590d66)'
```

Both empty. If they weren't — if a bookmark or some new work were attached to
the old line — we'd stop here and move or rebase those first.

Now we can abandon the old line. This is the same inclusive-range trick from
the revsets chapter, from the old line's first commit through its tip:

```console
$ jj abandon '511b69ca::6f590d66'
Abandoned 2 commits:
  znquvmmz/1 6f590d66 (divergent) add an evaluator
  zluwzunl/1 511b69ca (divergent) add a parser
```

The range matters. If we'd abandoned only the tip, `511b69ca` would have become
the old line's new head — still visible, still divergent, and we'd be playing
whack-a-mole down the whole stack. Abandon the line, not the commit.

And we're back to normal:

```console
$ jj log -r 'divergent()'
$ jj log
@  tkvuwxwx steve@steveklabnik.com 2024-03-20 11:03:44 c5324b95
│  (empty) (no description set)
○  znquvmmz steve@steveklabnik.com 2024-03-20 11:03:44 feature cfffda13
│  add an evaluator
○  zluwzunl steve@steveklabnik.com 2024-03-20 11:03:44 c2e9dfd1
│  add a parser
◆  zzzzzzzz root() 00000000
```

No offsets, no `(divergent)`, one version of each change.

Two more things worth knowing. If the operation that caused the divergence was
the *last* thing you did, `jj undo` fixes it in one step, as usual. And
nothing here was ever dangerous: a divergent change is two visible commits,
both intact, both recoverable. The only real hazard is absent-mindedly
continuing to work on top of the side you meant to throw away — which is why
it's worth cleaning up promptly rather than living with the offsets.

## When both sides have work: `jj converge`

Here the old line was a duplicate we wanted gone, so `jj abandon` was the tool.
The other case is two versions that each carry real, *different* work — two
machines that both edited the same change, say — where you want to keep both
sets of edits rather than throw one away. `jj` 0.45 added `jj converge` for
that:

```console
$ jj converge -r 'divergent()'
Found 1 divergent change(s) in the specified revset:
- Change: znquvmmzlxqr with 2 commits:
    znquvmmz/0 cfffda13 (divergent) add an evaluator
    znquvmmz/1 6f590d66 feature | (divergent) add an evaluator

Attempting to converge change znquvmmzlxqr...

Successfully converged change: created commit 41e0c3d8b2ef.
```

It groups the divergent commits by change ID and tries to replace each group
with a single commit, then rebases descendants and moves bookmarks to follow.
The replacement is built from heuristics, and where they can't decide it prompts
you — to merge two descriptions, to pick parents, and, rarely, to choose an
author. `--no-interactive` prints a warning and stops instead, which is what a
script wants. As always, `jj undo` reverses the whole thing if the result isn't
what you hoped, and `jj op show -p` shows exactly what it did.

The [official guide to divergent changes][divergence] covers a few rarer ways
in and out, if you'd like more.

[divergence]: https://docs.jj-vcs.dev/latest/guides/divergence/
