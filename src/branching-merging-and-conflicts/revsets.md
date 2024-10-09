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
the working directory into the parent change? Well, the `-` operator refers to
the parent of a given revision, and `@` is the change referring to the current
working directory, so we might say "we squashed the contents of `@` into `@-`.
And in fact, `jj squash` is short for `jj squash -r @`. There are many operators,
including, but not limited to:

* `x & y`: changes that are in both x and y
* `x | y`: changes that are in either x or y
* `::x` Ancestors of x
* `x::` Descendants of x

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
* `heads(x)`: commits in `x` that are not ancestors of other commits in `x`.
* `description(x)`: commits that have `x` in their description

## Putting it all together

Now we can understand `heads(all())` from before: these are two functions, where
we're asking for the head commits of every commit in the repository.

Revsets are very powerful, and very convenient. Would you like to find every
commit by me containing the world "print" in the description? Try this:

```console
$ jj log -r 'author("Steve Klabnik") & description(print)'
```

Another really useful revset function is `trunk()`:

```console
$ jj log -r 'trunk()'
◉  zzzzzzzz root() 00000000
```

Right now, this doesn't look very useful, but it will be more useful when we
get into sharing our changes. `trunk()` looks for a remote named `origin` or
`upstream`, and looks for a `main`, `master`, or `trunk` branch, and then
provides that. Since we don't have any of those right now, it gives us the same
as `root()`.

Additionally, on the `jj` Discord, several folks have settled on this as a
decent revset for larger repositories:

```console
$ jj log -r '@ | ancestors(remote_branches().., 2) | trunk()'
```

This will show the history from the working directory, some detail about remote
branches, as well as the trunk. What's good varies between what you're trying to
do and what your repository looks like, so experiment with some of this stuff
to find something that works well for you.

Revsets are very powerful, and you'll learn some useful ones as you explore
more. At some point, we'll even talk about how to create custom aliases for
revsets, but for now, let's get back to dealing with branches and how to merge
them.
