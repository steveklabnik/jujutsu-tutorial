# Watching a change evolve with `jj evolog`

We've said a few times now that a change ID is stable while the commit ID
underneath it changes. Every time you describe a change, edit its files, squash
something into it, or let it get automatically rebased, you get a new commit ID
and the same change ID.

`jj evolog` shows you that history. Here's a change that's been through a few
rounds:

```console
$ jj evolog -r usvzmupx
○  usvzmupx steve@steveklabnik.com 2024-03-20 10:02:11 c39e9f3f
│  add a better feature
│  -- operation c4e5378aa59c snapshot working copy
○  usvzmupx/1 steve@steveklabnik.com 2024-03-20 10:01:58 0ea8e5fc (hidden)
│  add a better feature
│  -- operation 3286efef174e describe commit e7b530dc15e0
○  usvzmupx/2 steve@steveklabnik.com 2024-03-20 10:01:58 e7b530dc (hidden)
│  add a feature
│  -- operation 685d3932d80a snapshot working copy
○  usvzmupx/3 steve@steveklabnik.com 2024-03-20 10:01:40 6d256008 (hidden)
│  add a feature
│  -- operation 91adfe1a92ae describe commit 39f08e04f359
○  usvzmupx/4 steve@steveklabnik.com 2024-03-20 10:01:40 39f08e04 (hidden)
│  (no description set)
│  -- operation cbf76d26122b snapshot working copy
○  usvzmupx/5 steve@steveklabnik.com 2024-03-20 10:01:22 6161a9a2 (hidden)
   (empty) (no description set)
   -- operation 4261b1b2ee68 add workspace 'default'
```

If we read it from bottom to top, we can watch the change evolve. It starts as
the empty commit `jj git init` made. It gets some content. It gets described as
"add a feature". More content. Then a better description. Every one of those
steps left a commit behind, and they're all still here.

That `usvzmupx/1` notation is how `jj` refers to older versions of a change: the
change ID, then which step back you mean. And `(hidden)` means what it did in
`jj op show` — the commit exists, it's just not part of the visible history any
more.

Each entry also names the operation that produced it, on the `-- operation`
line, which is the bridge back to the last chapter. If you spot the version you
want here, you can find the operation that came after it and `jj op restore` to
just before the step you regret.

If we pass `-p`, we get the diff at each step. This is the version I usually
reach for, because it shows us what the change *contained* over time, not just
what it was called.

## When two versions stay visible

Normally, only the newest entry in an evolog is visible. There is one awkward
case worth recognizing: we can end up with both versions of a rewritten stack
in `jj log`. They are marked `(divergent)` because each change ID now names two
visible commit IDs.

The rewrite alone does not cause this. Normally it creates a successor for
each commit, keeps the change IDs, and hides the predecessors. A hidden commit
can become visible again, though, if we put a bookmark on it, edit it, make it
a working copy, or add a visible descendant. The [Jujutsu guide to divergent
changes][divergence] calls out each of these cases, as well as two processes
rewriting the same change at the same time.

We can reconstruct what happened in this repository from the operation log.
After the rewrite, the old `new-chapters@origin` and `pdf-print@origin` targets
were both marked `(hidden)`, and `jj log -r 'divergent()'` was empty. A later
operation accidentally moved `pdf-print` back to the old commit `37b65b73`:

```console
$ jj op log --no-graph
7152344d1e5b you@example.com default@ 2026-08-22 15:29:06, lasted 7 milliseconds
point bookmark pdf-print to commit 37b65b734abcd1e2f3a4b5c6d7e8f9a0b1c2d3e4
args: jj bookmark set pdf-print -r 37b65b73
58d5d578ddfc you@example.com default@ 2026-08-22 15:28:58, lasted 9 milliseconds
new empty commit
args: jj new
9889fa7911c5 you@example.com default@ 2026-08-22 15:28:51, lasted 12 milliseconds
snapshot working copy
args: jj new
```

Putting the bookmark there made that commit and all its ancestors visible.
Those ancestors were the predecessor versions of the rewritten stack, so we
now had two visible commit IDs for each of 28 change IDs. This is why a whole
series appeared divergent at once. It was a consequence of that accidental
bookmark move, not something that every rewrite produces. Another way to get
the same result would be to fetch a new commit whose history descends from one
of our hidden predecessors.

