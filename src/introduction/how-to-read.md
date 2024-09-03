# How to read this tutorial

This tutorial is split up into a few different sections. While I make sure that
each concept is introduced before its used, if you're okay with looking some
stuff up on your own, you can skip around if you'd like. 

This tutorial is *intended* to be read from front to back. You can even follow
along with the examples if you'd like. But I'm not your dad, you can do whatever
you'd like.

Here's what each section is about:

## Hello, world!

In this section, we show off the very core concepts of `jj`: repositories,
changes, looking at history, stuff like that. This stuff is simple but also
very important! Everything else is built on top of this.

If you want to skip around, I would at least consider reading this part first.
Okay, just skim. Point is, if you don't already know this stuff, you'll have a
harder time understanding later sections.

## Real-World Workflows

The workflow we developed in "Hello, world!" works, but isn't as pleasant as it
should be. There are two basic workflows that are used by the majority of `jj`
enthusiasts, and so we go over both of them in detail.

## Branching, merging, and conflicts

Once you've got a core workflow down, the next concept involves branching, and
how to merge branches. Once you do that enough, you'll also have to learn how
to resolve merge conflicts.

`jj` natively is a "branchless" VCS, which sounds especially weird coming from
`git`. If you've ever wondered what that's like, this part will show you how,
and if you're wondering how you interact with `git`'s named branches, well,
you're getting ahead of me! That's next!

## Sharing your code with others

Because you'll be sharing your code with `git` users, you'll need to understand
how to use named branches in `jj`. We'll cover that, and then also share how you
can upload your code to `GitHub` (or any other remote repository, really) and
even respond to pull request feedback that upstream may give you.

## More advanced workflows

Now that we have a very solid understanding of `jj`, we can start to do very
cool things. Up until now, we've been fitting `jj` into `git` shaped workflows,
to make it easier to understand. This section will show you the power of `jj`'s
various primitive operations by some practical workflows you may decide to
incorporate into your usage of `jj`.

Additionally, we'll cover some advanced features of the tool: workspaces and
colocated repositories. You may not use these, but it's good to know about them.

## Fixing problems

At some point, you'll probably make some sort of mistake. Maybe you combine two
commits you didn't intend to combine. Maybe you tried to fix a merge conflict,
but you don't like where you ended up.

If you've ever used `git reflog`, you know exactly what this chapter is about.
But if you don't, well, you know how some people say "you can't ever lose your
work in `git`" even though you absolutely can? This section will show you why
it's even more difficult to do so in `jj`.

## Customizing your experience

Finally, `jj` is very customizable. We'll talk about how you can customize
various parts of `jj`, including its very powerful templating system for showing
exactly the output you want from any command.
