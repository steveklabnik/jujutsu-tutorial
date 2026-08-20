# Stacked pull requests

Back in the conflicts chapter, I said that recording conflicts and rebasing
automatically were the ingredients for a useful workflow. It's time to see what
I meant.

Let's say we're building something that's too big for one pull request. We
could open one very large pull request, or open a small one and wait for it to
merge before starting the next piece. Neither option lets us keep moving while
still giving the reviewer small changes.

The third option is a *stack*: a chain of small pull requests, each based on the
one below, reviewed and merged from the bottom up. This is possible in `git`,
but maintaining the stack takes work. Every time the bottom changes during
review, every branch above it needs to be rebased.

`jj` has been doing those rebases automatically since chapter one. That makes
the workflow much easier to maintain.

## Building the stack

Let's build a feature in three pieces, each on top of the last:

```console
$ jj new trunk -m "add the parser"
$ jj new -m "add the evaluator"
$ jj new -m "wire it up to the CLI"
$ jj log
@  nvpxkvxo steve@steveklabnik.com 2024-03-21 09:45:24 6850afd9
│  wire it up to the CLI
○  mwzzqwkt steve@steveklabnik.com 2024-03-21 09:45:24 716bd51d
│  add the evaluator
○  rtotzlru steve@steveklabnik.com 2024-03-21 09:45:24 904decf4
│  add the parser
◆  mnysvqxx steve@steveklabnik.com 2024-03-21 09:45:23 trunk eecde2ae
│  Update Cargo.toml
```

Nothing new so far — that's just three changes in a row. GitHub needs names,
though, so let's give each one a bookmark:

```console
$ jj bookmark create parser -r rtotzlru
Created 1 bookmarks pointing to rtotzlru 904decf4 parser | add the parser
$ jj bookmark create evaluator -r mwzzqwkt
Created 1 bookmarks pointing to mwzzqwkt 716bd51d evaluator | add the evaluator
$ jj bookmark create cli -r nvpxkvxo
Created 1 bookmarks pointing to nvpxkvxo 6850afd9 cli | wire it up to the CLI
```

`jj git push` takes `-b` more than once:

```console
$ jj git push -b parser -b evaluator -b cli
Changes to push to origin:
  bookmark: cli [add to 6850afd9040b]
  bookmark: evaluator [add to 716bd51df2e4]
  bookmark: parser [add to 904decf432ba]
```

Now open three PRs, each targeting the one below: `parser` into `trunk`,
`evaluator` into `parser`, `cli` into `evaluator`. GitHub will show each PR
containing only its own commit, which is the point — your reviewer sees three
small diffs instead of one big one.

## Updating the stack

Now let's say someone reviews the bottom pull request and asks for a change. In
`git`, we'd fix the parser commit, rebase `evaluator` onto it, and then rebase
`cli` onto that. In `jj`, we can edit the commit directly:

```console
$ jj edit parser
Working copy  (@) now at: rtotzlru 904decf4 parser | add the parser
Parent commit (@-)      : mnysvqxx eecde2ae trunk | Update Cargo.toml
```

Make the fix the reviewer asked for, and look at the log:

```console
$ jj log
Rebased 2 descendant commits onto updated working copy.
○  nvpxkvxo steve@steveklabnik.com 2024-03-21 09:45:41 cli* c3b1a9f6
│  wire it up to the CLI
○  mwzzqwkt steve@steveklabnik.com 2024-03-21 09:45:41 evaluator* 5a6f2c6a
│  add the evaluator
@  rtotzlru steve@steveklabnik.com 2024-03-21 09:45:41 parser* 3b550959
│  add the parser
◆  mnysvqxx steve@steveklabnik.com 2024-03-21 09:45:23 trunk eecde2ae
│  Update Cargo.toml
```

There's a lot in this one screen. `jj` rebased both descendants automatically,
so every commit ID changed. The important bit is that *all three bookmarks are
still attached to their commits*.
They each grew an asterisk, meaning they've drifted from what the remote has,
but none of them got left behind.

