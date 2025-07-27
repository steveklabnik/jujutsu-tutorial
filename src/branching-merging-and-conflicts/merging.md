# Merging anonymous branches

Let's recall where we are:

```console
> jj log
@  xrslwzvq steve@steveklabnik.com 2024-02-29 23:06:23.000 -06:00 a70d464c
│  (empty) create hello and goodbye functions
│ ◉  yykpmnuq steve@steveklabnik.com 2024-02-29 23:03:22.000 -06:00 210283e8
├─╯  (empty) add better documentation
◉  ootnlvpt steve@steveklabnik.com 2024-02-28 23:26:44.000 -06:00 b5db7940
│  only print hello world
```

We have what we consider to be the head of our repository, `ootnlvpt`, and then
two branches, `xrslwzvq` and `yykpmnuq`. Let's create another change with
`ootnlvpt` as the parent, to simulate the idea that some changes have landed
on our main branch while we were doing the work:

```console
> jj new o -m "added some cool new feature"
Working copy now at: pzoqtwuv 9353442b (empty) added some cool new feature
Parent commit      : ootnlvpt b5db7940 only print hello world
```

Let's take a look:

```console
> jj log --limit 5
@  pzoqtwuv steve@steveklabnik.com 2024-03-01 15:06:59.000 -06:00 9353442b
│  (empty) added some cool new feature
│ ◉  xrslwzvq steve@steveklabnik.com 2024-02-29 23:06:23.000 -06:00 a70d464c
├─╯  (empty) create hello and goodbye functions
│ ◉  yykpmnuq steve@steveklabnik.com 2024-02-29 23:03:22.000 -06:00 210283e8
├─╯  (empty) add better documentation
◉  ootnlvpt steve@steveklabnik.com 2024-02-28 23:26:44.000 -06:00 b5db7940
│  only print hello world
◉  nmptruqn steve@steveklabnik.com 2024-02-28 23:09:11.000 -06:00 90a2e97f
│  refactor printing
```

We passed the `--limit` flag so that we didn't see every commit; our history
is already getting a little long.

## Merging branches

Now, you may expect that you'd use a command like `jj merge` to merge branches
together. However, as of `0.14.0`, `jj merge` is deprecated, and will be removed
some time later this year. So how the heck do we create merges?

Well, what is a merge anyway? It's a new change that has more than one parent.
How do we make new changes? With `jj new`. So let's ask it to make a change that
has both `pzoqtwuv` and `yykpmnuq` as parents:

```console
> jj new pzoqtwuv yykpmnuq -m "merge better documentation"
Working copy now at: rxzyvnkx f1c1bde8 (empty) merge better documentation
Parent commit      : pzoqtwuv 9353442b (empty) added some cool new feature
Parent commit      : yykpmnuq 210283e8 (empty) add better documentation
```

Just like we'd pass a parent revision to `jj new`, we can pass multiple parents,
and it just works. No need for a special command. Let's look at our history,
choosing six as the limit since we just added a new change:

```console
> jj log --limit 6
@    rxzyvnkx steve@steveklabnik.com 2024-03-01 15:21:11.000 -06:00 f1c1bde8
├─╮  (empty) merge better documentation
│ ◉  yykpmnuq steve@steveklabnik.com 2024-02-29 23:03:22.000 -06:00 210283e8
│ │  (empty) add better documentation
◉ │  pzoqtwuv steve@steveklabnik.com 2024-03-01 15:06:59.000 -06:00 9353442b
├─╯  (empty) added some cool new feature
│ ◉  xrslwzvq steve@steveklabnik.com 2024-02-29 23:06:23.000 -06:00 a70d464c
├─╯  (empty) create hello and goodbye functions
◉  ootnlvpt steve@steveklabnik.com 2024-02-28 23:26:44.000 -06:00 b5db7940
│  only print hello world
◉  nmptruqn steve@steveklabnik.com 2024-02-28 23:09:11.000 -06:00 90a2e97f
│  refactor printing
```

