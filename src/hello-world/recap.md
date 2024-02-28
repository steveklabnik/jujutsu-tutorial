# A recap and some thoughts

So here is our current workflow:

1. Create new repositories with `jj git init`.
2. To start working on a new change, use `jj new`.
3. To describe a change so humans can understand them, use `jj describe`.
4. We can look at our work with `jj st`.
5. When we're done, we can start our next change with `jj new`.

Finally, we can review our repository's contents with `jj log`.

This is... pretty simple! We don't need that many concepts to get started.
Of course, we aren't yet able to do some very important things like "share our
code with others." But we'll get there.

An interesting thing that I've noticed while using `jj` is that sometimes things
feel the same, but backwards. In `git`, we finish a set of changes to our code
by committing, but in `jj` we start new work by creating a change, and *then*
make changes to our code. It's more useful to write an initial description of
your intended changes, and then refine it as you work, than creating a commit
message after the fact.

But also, this stuff is flexible: why *should* you have to create a commit
message at the time of creating a commit, and not whenever you feel like it?
The same stuff exists, but in more flexible pieces that I can combine together.

Before we get into topics like "how to share code with others" and more details
about some of the things we've already learned, let's talk about making our
workflow a little nicer; you should get some more practice using `jj` before we
start collaborating.
