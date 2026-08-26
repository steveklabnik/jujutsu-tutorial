# Resolving conflicts with `jj resolve`

In the last chapter, we resolved a conflict by opening the file and editing the
markers by hand. That always works, and for a small conflict it's often the
quickest approach. We can also use `jj resolve`, which hands each conflicted
file to a merge tool one at a time.

Since we cleaned up the conflict in our project already, I've made a little
scratch repository to play with: two changes, "say hi" and "say hey", that
each put a different greeting in `greeting.txt`, and a merge of the two on
top. That merge is, of course, conflicted.

First, let's find out what's actually conflicted:

```console
$ jj resolve --list
greeting.txt    2-sided conflict
```

Two-sided is the ordinary case — two branches, two versions. You'll see
three-sided and worse if you merge more than two things at once, or conflict on
top of a conflict.

Then run it:

```console
$ jj resolve
```

That opens your merge editor on `greeting.txt`, with the two sides and the base
version. Save and exit, and `jj` records the result; exit without changing
anything and it stops, leaving the conflict alone. If several files are
conflicted, you get them one after another, and you can name paths —
`jj resolve greeting.txt` — to deal with one at a time.

Which tool opens depends on `ui.merge-editor`, which we'll set in the
customization section. The default is `jj`'s built-in editor.

## Taking one side wholesale

Sometimes there's nothing to merge because one side is simply right. There are
two built-in tools that let us skip the editor:

```console
$ jj resolve --tool :ours
Working copy  (@) now at: plwvlnst 80d01ccc merge the two
Parent commit (@-)      : tmqzyrmt ef8a1082 say hi
Parent commit (@-)      : xvqtknsp 828bdd03 say hey
```

```console
$ cat greeting.txt
hi
```

`:ours` takes the first side of the conflict, `:theirs` the second. Be careful
with the names: in a merge, "ours" is the first parent you gave `jj new`, and
in a rebase it's the commit being rebased onto. Check with `jj diff` afterwards
rather than trusting the word.

## Resolving somewhere else

Conflicts don't have to be resolved in the working copy. `-r` points `jj resolve`
at any conflicted commit. Back in the last chapter, when `povouosx` was sitting
above us in conflict, we could have fixed it without moving `@` at all:

```console
$ jj resolve -r povouosx
```

This is worth knowing because `jj`'s hint suggests another approach: make a new
commit on top of the conflicted one, fix it there, then use `jj squash` to move
the fix down. Both work. The hint's version keeps the conflicted commit
untouched while we experiment, while `-r` edits it directly.

## Afterwards

Resolving is just editing, so the usual tools apply. `jj st` stops warning you:

```console
$ jj st
Working copy changes:
M greeting.txt
Working copy  (@) : plwvlnst 80d01ccc merge the two
Parent commit (@-): tmqzyrmt ef8a1082 say hi
Parent commit (@-): xvqtknsp 828bdd03 say hey
```

`jj diff` shows what you settled on, and `jj undo` puts the conflict back if you
got it wrong. A resolution is a normal edit to a normal commit, with all the
same escape hatches.

And as we saw last chapter, resolving a conflict once is enough: descendants get
rebased onto the resolved version automatically, and the conflict doesn't come
back for each commit in turn the way it does with `git rebase`.

That's branching, merging, and conflicts covered. Before we can get to the
stacked pull request workflow those behaviors make possible, we have to talk
about using `jj` with GitHub in the first place. Let's go over that next.
