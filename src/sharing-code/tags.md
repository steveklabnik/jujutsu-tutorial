# Tagging a release with `jj tag`

We've seen that bookmarks move. That's the point of them: `trunk` follows our
main line of development, and a pull request bookmark follows the tip of the
pull request. Sometimes we want the opposite, though: a name for one exact
commit. That's a tag.

If you've used `git tag`, this will feel familiar. Let's mark the commit that
`trunk` currently points at:

```console
$ jj tag set v1.0.0 -r trunk
Created 1 tags pointing to ksrmwuon e202b67c trunk | Update Cargo.toml
```

We can see our tags with `jj tag list`:

```console
$ jj tag list
v1.0.0: ksrmwuon e202b67c Update Cargo.toml
```

And tags show up in `jj log` alongside bookmarks:

```console
$ jj log --limit 3
@  msmntwvo steve@steveklabnik.com 2024-03-02 11:47:08 push-vmunwxsksqvk 752534be
│  add a new function
○  vmunwxsk steve@steveklabnik.com 2024-03-02 11:47:08 f6f7dce9
│  add a comment to main
◆  ksrmwuon steve@steveklabnik.com 2024-03-01 23:10:35 trunk v1.0.0 e202b67c
│  Update Cargo.toml
```

## Tags don't move

Here's the difference from a bookmark. Let's try to point `v1.0.0` somewhere
else:

```console
$ jj tag set v1.0.0 -r @
Error: Refusing to move tag: v1.0.0
Hint: Use --allow-move to update existing tags.
```

`jj bookmark set` moves a bookmark without complaint, because that's what
bookmarks are for. A tag is supposed to be a fixed point, so `jj` makes you say
you meant it:

```console
$ jj tag set v1.0.0 -r @ --allow-move
Moved 1 tags to msmntwvo 752534be add a new function
```

Most of the time, you shouldn't need that flag. If you find yourself reaching
for it a lot, you probably wanted a bookmark.

## Tags make history immutable

Remember immutability from the last couple of chapters? Tags are part of the
immutable set, right alongside `trunk()`. So tagging a commit freezes it and
everything behind it:

```console
$ jj tag set v2.0.0 -r @-
Created 1 tags pointing to vmunwxsk f6f7dce9 add a comment to main
$ jj describe @- -m "second thoughts about this one"
Error: Commit f6f7dce9d9a1 is immutable
Hint: Could not modify commit: vmunwxsk f6f7dce9 add a comment to main
Hint: Immutable commits are used to protect shared history.
```

This can be surprising. If we tag a commit in the middle of work we're still
rearranging, `jj` won't let us rearrange it any more. That makes sense: we told
`jj` this commit was a fixed point. If we tagged it by mistake, `jj tag delete`
puts things back:

```console
$ jj tag delete v2.0.0
Deleted 1 tags.
```

And if the commit you tag happens to be the one your working copy is sitting
on, you'll see the same shuffle we saw when we pushed `trunk`:

```console
$ jj tag set v1.1.0 -r @
Created 1 tags pointing to msmntwvo 752534be add a new function
Warning: The working-copy commit became immutable; a new commit has been created on top of it.
Working copy  (@) now at: yqrwvxnn 90f315f0 (empty) (no description set)
Parent commit (@-)      : msmntwvo 752534be add a new function
```

Same reason as before: you can't keep editing a commit you're no longer allowed
to edit, so `jj` moves you up to a fresh one.

## Pushing tags

Tags are local until you push them, and this works exactly like bookmarks did:
a tag the remote has never heard of doesn't get created by a bare `jj git push`.
You name it with `-t`, the way you named a new bookmark with `-b`:

```console
$ jj git push -t v1.0.0
Changes to push to origin:
  tag: v1.0.0 [add to e202b67c1f0a]
```

Once the remote has it, though, a bare push will keep it up to date along with
everything else:

```console
$ jj git push
Changes to push to origin:
  bookmark: trunk [move forward from e202b67c1f0a to 752534beb39f]
  tag: v1.0.0 [move forward from e202b67c1f0a to 752534beb39f]
```

`jj` also distinguishes the local tag from the remote one, the same way it does
for bookmarks. An asterisk means they've drifted apart, and `jj tag list` will
tell you by how much:

```console
$ jj tag list
v1.0.0: msmntwvo 752534be add a new function
  @origin (behind by 2 commits): ksrmwuon e202b67c Update Cargo.toml
```

There is one limitation worth knowing: `jj tag set` creates *lightweight* tags.
`jj` can read the annotated tags that `git tag -a` produces, but it can't create
them. If your project needs an annotated or signed tag for a release, make it
through your hosting service or with `git` directly.

Deleting works like bookmarks too: `jj tag delete` marks the deletion locally.
We can push tracked tag deletions explicitly with `jj git push --deleted`.