That's the payoff for bookmarks following rewrites, which we covered a couple of
chapters ago. Rewriting a commit carries its bookmark along, and an automatic
rebase is a rewrite. So the entire stack re-pointed itself.

One push updates all three pull requests:

```console
$ jj git push --all
Changes to push to origin:
  bookmark: cli [move sideways from 6850afd9040b to c3b1a9f61375]
  bookmark: evaluator [move sideways from 716bd51df2e4 to 5a6f2c6aa13c]
  bookmark: parser [move sideways from 904decf432ba to 3b550959d7d9]
```

That's the maintenance work we would otherwise have done by hand.

## When the bottom merges

Eventually the `parser` PR gets approved and merged. Fetch, and `trunk` has
moved out from under your stack:

```console
$ jj git fetch
bookmark: trunk@origin [updated] tracked
$ jj log
@  nvpxkvxo steve@steveklabnik.com 2024-03-21 09:45:41 cli c3b1a9f6
│  wire it up to the CLI
○  mwzzqwkt steve@steveklabnik.com 2024-03-21 09:45:41 evaluator 5a6f2c6a
│  add the evaluator
│ ◆  yzkopsqy maintainer@example.com 2024-03-21 09:46:07 trunk c5dd335b
╭─┤  (empty) Merge pull request #1 from steve/parser
│ │
│ ~
│
◆  rtotzlru steve@steveklabnik.com 2024-03-21 09:45:41 parser 3b550959
│  add the parser
~
```

Two things changed shape here, and both are worth a look.

`trunk` is a `◆` now, which we'd expect — it's the trunk, it's shared, it's
immutable. But so is `parser`. Your parser commit got merged into `trunk`, which
makes it an ancestor of `trunk`, which makes it immutable too. That work is
shared now, so `jj` has stopped letting us rewrite it.

The other thing: `evaluator` and `cli` are still sitting on the old `parser`
commit, off to the side, while `trunk` has moved on. Move them across:

```console
$ jj rebase -s evaluator -o trunk
Rebased 2 commits to destination.
Working copy  (@) now at: nvpxkvxo a0bc5b05 cli* | wire it up to the CLI
Parent commit (@-)      : mwzzqwkt 263f373c evaluator* | add the evaluator
```

`-s` takes the change and all its descendants, so `evaluator` and `cli` moved
together, and their bookmarks came along as before:

```console
$ jj log
@  nvpxkvxo steve@steveklabnik.com 2024-03-21 09:47:27 cli* a0bc5b05
│  wire it up to the CLI
○  mwzzqwkt steve@steveklabnik.com 2024-03-21 09:47:27 evaluator* 263f373c
│  add the evaluator
◆  yzkopsqy maintainer@example.com 2024-03-21 09:46:07 trunk c5dd335b
│  (empty) Merge pull request #1 from steve/parser
~
```

A clean stack again, one PR shorter. Push it, and retarget the `evaluator` PR at
`trunk` in the GitHub UI, since the branch it used to be based on is about to
disappear:

```console
$ jj git push -b evaluator -b cli
Changes to push to origin:
  bookmark: cli [move sideways from c3b1a9f61375 to a0bc5b0507af]
  bookmark: evaluator [move sideways from 5a6f2c6aa13c to 263f373c557b]
```

Then tidy up the bookmark for the PR that merged. `jj bookmark delete` marks it,
and the push carries the deletion to the remote:

```console
$ jj bookmark delete parser
Deleted 1 bookmarks.
$ jj git push --deleted
Changes to push to origin:
  bookmark: parser [delete from 3b550959d7d9]
```

## Is this worth it?

For a two-part change, I would probably open one pull request. A stack becomes
more useful when we have several dependent pieces and a reviewer who would
rather look at them one at a time.

It's also worth noticing that we didn't need a command specifically for stacked
pull requests. We used `jj new`, `jj edit`, `jj bookmark`, `jj rebase`, and
`jj git push`, just as we did in the earlier chapters. The workflow comes from
combining those familiar commands in a different way.