This can be surprising when the bookmark looks right. In a repository where I
ran into this, the local bookmark, its Git copy, and the tracked remote bookmark
all pointed at the new commit:

```console
$ jj bookmark list --all
new-chapters: ynossmku 0854177d clarify repository mode defaults
  @git: ynossmku 0854177d clarify repository mode defaults
  @origin: ynossmku 0854177d clarify repository mode defaults
```

The old stack was still an anonymous visible head. Moving and pushing a
bookmark selects the new line, but it does not abandon another visible head.
We can see both versions without guessing from the graph:

```console
$ jj log -r 'heads(divergent())' --no-graph
vxmynnln/0 you@example.com 2026-08-22 15:28:51 0c71df12 (divergent)
explain change ID stability across clones
vxmynnln/19 you@example.com 2026-08-22 15:07:55 11ff4879 (divergent)
note jj change ID stability across plain git clones
```

Here `0c71df124215` is on the line retained by `new-chapters`, while
`11ff48792102` is the tip of the old line. The repeated change ID tells us that
these are divergent versions of the same logical change. These are the two
tips. We can find their last shared commit with `fork_point()`.

`fork_point()` is shorthand for intersecting the two sets of ancestors and
selecting their heads: `heads(::0c71df124215 & ::11ff48792102)`.

Let's see:

```console
$ jj log -r 'fork_point(0c71df124215 | 11ff48792102)' --no-graph
ppxpvmnv you@example.com 2026-08-20 16:31:00 update-for-jj-0.44 8827278a
docs: correct conflict graph glyphs
```

Here `8827278aa4c9` is the nearest ancestor shared by both tips. If we want to
see the first commit on each side of the split as well, we can remove that
commit and its ancestors from the two histories, then ask for the roots of
what remains:

```console
$ jj log -r \
    'roots(8827278aa4c9..(0c71df124215 | 11ff48792102))' \
    --no-graph
orqqvtqp/0 you@example.com 2026-08-22 15:25:02 0bf91872 (divergent)
explain commit shorthand and push dry runs
orqqvtqp/1 you@example.com 2026-08-20 16:34:40 d5487b35 (divergent)
docs: document commit shorthand and push dry runs
```

The repeated `orqqvtqp` change ID shows us exactly where the two series
separated: these are the first two visible versions after their shared parent.
`jj log -r 'divergent()'` shows every affected version, and `jj log -r
'heads(all())'` helps us spot unbookmarked heads.

Before throwing the old line away, let's check that no bookmark points into it
and that nothing descends from its tip. We use commit IDs throughout, because
the change IDs are precisely what is ambiguous here:

```console
$ jj log -r \
    'd5487b35a605::11ff48792102 & (bookmarks() | remote_bookmarks())'
$ jj log -r 'children(11ff48792102)'
```

Both commands produced no commits in this case. We can now abandon the whole
old range, from its first divergent commit through its tip:

```console
$ jj abandon 'd5487b35a605::11ff48792102'
Abandoned 27 commits:
  vxmynnln/19 11ff4879 (divergent) note jj change ID stability across plain git clones
  ...
$ jj log -r 'divergent()'
```

That last log is empty. Abandoning only `11ff48792102` would not have finished
the job: its parent would become the old line's next anonymous head, leaving the
other 26 changes divergent. If the checks find a bookmark or descendants, we
should stop and decide whether to move or rebase them before abandoning
anything.

[divergence]: https://docs.jj-vcs.dev/latest/guides/divergence/

## evolog versus op log

These two commands can be easy to confuse, so let's put the distinction in one
line each.

`jj op log` is a history of your **repository**: what you did, in order, to
everything.

`jj evolog` is a history of one **change**: every version that one logical piece
of work has ever had.

They're two different views of the same data. When we know something went wrong
but not which change it affected, we can use `jj op log`. When we know which
change went wrong but not when, we can use `jj evolog`.

One last note: `jj evolog` used to be called `jj obslog`, for "obsolete log".
That name still works if you type it, and you'll see it in older blog posts and
Discord messages, but `evolog` — for "evolution" — is what it's called now, and
it's a much better description of what you're looking at.
