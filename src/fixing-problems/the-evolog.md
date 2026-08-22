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
case worth recognizing: we can rewrite a stack, move its bookmark to the new
tip, and still see both versions of its changes in `jj log`. They are marked
`(divergent)` because each change ID now names two visible commit IDs.

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
$ jj log -r 'heads(divergent())' --no-graph \
    -T 'change_id.short() ++ " " ++ commit_id.short() ++ "\n"'
vxmynnlnrlzt 0c71df124215
vxmynnlnrlzt 11ff48792102
```

Here `0c71df124215` is on the line retained by `new-chapters`, while
`11ff48792102` is the tip of the old line. The repeated change ID tells us that
these are divergent versions of the same logical change. `jj log -r
'divergent()'` shows every affected version, and `jj log -r 'heads(all())'`
helps us spot unbookmarked heads.

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