We can see the lines connecting to both of our parents, and we can still see
the `xrslwzvq` branch is left over too.

But here's something really wild: we can just do this as much as we want, no
need to stop at two parents. To try this out, we're going to run a command
I haven't told you about yet:

```console
$ jj undo
Working copy now at: pzoqtwuv 9353442b (empty) added some cool new feature
Parent commit      : ootnlvpt b5db7940 only print hello world
$ jj log --limit 5
@  pzoqtwuv steve@steveklabnik.com 2024-03-01 15:06:59.000 -06:00 9353442b
│  (empty) added some cool new feature
│ ◉  xrslwzvq steve@steveklabnik.com 2024-02-29 23:06:23.000 -06:00 a70d464c
├─╯  (empty) create hello and goodbye functions
│ ◉  yykpmnuq steve@steveklabnik.com 2024-02-29 23:03:22.000 -06:00 210283e8
├─╯  (empty) add better documentation
◉  ootnlvpt steve@steveklabnik.com 2024-02-28 23:26:44.000 -06:00 b5db7940
│  only print hello world
◉  nmptruqn steve@steveklabnik.com 2024-02-28 23:09:11.000 -06:00 90a2e97f
│  refactor printing
```

That's right, we can undo our last command with a simple `jj undo`. We'll talk
about it more in the future. But for now, it's like our merge never happened.

Let's try merging all three in at the same time:

```console
$ jj new pzoqtwuv yykpmnuq xrslwzvq -m "merge three branches"
Working copy now at: pzoqtwuv 9353442b (empty) added some cool new feature
Parent commit      : yykpmnuq 210283e8 (empty) add better documentation
Parent commit      : xrslwzvq a70d464c (empty) create hello and goodbye functions
$ jj log --limit 6
@      vuztuxmz steve@steveklabnik.com 2024-03-01 15:38:49.000 -06:00 717232df
├─┬─╮  (empty) merge three branches
│ │ ◉  xrslwzvq steve@steveklabnik.com 2024-02-29 23:06:23.000 -06:00 a70d464c
│ │ │  (empty) create hello and goodbye functions
│ ◉ │  yykpmnuq steve@steveklabnik.com 2024-02-29 23:03:22.000 -06:00 210283e8
│ ├─╯  (empty) add better documentation
◉ │  pzoqtwuv steve@steveklabnik.com 2024-03-01 15:06:59.000 -06:00 9353442b
├─╯  (empty) added some cool new feature
◉  ootnlvpt steve@steveklabnik.com 2024-02-28 23:26:44.000 -06:00 b5db7940
│  only print hello world
◉  nmptruqn steve@steveklabnik.com 2024-02-28 23:09:11.000 -06:00 90a2e97f
│  refactor printing
```

Just as easy as that: a merge commit with three different parents.

Once again I am reminded of the theme I discussed at the start: simpler can also
be more powerful. `jj` has eliminated the need for an entire command but not
lost any functionality.

But what if we didn't want to create a merge commit? Don't worry, `jj` has
rebase as well.

## Rebasing branches

Like `git`, `jj` has a command called `rebase`. It does what it says, it takes
a change and, instead of its current "base," aka parent, moves it to have a
different parent, "basing" it again, or "re-basing" it.

Let's undo our merge again:

```console
$ jj undo
Working copy now at: pzoqtwuv 9353442b (empty) added some cool new feature
Parent commit      : ootnlvpt b5db7940 only print hello world
$ jj log --limit 5
@  pzoqtwuv steve@steveklabnik.com 2024-03-01 15:06:59.000 -06:00 9353442b
│  (empty) added some cool new feature
│ ◉  xrslwzvq steve@steveklabnik.com 2024-02-29 23:06:23.000 -06:00 a70d464c
├─╯  (empty) create hello and goodbye functions
│ ◉  yykpmnuq steve@steveklabnik.com 2024-02-29 23:03:22.000 -06:00 210283e8
├─╯  (empty) add better documentation
◉  ootnlvpt steve@steveklabnik.com 2024-02-28 23:26:44.000 -06:00 b5db7940
│  only print hello world
◉  nmptruqn steve@steveklabnik.com 2024-02-28 23:09:11.000 -06:00 90a2e97f
│  refactor printing
```

