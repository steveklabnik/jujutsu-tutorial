# jj new

How does `jj` know when we are done? If it just saves things every time we make
changes... how do we start working on a different changeset? Making a new
one is very easy:

```console
> jj new
Working copy now at: tmlnvvtr 89b6b019 (empty) (no description set)
Parent commit      : ovruwlst fc8c588d Hello, world!
```

There we go! It's that easy. Now we can make more changes, and they'll get
saved as commits in this changeset. We also didn't give our changeset a
description yet. We can do that with `jj describe` like we just did with
our last changeset, but if we know what description we want when we
create the changeset, `jj new` accepts a `-m <MESSAGE>` parameter that will let
you set a description from the command line.

After setting the description, we'll see it like we would in `jj st`:

```console
> jj describe
Working copy now at: tmlnvvtr 0ff64c6d jj-new chapter
Parent commit      : ovruwlst fc8c588d Hello, world!
```

So we can see that the workflow has the same pieces as the git workflow, but
in a different order:

| tired                                | wired                                                                    |
|--------------------------------------|--------------------------------------------------------------------------|
| making changes, then committing them | creating a new changeset, then making changes                            |
| commits require a message            | describing a changeset with some human-readable text if you feel like it |

Now that we can create changesets, how do we share them with others? What
do we do about pushing to GitHub?
