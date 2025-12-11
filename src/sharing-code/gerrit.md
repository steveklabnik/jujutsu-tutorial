# Working with Gerrit

The Gerrit-based workflow prefers small commits that get reviewed individually,
as "changelists" (CLs). `jj` is well suited for this workflow: each `jj` change
can be a single CL which you can update over time. Each new update to a change
is tracked as a new patchset, this will allow you to see how a
change evolved over time directly in the UI.

## Change IDs

Similar to `jj`'s change IDs, Gerrit associates commits with a common "change"
using a `Change-Id` footer in the commits. When using the `jj gerrit upload`
command, the footer is added automatically to all commits in the revset that
don't already have one.

## Push workflow

In a traditional Gerrit workflow, you push commits to the review server using
`git push origin <rev>:refs/for/main`.

JJ support this workflow natively thanks to the `jj gerrit upload` command.
With this command you can upload revsets to Gerrit directly, if a JJ change is
missing the change-id footer, the command will add it automatically. You can
also modify the footer manually if you wish to associate a specific JJ change
with an already existing Gerrit change.

If you want to triple check which commits will be modified and pushed with this
command you can run `jj gerrit upload -r <revset> --remote-branch <branch-name> --dry-run`

### Remote branch selection

When using `jj gerrit upload` you can either have a default remote branch
selected or overwrite it with each upload.

To configure a default remote branch please run:
`jj config set --user gerrit.default-remote-branch <branch name>`

while if you want to change it for a specific push you can do:
`jj gerrit upload -r <revset> --remote-branch <branch-name>`