Excellent. Let's keep a linear history by using a rebase instead of a merge.

You can use `jj rebase` in a few different ways. Let's show off the simplest:
rebasing a single change. Let's rebase our "create hello and goodbye functions"
change on top of our current change:

```console
$ jj rebase -r xrslwzvq -d pzoqtwuv
```

This rebases a single revision with `-r`, to a certain destination revision,
hence `-d`. Since our branch only had one revision, this would be the same as
passing `-b xrslwzvq`, which would move the whole branch that revision is on,
or `-s xrslwzvq`, which rebases that revision as well as all of its descendants.

We didn't get any output though. Let's look at our log:

```console
$ jj log --limit 5
◉  xrslwzvq steve@steveklabnik.com 2024-03-01 16:08:37.000 -06:00 6c4afc8f
│  (empty) create hello and goodbye functions
@  pzoqtwuv steve@steveklabnik.com 2024-03-01 15:06:59.000 -06:00 9353442b
│  (empty) added some cool new feature
│ ◉  yykpmnuq steve@steveklabnik.com 2024-02-29 23:03:22.000 -06:00 210283e8
├─╯  (empty) add better documentation
◉  ootnlvpt steve@steveklabnik.com 2024-02-28 23:26:44.000 -06:00 b5db7940
│  only print hello world
◉  nmptruqn steve@steveklabnik.com 2024-02-28 23:09:11.000 -06:00 90a2e97f
│  refactor printing
```

We have rebased our commit successfully. But you may have noticed something
surprising: `@` is still at `pzoqtwuv`. This actually belies a very deep
difference between `jj` and `git` that I learned from [Austin
Seipp][thoughtpolice], one of `jj`'s maintainers. And here it is:

`jj` commands primarily operate on the data structures stored in its repository,
rather than on the working copy.

This simple statement has some profound implications. One of the simplest
consequences of this is `jj`'s speed. Because the working copy is itself a
commit, and commits are in the database, it can treat it like any other commit.
In this case, the difference is even larger: `git rebase` works on the working
copy. This is why it has to stop you and make you resolve things in the case
where a conflict happens, because it's about to create a new commit from the
working copy, and if that's in conflict, it has to be fixed or the next commit
is nonsense. We'll talk about how `jj` handles conflicts shortly, but as I
said before: rebases *always* succeed in `jj`. So this change is quick: it's
only modifying information in the repository, not touching any of the files we
have in our working directory. This also means our working directory hasn't
changed, so `@` is in the same place it was before the rebase.

Some commands do move `@` by default, like `jj new`. This is because if you're
creating a new change, you probably want to start working on it. But it has a
flag you can pass instead to create a new change but not modify `@`:

```console
$ jj new -m "not gonna start this yet" --no-edit
Created new commit owlpoptm df6620cb (empty) not gonna start this yet
$ jj log --limit 6
◉  owlpoptm steve@steveklabnik.com 2024-03-01 16:28:54.000 -06:00 df6620cb
│  (empty) not gonna start this yet
│ ◉  xrslwzvq steve@steveklabnik.com 2024-03-01 16:08:37.000 -06:00 6c4afc8f
├─╯  (empty) create hello and goodbye functions
@  pzoqtwuv steve@steveklabnik.com 2024-03-01 15:06:59.000 -06:00 9353442b
│  (empty) added some cool new feature
│ ◉  yykpmnuq steve@steveklabnik.com 2024-02-29 23:03:22.000 -06:00 210283e8
├─╯  (empty) add better documentation
◉  ootnlvpt steve@steveklabnik.com 2024-02-28 23:26:44.000 -06:00 b5db7940
│  only print hello world
◉  nmptruqn steve@steveklabnik.com 2024-02-28 23:09:11.000 -06:00 90a2e97f
│  refactor printing
```

