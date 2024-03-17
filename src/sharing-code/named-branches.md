# Using named branches in `jj`

Named branches are mostly an interoperability feature in `jj`; other than some
sort of "main branch" that indicates where shared history lives, other branches
aren't neccesary to get work done. However, if you use a tool like GitHub, which
bases a lot of its functionality around `git` branches, then you'll end up using
more than one named branch.

To create a named branch in `jj`, we can use `jj branch create`:

```console
$ jj branch create trunk
$ jj log --limit 2
@  povouosx steve@steveklabnik.com 2024-03-01 18:12:43.000 -06:00 trunk f68d1623
│  remove goodbye message
◉  vvmrvwuz steve@steveklabnik.com 2024-03-01 17:49:07.000 -06:00 d41c079b
│  refactor printing
```

I like the name `trunk` here, but you can use `main` if you prefer, whatever you
like really. But if we look on the right hand side of the first log line above,
we can see `trunk` as an identifier here. We can use the name `main` as a
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
Working copy now at: moxnkoxx 3f14c03f (empty) (no description set)
Parent commit      : ytkvxlpy 7ec11c41 trunk | remove goodbye message
> jj log
@  moxnkoxx steve@steveklabnik.com 2024-03-17 17:10:57.000 -05:00 3f14c03f
│  (empty) (no description set)
│ ◉  qtlkpytx steve@steveklabnik.com 2024-03-17 17:09:25.000 -05:00 e6667f9e
├─╯  (empty) (no description set)
◉  ytkvxlpy steve@steveklabnik.com 2024-03-17 17:09:25.000 -05:00 trunk 7ec11c41
│  remove goodbye message
```

Oh look, we have an extra empty commit lying around. That happens sometimes,
let's forget about it:

```console
> jj abandon qt
Abandoned commit qtlkpytx e6667f9e (empty) (no description set)
> jj log --limit 3
@  moxnkoxx steve@steveklabnik.com 2024-03-17 17:10:57.000 -05:00 3f14c03f
│  (empty) (no description set)
◉  ytkvxlpy steve@steveklabnik.com 2024-03-17 17:09:25.000 -05:00 trunk 7ec11c41
│  remove goodbye message
◉  krmulszn steve@steveklabnik.com 2024-03-17 17:06:59.000 -05:00 e98c1626
│  refactor printing
```

Even though `@` has moved to `moxnkoxx`, `trunk` is still at `ytkvxlpy`. This
behavior is a bit surprising for folks coming from `git`, though it fits in with
`jj` more nicely, I think.

Regardless, let's update `trunk` to point at `@`:

```console
$ jj branch set trunk
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
