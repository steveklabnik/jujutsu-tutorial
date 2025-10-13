# Using named branches in `jj`

Named branches (or, starting with `jj 0.22`, "bookmarks") are mostly an interoperability feature in `jj`; other than some
sort of "main branch" that indicates where shared history lives, other branches
aren't necessary to get work done. However, if you use a tool like GitHub, which
bases a lot of its functionality around `git` branches, then you'll end up using
more than one named branch.

To create a named branch (bookmark) in `jj`, we can use `jj bookmark create`:

```console
$ jj bookmark create trunk
$ jj log --limit 2
@  povouosx steve@steveklabnik.com 2024-03-01 18:12:43.000 -06:00 trunk f68d1623
│  remove goodbye message
◉  vvmrvwuz steve@steveklabnik.com 2024-03-01 17:49:07.000 -06:00 d41c079b
│  refactor printing
```

I like the name `trunk` here, but you can use `main` if you prefer, whatever you
like really. But if we look on the right hand side of the first log line above,
we can see `trunk` as an identifier here. We can use the name `trunk` as a
revision or use it in a revset if we'd like:

```console
> jj log -r 'ancestors(trunk, 2)'
@  povouosx steve@steveklabnik.com 2024-03-01 18:12:43.000 -06:00 trunk f68d1623
│  remove goodbye message
◉  vvmrvwuz steve@steveklabnik.com 2024-03-01 17:49:07.000 -06:00 d41c079b
│  refactor printing
~
```

One interesting thing about branches in `jj` that's different than branches
in `git` is that branches do not automatically move. For example, let's make a
new change:

```console
> jj new
Working copy now at: pzkrzopz 3f14c03f (empty) (no description set)
Parent commit      : povouosx 7ec11c41 trunk | remove goodbye message
> jj log
@  pzkrzopz steve@steveklabnik.com 2024-03-01 22:41:37.000 -06:00 fcf669c5
│  (empty) (no description set)
│ ◉  qtlkpytx steve@steveklabnik.com 2024-03-01 20:09:25.000 -06:00 e6667f9e
├─╯  (empty) (no description set)
◉  povouosx steve@steveklabnik.com 2024-03-01 18:12:43.000 -06:00 trunk f68d1623
│  remove goodbye message
```

Oh look, we have an extra empty commit lying around. That happens sometimes,
let's forget about it:

```console
> jj abandon qt
Abandoned commit qtlkpytx e6667f9e (empty) (no description set)
> jj log --limit 3
@  pzkrzopz steve@steveklabnik.com 2024-03-01 22:41:37.000 -06:00 trunk fcf669c5
│  (empty) (no description set)
◉  povouosx steve@steveklabnik.com 2024-03-01 18:12:43.000 -06:00 trunk f68d1623
│  remove goodbye message
◉  vvmrvwuz steve@steveklabnik.com 2024-03-01 17:49:07.000 -06:00 d41c079b
│  refactor printing
```

Even though `@` has moved to `pzkrzopz`, `trunk` is still at `povouosx`. This
behavior is a bit surprising for folks coming from `git`, though it fits in with
`jj` more nicely, I think.

Regardless, let's update `trunk` to point at `@`:

```console
$ jj bookmark set trunk
Moved 1 bookmarks to pzkrzopz fcf669c5 trunk | (empty) (no description set)
$ jj log --limit 2
@  pzkrzopz steve@steveklabnik.com 2024-03-01 22:41:37.000 -06:00 trunk fcf669c5
│  (empty) (no description set)
◉  povouosx steve@steveklabnik.com 2024-03-01 18:12:43.000 -06:00 f68d1623
│  remove goodbye message
```

If you want to replicate `git`'s behavior, typing an additional command after
each change is done feels like overkill. But I would argue this is not the right
way to use `jj`; as you'll see, we'll be either re-writing commits at the tip of
branches, or doing multiple steps of work before updating where a branch points.
In practice, it means I check the branch status *before* pushing code, rather
than as I work. That is, the branch name tends to sit at the same change as the
remote server, and when it's time to update the remote, that's when I update
things locally.

Speaking of remotes, let's talk about that next.
