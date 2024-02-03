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

Keep typing, and again:

```console

```
