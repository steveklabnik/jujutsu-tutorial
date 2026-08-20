# Fixing Problems

At some point you're going to make a mess. You'll abandon the wrong change,
squash something into the wrong parent, or resolve a conflict in a way you
regret twenty minutes later.

If you've used `git` for a while, you know the drill: you go and look up
`git reflog`, you find a hash that looks about right, and you hope. It works,
mostly, and it feels like surgery.

`jj` takes this seriously enough that it's not a recovery tool bolted on the
side; it's how the whole thing works. Every command you run is recorded, the
old state is kept, and going back is a normal thing to do rather than an
emergency.

Here's what we're going to learn:

* Undoing your last command with `jj undo`
* Reading the history of your repository's *states* with `jj op log`, and
  jumping back to one
* Watching a single change evolve over time with `jj evolog`
* Throwing away changes on purpose, with `jj restore` and `jj revert`

You've already used `jj undo` a couple of times in this book, on the promise
that I'd explain it later. It's later.
