# Copying a change with `jj duplicate`

Every history command we've used so far has *moved* work. `jj rebase` moves a
change to a new parent, `jj squash` moves it into another commit, and
`jj absorb` moves hunks around. Sometimes we want to copy a change instead.
That's what `jj duplicate` does: the original stays where it is, and a second,
independent change appears somewhere else.

This is the equivalent of `git cherry-pick`. A common example is a fix that
landed on our main line of work and is also needed on a release branch:

```console
$ jj log
@  qrmutqws steve@steveklabnik.com 2024-03-27 10:30:47 5e653078
│  fix a bug
│ ○  myvqszlv steve@steveklabnik.com 2024-03-27 10:30:47 c25044fc
├─╯  release prep
○  unrvswsv steve@steveklabnik.com 2024-03-27 10:30:47 ecc089fe
│  initial
◆  zzzzzzzz root() 00000000
```

```console
$ jj duplicate qrmutqws --onto myvqszlv
Duplicated 5e653078ccee as mzrozlul 5b488518 fix a bug
```

```console
$ jj log
@  qrmutqws steve@steveklabnik.com 2024-03-27 10:30:47 5e653078
│  fix a bug
│ ○  mzrozlul steve@steveklabnik.com 2024-03-27 10:30:47 5b488518
│ │  fix a bug
│ ○  myvqszlv steve@steveklabnik.com 2024-03-27 10:30:47 c25044fc
├─╯  release prep
○  unrvswsv steve@steveklabnik.com 2024-03-27 10:30:47 ecc089fe
│  initial
◆  zzzzzzzz root() 00000000
```

Excellent. The fix is now in both places, and our working copy never moved.

## The copy is a different change

Let's look closely at the two IDs. The original is `qrmutqws`; the copy is
`mzrozlul`. Same description, same contents, different change.

This is the important distinction between `duplicate` and `rebase`. A rebased
commit keeps its change ID because it's still the same change in a new place. A
duplicated commit gets a new one because there are now two independent changes.
If we edit one, the other doesn't follow.

It also means the two copies will be pushed as separate commits and reviewed
separately, which is what you want for a backport, and is exactly what a
`git cherry-pick` does.

## Where the copy lands

We used `--onto` to put the copy on a destination. The same `-A` and `-B` flags
we saw with `jj rebase` work here too. They let us insert the copy into the
middle of a stack rather than on top of something:

```console
$ jj duplicate qrmutqws -A myvqszlv
```

With no destination at all, the copy becomes a sibling of the original and
shares its parents. We can use that form when we want to try two versions of the
same idea side by side and keep both.

You can duplicate several changes at once by naming a revset, which is how you
backport a whole feature rather than one commit.

## When not to

If we find ourselves duplicating the same fix onto several release branches
every time, a merge may be a better fit. Duplicating makes sense when the
destination genuinely needs its own version of the change, such as a backport
that will be maintained separately or an experiment we want two versions of.
If we only want to move a change, `jj rebase` leaves us with one copy to keep
track of.
