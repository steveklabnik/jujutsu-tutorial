# Branching, merging, and conflicts

You may have noticed that we haven't talked about branches at all yet. This is
another significant difference between `jj` and `git`: `jj` prefers to use
anonymous branches, rather than named ones. People sometimes call this a
"branchless" workflow. In this chapter, we'll learn about how to do branches
in the way `jj` prefers, and in the next chapter, we'll talk about named
branches.

Here's what we're going to learn:

* What anonymous branches are, and how to use them
* Figuring out where our changes are with revsets
* Customizing the output of various `jj` commands with templates
* Merging anonymous branches
* Dealing with conflicts
