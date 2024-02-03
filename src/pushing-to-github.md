# Pushing to GitHub

Time to do some Social Coding. One nice thing about `jj` is that you can use
it personally, but share your work as a `git` repository. Nobody needs to know.

In order to do this, we need to create a branch. Quoting the official tutorial:

> Branches are named pointers to revisions (just like they are in Git).

Cool. Let's create one. Since this is our first branch, we'll call it `main`,
to keep with `git` conventions. You can of course do whatever you'd like.

```console
> jj branch create main
>
```

Well that was... anticlimactic. Let's take a look at our branches:

```console
> jj branch list
main: quwouuro fd149de1 pushing to github
```

(Yes, I am making more changesets than you neccesarily see here, this book isn't
100% writing itself.)

We can now push our branch to GitHub:

```console
> jj git push
Branch changes to push to origin:
  Add branch main to 5de7f6ed5f3c
```

Woo! It now works. But here's a kind of weird thing. It's kinda scary to me,
coming from Git. Branches are literally pointers to commits. But with
`git commit`, it will update the branch as it goes along. Not so here! I've
been writing some more text since I created the branch, and look:

```console
> jj branch list
main: quwouuro f6b01701 pushing to github
  @origin (ahead by 1 commits, behind by 1 commits): quwouuro hidden 5de7f6ed pushing to github
```

And if I create a new changeset:

```console
> jj new -m "creating a new changeset"
Working copy now at: ppqtnzzt fed80ab2 (empty) creating a new changeset
Parent commit      : quwouuro 44efc909 main* | pushing to github
> jj branch list
main: quwouuro 44efc909 pushing to github
  @origin (ahead by 1 commits, behind by 1 commits): quwouuro hidden 5de7f6ed pushing to github
```

Our `main` branch is still pointed at `quwouuro` and not `ppqtnzzt`. What
happens if we push now?

```console
> jj git push
Branch changes to push to origin:
  Force branch main from 5de7f6ed5f3c to 44efc9093ae1
> jj branch list
main: quwouuro 44efc909 pushing to github
```

It updated the remote branch to the latest commit we had in our "pushing to
github" changeset. But the branch is still pointing to that commit. We didn't
add on our "creating a new changeset" commit yet. So how do we do that?

### Updating a git branch by adding commits

To do this, we want to:

1. create a new changeset: if we push the current state of our current changeset,
  when we make more changes, we'll give the changeset a new revision, which
  means when we push it up, it'll re-write the history, which is what we are
  trying *not* to do. 
2. update the branch to point at our previous, finished changeset
3. push to github

Edit some stuff in your project so that `creating a new changset` has some
commits in it representing some changes. And then:

```console
> jj new
Working copy now at: uvxmoxqx 4e0639c8 (empty) (no description set)
Parent commit      : ppqtnzzt f867ba1d creating a new changeset
```

This is step 1. See how nice it is to not need a description? Who knows what
we'll be working on next, so let's not describe it for now.

```console
> jj branch set main -r ppqtnzzt
```

We give the `-r` flag to `jj branch set` to say which "revision" we want to
set the branch to. "revision" is a synonym for "commit". We gave it a changeset
ID here, and that works too: it'll use the latest commit from that changeset
for you.

But we also could have done this:

```console
> jj branch set main -r @-
```

`@` is a shorthand for the current revision, almost like `HEAD` in `git`. And the
`-` afterwards means "previous," kind of like `^` in `git`. So `@-` is kind of
like `HEAD^`.

Anyway, let's look at our work:

```console
> jj branch list
main: ppqtnzzt f867ba1d creating a new changeset
  @origin (behind by 1 commits): quwouuro 44efc909 pushing to github
```

Okay, let's push it up:

```console
> jj git push
Branch changes to push to origin:
  Move branch main from 44efc9093ae1 to f867ba1d16be
```

Previously, it said "force branch main" and now it says "move branch main." We
have simply added a new commit. 

### Isn't this annoying?

This is a thing that was a red flag to me when I heard it at first. I have to
update my branch? Every time? That feels... bad.

And we'll see how I feel after I use `jj` more. But thinking about it some more...
I appreciate that `jj` doesn't just update things, because that would mean
rewriting history in some cases. But more than that, I also always type:

```console
> git push origin master
```

and not just

```console
> git push
```

I like to know what state I'm pushing, and where. If I forget to update my
branch, `jj git push` will do nothing. That's fine. And when checking if my
changes are ready for review, making sure they're on the right branch isn't
really any different than now?

I also am realizing that this is only a feature of `jj` thanks to `git`, or
at least, it's less clear to me why I'd care about branches in a world that
isn't centered on the way `git` thinks about things. I care a lot about
branches because that's important to `git`, but it's also what GitHub PRs are
designed around. If the unit of code review is a changeset and not a branch,
I don't need to keep a branch auto-updated: a changeset has its stable ID,
no matter how many commits I make within it. No biggie.

Much to consider.
