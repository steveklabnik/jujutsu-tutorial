# Running a command across revisions with `jj run`

`jj fix` pipes each file through a formatter. Sometimes we want to run a more
general command against several revisions. `jj run` checks out every revision
we name in an isolated working copy, runs a command there, and, by default,
amends the revision with whatever the command changed. Descendants rebase as
usual, so we can apply one command to a whole stack.

There are two jobs I find useful here, and they want opposite flags.

## Testing a whole stack

First, we can run the tests against every commit in a stack and find out which
one is broken. We don't want a test to rewrite anything, so we'll pass
`--ignore-changes`. This throws away changes made inside the isolated checkout,
which means no revision gets rewritten and immutable commits are fair game too.
It is not a sandbox, though: side effects elsewhere, such as network requests
or writes outside the checkout, still happen.

```console
$ jj run -r 'trunk()..@' --ignore-changes -- cargo test
```

When everything passes, `jj` says so and leaves the stack untouched:

```console
$ jj run -r 'description(glob:"step*")' --ignore-changes -- sh -c 'test $(cat n.txt) -gt 3'
Nothing changed.
```

When something fails, `jj` stops and names the offending revision:

```console
Error: the command 'sh -c test $(cat n.txt) -gt 3' failed with exit status: 1
Hint: Failed revision: nwwnpuuz 7f1e1a07 step one
```

This is useful because a pre-commit hook checks the tip, while
`jj run --ignore-changes` checks every commit in the stack independently. We
can catch a stack that passes at the top but is broken three commits down. It's
the same idea as
[`jj bisect`](../fixing-problems/bisecting.md), run exhaustively instead of by
binary search.

Each command runs with `JJ_CHANGE_ID`, `JJ_COMMIT_ID`, and `JJ_WORKSPACE_ROOT`
in its environment, and `-j` runs several revisions in parallel:

```console
$ jj run -r 'trunk()..@' --ignore-changes -j 4 -- cargo test
```

## Rewriting a whole stack

For the second job, let's leave off `--ignore-changes`. Now the filesystem
changes stick, and each revision is amended with the command's output. This is
how we can apply a codemod across history instead of applying it to the tip and
patching up the earlier commits afterwards.

```console
$ jj run -r 'description(glob:"call foo*")' -- sh -c "find . -name '*.rs' -exec sed -i '' 's/foo/bar/g' {} +"
Rewrote 2 commits.
```

Every commit that called `foo` now calls `bar`, as if you'd made the rename from
the start. `jj fix` covers the formatting case with less ceremony; reach for
`jj run` when the transformation is a real command, not a stdin-to-stdout
filter.

## Commands that are safe to repeat

There's one important rule: our command has to be *idempotent* across the stack,
because `jj` runs it once per revision and then rebases each result onto the
last. A rename is fine; if we apply it twice, the second pass is a no-op. A
blind `echo X >> file` is not. Every commit appends its own `X`, the rebased
contents no longer agree, and we get conflicts instead of a clean rewrite. If
the command's output depends on what came before, we should use a different
approach.

For anything read-only — tests, linters, a build check — pass `--ignore-changes`
and the question doesn't arise.
