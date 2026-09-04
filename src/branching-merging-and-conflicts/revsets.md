# Figuring out where our changes are with revsets

We have learned about two kinds of identifiers in `jj`: change IDs and commit
IDs. But what if we want to talk about, for example, a range of commits?

`jj` has a concept called a "revset," short for "revision set." Sometimes
people say "revision" instead of "commit," and "revset" is just nicer to say
than "comset", so it stuck.

This sounds pretty intense at first, but I promise it's simpler than you
think: `jj` supports a functional language to describe revsets. Almost
every command in `jj` takes a `-r`/`--revision` flag, which is the revision
to operate on. This defaults to `@`. This means when we do `jj new`, we're
basically doing `jj new -r @`, that is, create a new change with a parent
revision of the current working copy.

## Symbols

`@` is actually our first example of the revset language. This is called a
"symbol". Symbols are a means of specifying a single commit. `@` refers to
the change containing the current working copy, but a change ID or commit ID
are other examples of symbols.

## Operators

Operators let you describe more complex relationships between changes. For
example, remember how in the squash workflow, we would move the contents of
the working copy into the parent change? Well, the `-` operator refers to the
parent of a given revision, and `@` is the change referring to the current
working copy, so we might say "we squashed the contents of `@` into `@-`. And
in fact, `jj squash` is short for `jj squash -r @` or equivalently `jj squash
--from @ --into @-`. There are many operators, including, but not limited to:

* `x & y`: changes that are in both x and y
* `x | y`: changes that are in either x or y
* `::x` Ancestors of x
* `x::` Descendants of x
* `x-`: direct parents of x
* `x+`: direct children of x
* `x::y`: descendants of x that are also ancestors of y, including both x and y
* `x+::y`: paths from direct children of x to y, excluding x and including y
* `x..y`: ancestors of y that are not ancestors of x, including y but not x

And more. The final bit is the most interesting, and that's functions.

## Functions

Functions allow for even more complex selection of a series of changes. The
simplest functions are:

* `root()`: a function that returns the root change
* `all()`: this function returns all visible changes
* `mine()`: this function returns all changes authored by the current user

More complex functions can take arguments:

* `parents(x)`: the parent changes of `x`
* `ancestors(x)`: the same as `::x`, but see the next example
* `ancestors(x, depth)`: limits the results to a certain depth, which you can't do with the `::x` syntax
* `heads(x)`: commits in `x` that are not ancestors of other commits in `x`
* `description(substring:x)`: commits that have a substring `x` in their description

That `substring:` prefix on the last one matters, by the way. If you leave it
off and write `description("print")`, `jj` looks for a description that is
*exactly* the string `print` — which, since descriptions keep their trailing
newline, matches nothing at all. A bare string quietly gives you an empty set,
and whatever command you ran does nothing. Use `substring:` or `glob:` and
you'll get what you meant.

## Putting it all together

Now we can understand `heads(all())` from before: these are two functions, where
we're asking for the head commits of every commit in the repository.

Revsets are very powerful, and very convenient. Would you like to find every
commit by me containing the word "print" in the description? Try this:

```console
$ jj log -r 'author("Steve Klabnik") & description(substring:print)'
```

Another really useful revset function is `trunk()`:

```console
$ jj log -r 'trunk()'
◆  zzzzzzzz root() 00000000
```

Right now, this doesn't look very useful, but it will be more useful when we
get into sharing our changes. `trunk()` resolves to the tip of the shared main
line of development — in most repositories that's `main@origin`, the `main`
branch on the `origin` remote. More precisely, it looks for a remote named
`origin` or `upstream`, looks for a `main`, `master`, or `trunk` branch on it,
and provides that. Since we don't have any of those right now, it gives us the
same as `root()`.

Additionally, on the `jj` Discord, several folks have settled on this as a
decent revset for larger repositories:

```console
$ jj log -r '@ | ancestors(remote_bookmarks().., 2) | trunk()'
```

