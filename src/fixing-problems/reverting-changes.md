# Throwing work away on purpose

Everything so far has been about getting work back. Sometimes we decide that
some work should stay out of the project instead. `jj` has a few different
tools for that, depending on exactly what we want to remove.

## Discarding edits in the working copy

Let's say we've been working on something, it hasn't worked out, and we want the
files back the way they were. That's what `jj restore` does:

```console
$ jj st
Working copy changes:
M f.txt
Working copy  (@) : wtylzslq 4528b111 next
Parent commit (@-): usvzmupx c39e9f3f add a better feature
$ jj restore
Working copy  (@) now at: wtylzslq 4f4e99e0 (empty) next
Parent commit (@-)      : usvzmupx c39e9f3f add a better feature
Added 0 files, modified 1 files, removed 0 files
```

Our change is empty again. This is the rough equivalent of `git restore` or the
old `git checkout -- .`, and by default it restores everything in `@` from its
parent. Name paths if you only want some of them back, or use `--from` to pull
the contents from somewhere other than the parent.

Nothing is really lost here either, of course — the pre-restore state is an
operation, so `jj undo` brings your edits back.

## Getting rid of a whole change

We've been using `jj abandon` throughout the book for this. It makes the change
go away and rebases anything that was sitting on top of it:

```console
$ jj abandon mlzwmxzs
Abandoned 1 commits:
  mlzwmxzs 9a4ad229 (empty) (no description set)
```

Use it for work you never want to see again — a scratch change, a dead end, an
empty commit you accidentally created.

## Undoing a change that's already shared

`jj abandon` rewrites history: the change simply stops existing. That's fine
while the work is ours, but it can cause problems after other people have
pulled it.
Pushing alone doesn't necessarily make a commit immutable: commits published
through tracked feature bookmarks remain mutable by default. Don't rely on the
immutability guard to decide whether a rewrite is safe to share.

Instead, we want a *new* commit that undoes the old one. We can make one with
`jj revert`:

```console
$ jj revert -r usvzmupx -o @
Reverted 1 commits as follows:
  uykkwoyr 1ce4fe35 Revert "add a better feature"
```

```console
$ jj log --limit 3
○  uykkwoyr steve@steveklabnik.com 2024-03-20 10:04:14 1ce4fe35
│  Revert "add a better feature"
@  wtylzslq steve@steveklabnik.com 2024-03-20 10:04:14 4f4e99e0
│  (empty) next
○  usvzmupx steve@steveklabnik.com 2024-03-20 10:02:11 c39e9f3f
│  add a better feature
```

`add a better feature` is still there, untouched, and there's now a commit after
it that applies its diff backwards. History is intact and the effect is gone,
which is what you want on a shared branch. `-r` says what to revert and `-o`
says where to put the result, the same way `-o` works for `jj rebase`.

## Which one do I want?

* If the files in `@` are wrong, use `jj restore`.
* If a whole change is unwanted and still local, use `jj abandon`.
* If a whole change is unwanted and already pushed, use `jj revert`.

If we pick the wrong one, `jj undo` gives us a way back.
