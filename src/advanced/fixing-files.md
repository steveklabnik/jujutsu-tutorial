# Running formatters with `jj fix`

In `git`, we may reach for a pre-commit hook to run a formatter. `jj` doesn't
run `git` hooks, but it does have a different approach: `jj fix` can run a
formatter over a whole stack of commits.

We configure the command to run and the files it applies to:

```toml
[fix.tools.rustfmt]
command = ["rustfmt", "--emit", "stdout"]
patterns = ["glob:**/*.rs"]
```

The contract is simple: our tool reads a file on stdin, writes the fixed version
to stdout, and `jj` puts the result back. Any formatter that can do that works,
and we can configure as many as we like. Each `[fix.tools.<name>]` section gets
its own `patterns`.

Then:

```console
$ jj fix
Fixed 2 commits of 4 checked.
Working copy  (@) now at: kvrxwpqm 1a7d451e (empty) add docs
Parent commit (@-)      : plzxxqzu 55ac427d (no description set)
```

`jj` examined four commits and found two that actually needed changing. It
rewrote both in place and rebased everything downstream, just like the other
history-editing commands we've used.

## What gets fixed

With no arguments, `jj fix` covers `reachable(@, mutable())`: our working copy
and every mutable commit connected to it. Remember, "mutable" means `jj` permits
us to rewrite the commit; it doesn't mean the commit is unpushed. In particular,
commits on a tracked remote bookmark stay mutable by default so that pull
request stacks can be rewritten. Bare `jj fix` can therefore touch work that's
already under review, but it won't touch immutable commits such as trunk.

This is what makes it different from a hook. A formatter run as a
pre-commit hook leaves every earlier commit in the stack unformatted, so a
stack of five commits gets five rounds of "fix formatting" noise, or one
formatting commit at the end that touches files the other four introduced.
`jj fix` reformats all five as if you'd had the formatter on the whole time.

We can use `-s` to narrow or widen that set:

```console
$ jj fix -s @
$ jj fix -s 'mutable()'
```

Naming a revision fixes that revision *and its descendants*, because changing a
file in the middle of a stack means everything above needs rebasing anyway. And
the immutable rule still applies — asking for `all()` gets you a refusal rather
than a rewrite of trunk:

```console
$ jj fix -s 'all()'
Error: The root commit 000000000000 is immutable
```

That's a purely local repository talking; in a repository with a pushed trunk,
the refusal names the trunk commit instead. Either way, nothing shared gets
rewritten.

If you'd rather it defaulted to something else, `revsets.fix` in your config
sets the default scope.

## When to run it

`jj fix` rewrites commits, so every commit it touches gets a new commit ID. For
work we haven't pushed, that doesn't matter. On a branch someone is reviewing,
it means the next push will rewrite the remote commits, and the review tool may
show the formatting changes again. I prefer to run it before pushing when I
can.

It may run your formatter once for every distinct version of a file in the
stack, which on a long stack is not free. Identical file contents are
deduplicated, but formatting history still costs more than formatting only the
tip. That's the price of the history coming out clean.

## The other use

Formatting is the obvious case, but we can use the same mechanism for any tool
that reads and writes a file this way. A `sed` script or a purpose-built codemod
works too. `jj fix` lets us apply it consistently across a stack rather than
only at the tip.
