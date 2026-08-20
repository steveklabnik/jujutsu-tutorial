# Fixing Problems

At some point, we're going to make a mistake. We may abandon the wrong change,
squash something into the wrong parent, or resolve a conflict and then realize
we preferred the other version.

If you've used `git` for a while, you may have reached for `git reflog`, found a
hash that looked about right, and tried to put things back together. This often
works, but it can feel a bit like emergency recovery.

Recovery works differently in `jj`. Every command we run is recorded, the old
state is kept, and going back is a normal part of the tool rather than a
separate emergency procedure.

Here's what we're going to learn:

* Undoing your last command with `jj undo`
* Reading the history of your repository's *states* with `jj op log`, and
  jumping back to one
* Watching a single change evolve over time with `jj evolog`
* Throwing away changes on purpose, with `jj restore` and `jj revert`
* Hunting down the change that broke something, with `jj bisect`

You've already used `jj undo` a couple of times in this book, on the promise
that I'd explain it later. It's later.