This will show the history from the working copy, some detail about remote
branches, as well as the trunk. What's good varies between what you're trying
to do and what your repository looks like, so experiment with some of this
stuff to find something that works well for you.

## Selecting a whole anonymous branch

Let's put those operators to work on a real problem. In the last chapter, we
made two anonymous branches from the same parent. Sooner or later, we'll make
a branch we decide we don't want: suppose we tried one idea for three changes,
started another idea from the same point, and then realized that the first
idea was a dead end. Our history would look like this:

```text
common ─┬─ n1 ── n2 ── n3
        ╰─ x1 ── x2 (@)
```

Here, `common`, `n1`, and the other labels stand for change IDs. The `n` and
`x` changes form two anonymous branches, but of course, there's no branch
marker to delete — that's the whole point of anonymous branches. Even if we
had put a bookmark on `n3` and deleted it, `n3` would still be a visible head.
If we want that whole first idea gone from our visible history, we need to
abandon all three of its changes, and revsets let us name all three at once.

Let's start by asking `jj` to show us what `n3` adds on top of `common`:

```console
$ jj log -r 'common..n3'
○  n3
○  n2
○  n1
~
```

The `..` operator excludes `common`: `common..n3` means "ancestors of `n3` that
are not ancestors of `common`." In this simple graph, that gives us exactly the
three changes on the branch we want to abandon. It also leaves `x1` and `x2`
alone, because neither is an ancestor of `n3`.

You may remember another range operator that looks similar. `common::n3` means
"changes on a path from `common` to `n3`," including *both* ends. It selects
`common`, `n1`, `n2`, and `n3` — so we must not abandon that range, because
the other branch still needs `common`. This is the sort of off-by-one that's
worth previewing with `jj log` before running anything destructive.

If we know the first change on the unwanted branch, `n1::n3` gives us the
exact inclusive range from `n1` through `n3`. Let's preview it, then abandon
it:

```console
$ jj log -r 'n1::n3'
○  n3
○  n2
○  n1
~
$ jj abandon 'n1::n3'
Abandoned 3 commits:
  n3
  n2
  n1
```

What if we know `common` but not `n1`? Then `common+::n3` selects the same
range: the `+` means "direct children of `common`," so this starts just after
`common`, and the `::n3` part keeps only changes on a path to `n3`. The two
spellings can differ if the branch merges in history that isn't descended from
`common` — `common..n3` would include that history, while `common+::n3`
excludes it — but for a simple branch like ours, they're the same three
changes.

While our two branches stay separate, all of these ranges leave `x1` alone,
because it isn't an ancestor of `n3`. If we had merged `x1` into `n3`, though,
both `common..n3` and `common+::n3` would pick it up, since it would then be an
ancestor of `n3`. `n1::n3` would still exclude it, because `x1` is not a
descendant of `n1`. So when a branch has merges in it, `n1::n3` is the precise
range to reach for.

Both boundaries of the range matter. Abandoning only `n3` wouldn't remove the
branch; it would just make `n2` the new anonymous head, and we'd be back where
we started. And if an `n4` existed *after* `n3`, abandoning `n1::n3` wouldn't
touch it — `jj` would rebase the survivor onto `common` and report:

```console
Rebased 1 descendant commits onto parents of abandoned commits.
```

So to discard an entire branch, we want its actual first change and its actual
tip as the boundaries. In our graph, `n3` really is the tip, so abandoning the
range leaves us with:

```text
common ── x1 ── x2 (@)
```

One reassuring note to close on: this hides the commits, it doesn't erase
them. `jj` records the abandon in the
[operation log](../fixing-problems/the-operation-log.md), whose earlier states
keep the hidden commits around. That's why `jj undo` can bring the branch back
if abandoning it was a mistake. Only once the old operations referring to
those commits are themselves abandoned can garbage collection remove them for
good.

Revsets are very powerful, and you'll learn some useful ones as you explore
more. At some point, we'll even talk about how to create custom aliases for
revsets, but for now, let's get back to dealing with branches and how to merge
them.
