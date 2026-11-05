# Using `jj` with Gerrit

Everything in this section so far has assumed GitHub, or something shaped like
it: branches, pull requests, one review per branch. Gerrit is the other major
shape of code review, and it turns out `jj` fits it remarkably well — arguably
better than `git` does.

In Gerrit, the unit of review isn't a branch, it's a single commit, called a
"change" (or a "CL"). You upload a commit, a reviewer comments on it, you
rewrite the commit and upload it again, and Gerrit shows each uploaded version
as a numbered "patch set" on the same change. If that sounds familiar, it
should: it's exactly how we've been treating `jj` changes all along. One `jj`
change becomes one Gerrit change, and rewriting it — with `jj squash`,
`jj absorb`, `jj describe`, whatever we like — produces its next patch set.

## Change IDs, twice over

Gerrit needs a way to recognize that a rewritten commit is a new version of an
existing change, and its answer predates `jj`: a `Change-Id:` trailer in the
commit message. That's the same problem `jj`'s change IDs solve, but they are
two different identifiers. When we upload, `jj` adds the trailer for us,
deriving it from the `jj` change ID, and leaves any existing trailer alone. We
don't need to manage it, but don't be surprised to see it in our descriptions
afterwards.

## Uploading

In a traditional Gerrit workflow, you push with a magic refspec:
`git push origin HEAD:refs/for/main`. `jj` wraps this up in a single command:

```console
$ jj gerrit upload -r 'trunk()..@' --remote-branch main
```

The `-r` revset selects what to upload — here, our whole stack — and
`--remote-branch` names the branch the changes are intended to land on. Each
revision in the revset becomes its own Gerrit change, so uploading a stack of
three commits opens three changes, each reviewable on its own, with their
relationships intact. This is the "stacked" workflow that takes real effort on
GitHub, and on Gerrit it's just how things work.

Because uploading may *rewrite* our commits — adding missing `Change-Id`
trailers — as well as push them, I like to dry-run a broad revset before
running it for real:

```console
$ jj gerrit upload -r 'trunk()..@' --remote-branch main --dry-run
```

Responding to review is the part that should feel comfortable by now: edit the
change like any other — `jj edit`, or fix things in `@` and `jj squash` or
`jj absorb` them down — and upload again. The stable trailer tells Gerrit it's
the same change, and the reviewer sees a new patch set, with Gerrit's UI able
to diff one patch set against another. There's no branch to force-push and no
bookmark to move.

## A default branch

Typing `--remote-branch main` every time gets old. We can set a default for
the repository:

```console
$ jj config set --repo gerrit.default-remote-branch main
```

After that, `jj gerrit upload -r 'trunk()..@'` is all we need. There's a
matching `gerrit.default-remote` setting for pointing at the Gerrit instance
itself, if the remote isn't where we usually push.
