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
add on our "creating a new changeset" commit yet. So how do we do that? Well,
we have two options, actually. Let's put on our `git` brain for a second.
Updating a branch is how we update a pull request on GitHub, for example.
People generally want PRs updated in one of two ways, and they feel VERY strongly
that their way is correct and we should burn those other heathens at the stake:

* Only add commits to a branch. Don't rewrite *any* pushed history.
* Feel free to rebase your branch. In fact, some go even futher, and want only
  one commit per pull request. "Squash and merge," as they say. You never rewrite
  the `main` branch, or any other long-living, shared with others branches. But
  the branch should be mergable at all times, so fold those changes in please.
