# The operation log

We now know that every repository change `jj` makes is recorded as an
operation. A single command can create more than one operation, while a
read-only command creates none. Let's look at that record with `jj op log`:

```console
$ jj op log
@  4079800aed31 steve@steves-laptop default@ now, lasted 12 milliseconds
│  snapshot working copy
│  args: jj op log
○  1775539db729 steve@steves-laptop default@ now, lasted 12 milliseconds
│  new empty commit
│  args: jj new -m 'add goodbye'
○  ecbf2e1106cf steve@steves-laptop default@ now, lasted 11 milliseconds
│  snapshot working copy
│  args: jj new -m 'add goodbye'
○  cac23ddcd31f steve@steves-laptop default@ now, lasted 12 milliseconds
│  new empty commit
│  args: jj new -m 'add a comment'
○  58e635d3bf7d steve@steves-laptop default@ now, lasted 10 milliseconds
│  describe commit 0439edc8c87e
│  args: jj describe -m 'hello world'
○  eab1b981cdcd steve@steves-laptop default@ now, lasted 11 milliseconds
│  snapshot working copy
│  args: jj describe -m 'hello world'
○  194587de219c steve@steves-laptop now, lasted 18 milliseconds
│  add workspace 'default'
○  000000000000 root()
```

This looks a bit like `jj log`, and that's on purpose, but don't confuse the
two. `jj log` shows you commits. `jj op log` shows you *states of the whole
repository*. Each of those IDs on the left names one complete snapshot of
everything: every commit, every bookmark, where `@` was pointing, the lot.

A few things to notice.

Each entry records the command that caused it, on the `args:` line. That's often
enough on its own to work out where things went wrong.

There are more entries than commands you ran. `jj new -m 'add goodbye'` produced
two: a "snapshot working copy" and then a "new empty commit". Remember from way
back in the `jj st` chapter that `jj` snapshots your working copy before doing
anything else. That snapshot is an operation like any other, which is why you
can undo your way back to a state you never explicitly committed.

Right at the bottom is `add workspace 'default'` — that's `jj git init` — and
below that, an operation root, in the same spirit as the commit root.

The name and hostname come from your OS account, not from the `user.name` and
`user.email` you set for commits. They're separate settings, `operation.username`
and `operation.hostname`, so anonymizing your commit identity doesn't anonymize
this. This is worth knowing if you care about that sort of thing.

## Going back to an old state

Say we've had a bad afternoon:

```console
$ jj log
@  toumkvst steve@steveklabnik.com 2024-03-20 09:14:40 bc36132e
│  (empty) even more mess
○  ozylkotm steve@steveklabnik.com 2024-03-20 09:14:40 215023b0
│  a description I regret
○  zzzzmznr steve@steveklabnik.com 2024-03-20 09:14:20 3e44726c
│  hello world
◆  zzzzzzzz root() 00000000
```

We abandoned a change we wanted, wrote a description we didn't, and piled
another commit on top. We *could* `jj undo` three times. But we know roughly
where the good state was, so let's just go there. Find it in the log:

```console
$ jj op log --limit 6
@  7fde600ecf01 steve@steves-laptop default@ now, lasted 9 milliseconds
│  new empty commit
│  args: jj new -m 'even more mess'
○  1e8a46d3d6ab steve@steves-laptop default@ now, lasted 8 milliseconds
│  describe commit 08a853a9e2b7
│  args: jj describe -m 'a description I regret'
○  a3ac895c94f3 steve@steves-laptop default@ now, lasted 11 milliseconds
│  abandon commit 0570dd0df085
│  args: jj abandon rxwzpxqt
○  8112955b56b7 steve@steves-laptop default@ now, lasted 7 milliseconds
│  undo: restore to operation 4079800aed31
│  args: jj undo
○  897f241ce8c8 steve@steves-laptop default@ now, lasted 10 milliseconds
│  abandon commit 0570dd0df085
│  args: jj abandon rxwzpxqt
○  cac23ddcd31f steve@steves-laptop default@ now, lasted 12 milliseconds
│  new empty commit
│  args: jj new -m 'add a comment'
```

`cac23ddcd31f` is the last state before it all went wrong. `jj op restore` takes
us there:

```console
$ jj op restore cac23ddcd31f
Restored to operation: cac23ddcd31f (2024-03-20 09:14:20) new empty commit
Working copy  (@) now at: rxwzpxqt 1e939d6c (empty) add a comment
Parent commit (@-)      : zzzzmznr 3e44726c hello world
Added 0 files, modified 1 files, removed 0 files
```

It's worth being clear about what happened here: `jj` restored the whole
repository to that state. Anything that happened afterwards is no longer
visible, including work we may have wanted to keep. `jj undo` steps back one
operation at a time, while `jj op restore` goes straight to the state we name.
Both are useful, but I pause for a moment before using `jj op restore` to make
sure I've picked the right operation.

And because `jj op restore` is itself an operation, it goes in the log too. If
we restore to the wrong place, `jj undo` takes us back to the state before the
restore. Let's look at one of those operations more closely.

## Looking at a single operation

`jj op show` tells you what one operation actually did. Let's look at that
abandon from earlier, the one that started all this:

```console
$ jj op show a3ac895c94f3
a3ac895c94f3 steve@steves-laptop default@ now, lasted 12 milliseconds
abandon commit 0570dd0df0853eecc772025ef84a6b913243b1c7
args: jj abandon rxwzpxqt

Changed commits:
○  + ozylkotm 9921e522 add goodbye
   - ozylkotm/1 afc85298 (hidden) add goodbye
○  - rxwzpxqt/0 0570dd0d (hidden) add a comment

Changed working copy default@:
+ ozylkotm 9921e522 add goodbye
- ozylkotm/1 afc85298 (hidden) add goodbye
```

Minus signs are commits that stopped being visible, plus signs are ones that
started. You can read the whole mistake off this: `add a comment` went away, and
`add goodbye` was replaced by a new version of itself — that's the automatic
rebase onto the new parent.

Note that word: *hidden*, not deleted. Those commits are still in the
repository, which is exactly why undo works. Add `-p` if you want the file
diffs as well.

There's also `jj op diff`, which compares two operations rather than showing
one, when you want to know what changed between two points in the afternoon.

## Does this grow forever?

You may be wondering: if every command is recorded, and hidden commits are
kept around, doesn't the repository just grow without bound? Yes, it does —
nothing prunes any of this automatically. In practice it takes a long time to
matter, since operations and hidden commits are small. If a busy repository
does get bulky, `jj op abandon ..<old-op-id>` marks old operations as
disposable, and `jj util gc` then reclaims whatever nothing refers to any
more. The price is exactly what you'd expect: you can no longer undo your way
back past the operations you abandoned. I've never needed to do this, but it's
good to know the ratchet has a release.

Operations cover the repository. Sometimes the question is narrower than that:
not "what did I do to this repo?" but "what happened to *this one change*?"
That's the next chapter.