New change, yet we're still where we are. Let's undo that real quick:

```console
$ jj undo
$ jj log --limit 5
◉  xrslwzvq steve@steveklabnik.com 2024-03-01 16:08:37.000 -06:00 6c4afc8f
│  (empty) create hello and goodbye functions
@  pzoqtwuv steve@steveklabnik.com 2024-03-01 15:06:59.000 -06:00 9353442b
│  (empty) added some cool new feature
│ ◉  yykpmnuq steve@steveklabnik.com 2024-02-29 23:03:22.000 -06:00 210283e8
├─╯  (empty) add better documentation
◉  ootnlvpt steve@steveklabnik.com 2024-02-28 23:26:44.000 -06:00 b5db7940
│  only print hello world
◉  nmptruqn steve@steveklabnik.com 2024-02-28 23:09:11.000 -06:00 90a2e97f
│  refactor printing
```

Cool. Okay so that theory sounds cool, and it's nice that it makes things fast,
but what about when we *do* want to move `@`? Well, the fact that the
`--no-edit` flag is what we passed to `jj new` gave it away:

```console
$ jj edit xrslwzvq
Working copy now at: xrslwzvq 6c4afc8f (empty) create hello and goodbye functions
Parent commit      : pzoqtwuv 9353442b (empty) added some cool new feature
$ jj log --limit 5
@  xrslwzvq steve@steveklabnik.com 2024-03-01 16:08:37.000 -06:00 6c4afc8f
│  (empty) create hello and goodbye functions
◉  pzoqtwuv steve@steveklabnik.com 2024-03-01 15:06:59.000 -06:00 9353442b
│  (empty) added some cool new feature
│ ◉  yykpmnuq steve@steveklabnik.com 2024-02-29 23:03:22.000 -06:00 210283e8
├─╯  (empty) add better documentation
◉  ootnlvpt steve@steveklabnik.com 2024-02-28 23:26:44.000 -06:00 b5db7940
│  only print hello world
◉  nmptruqn steve@steveklabnik.com 2024-02-28 23:09:11.000 -06:00 90a2e97f
│  refactor printing
```

We can now rebase our other change on top too:

```console
$ jj rebase -r yykpmnuq -d xrslwzvq
$ jj log --limit 5
◉  yykpmnuq steve@steveklabnik.com 2024-03-01 16:35:47.000 -06:00 7bea29b6
│  (empty) add better documentation
@  xrslwzvq steve@steveklabnik.com 2024-03-01 16:08:37.000 -06:00 6c4afc8f
│  (empty) create hello and goodbye functions
◉  pzoqtwuv steve@steveklabnik.com 2024-03-01 15:06:59.000 -06:00 9353442b
│  (empty) added some cool new feature
◉  ootnlvpt steve@steveklabnik.com 2024-02-28 23:26:44.000 -06:00 b5db7940
│  only print hello world
◉  nmptruqn steve@steveklabnik.com 2024-02-28 23:09:11.000 -06:00 90a2e97f
│  refactor printing
```

Excellent. But before we move `@`, I want to show you a little trick. We could
type in the change ID, and in this case, `yyk` is the unique prefix, so it
isn't that hard. But we can also use a revset:

```console
$ jj edit @+
Working copy now at: yykpmnuq 7bea29b6 (empty) add better documentation
Parent commit      : xrslwzvq 6c4afc8f (empty) create hello and goodbye functions
```

`+` means "the child of this revision", so `@+` is "the child revision of the
working copy", which in this case is exactly where we wanted to go.

We've alluded to conflicts several times in this tutorial. We're finally ready
to address those.

[thoughtpolice]: https://github.com/thoughtpolice
