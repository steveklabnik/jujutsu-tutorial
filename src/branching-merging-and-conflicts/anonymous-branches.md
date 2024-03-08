# What anonymous branches are, and how to use them

When I first heard of "anonymous branches," I got very confused. Git models
branches as a pointer to a commit, and that pointer needs a name, so that's a
branch.

Turns out, you don't really need to name your branches, and doing so is also
not really worth it. I have heard that, inside of Meta, where they use a similar
VCS tool that also has anonymous branches, almost nobody bothers to name their
branches once they get used to things.

Let's talk about it.

## What is a branch, conceptually?

When two changes share the same parent change, we say that they are "branching,"
because the graph of commits would look like this:

```text
                                 
                     ┌───┐ ┌───┐ 
                 ┌───┤ F ◄─┤ G │ 
                 │   └───┘ └───┘ 
                 │               
 ┌───┐  ┌───┐  ┌─▼─┐ ┌───┐ ┌───┐ 
 │ A ◄──┤ B ◄──┤ C ◄─┤ D ◄─┤ E │ 
 └───┘  └───┘  └───┘ └───┘ └───┘ 
                                 
```

Here, we'd say that `F` and `G` are two changes that are "on a branch," because
it looks like they're branching off from `D` and `E`. In reality, in `git`,
both would be "on a branch," because everything is on some sort of branch in
`git`. If you're not on a branch, `git` will say something like this:

> You are in 'detached HEAD' state. You can look around, make experimental
> changes and commit them, and you can discard any commits you make in this state
> without impacting any branches by switching back to a branch.

Git considers any commit that's not part of a branch to be garbage, and so
will garbage collect those commits at some point. Git is very branch-centric.

`jj` does not think about the world this way. Consider the diagram above: we
didn't name any of these branches, yet the diagram still made sense. There's not
really an inherent need to name our branches, just like there isn't an inherent
need to describe our changes. That said, it would be nice to know how to refer
to different branches, even if they're not named.

Let's see how this works.

## Creating two branches from the same commit

Run `jj st`, and make sure your working copy is at an empty change. If not,
use `jj new` to create one. We want to start from a blank slate here.

```console
$ jj new
Working copy now at: yykpmnuq 0bc7a425 (empty) (no description set)
Parent commit      : ootnlvpt b5db7940 only print hello world
```

A common reason for branching is to work on two different ideas at once. Let's
start two different features: one to add some more documentation to our project,
and another to split our print function into hello and goodbye functions.

First, we'll work on the documentation, so let's describe our current commit
to be working on that:

```console
> jj describe -m "add better documentation"
Working copy now at: yykpmnuq 4a95c1f9 (empty) add better documentation
Parent commit      : ootnlvpt b5db7940 only print hello world
```

Next, we want to make a change to work on our hello and goodbye functions.
We want this change to build on top of `ootnlvpt`, so we can just say that:

```console
$ jj new o
Working copy now at: xrslwzvq e9249c85 (empty) (no description set)
Parent commit      : ootnlvpt b5db7940 only print hello world
```

We want to create our new change with the parent `o`, which we could see is the
short name for `ootnlvpt`. Your change ID may be different if you're not
following me exactly, so you may want to double check you've got the right
change!

Let's describe this one too:

```console
$ jj describe -m "create hello and goodbye functions"
Working copy now at: xrslwzvq a70d464c (empty) create hello and goodbye functions
Parent commit      : ootnlvpt b5db7940 only print hello world
```

Excellent. We've got two different changes, `yykpmnuq` and `xrslwzvq`, both with
the parent `ootnlvpt`. Sucess! We have created a branch. And we didn't need to
name it.

We can see that there's a branch in the output of `jj log`:

```console
$ jj log
@  xrslwzvq steve@steveklabnik.com 2024-02-29 23:06:23.000 -06:00 a70d464c
│  (empty) create hello and goodbye functions
│ ◉  yykpmnuq steve@steveklabnik.com 2024-02-29 23:03:22.000 -06:00 210283e8
├─╯  (empty) add better documentation
◉  ootnlvpt steve@steveklabnik.com 2024-02-28 23:26:44.000 -06:00 b5db7940
│  only print hello world
◉  nmptruqn steve@steveklabnik.com 2024-02-28 23:09:11.000 -06:00 90a2e97f
│  refactor printing
◉  ywnkulko steve@steveklabnik.com 2024-02-28 22:09:40.000 -06:00 ed71bb54
│  print goodbye as well as hello
◉  puomrwxl steve@steveklabnik.com 2024-02-28 20:38:13.000 -06:00 7a096b8a
│  it's important to comment our code
◉  yyrsmnoo steve@steveklabnik.com 2024-02-28 20:24:56.000 -06:00 ac691d85
│  hello world
◉  zzzzzzzz root() 00000000
```

We've got our two changes, and there's a fork in the road.

So if these branches don't have names, how do we tell them apart? Well, the
descriptions of their commits are right there. We can look at them and easily
tell which of the two we care about, and then use their change IDs to
distinguish between the two. Coming up with an extra name isn't inherently
helpful here. Of course, *sometimes* it can be, and that's why eventually we'll
talk about named branches. But the important thing to realize here is that you
only have to name branches where adding a name adds some sort of value.


## Getting a list of branches

Okay, but what if we had tons of branches? How would we be able to see them?
In this view, with only two, it makes sense, but what if we had way more?

We can actually ask `jj log` to show us the head of every anonymous branch.
We do it like this:

```console
> jj log -r 'heads(all())'
@  xrslwzvq steve@steveklabnik.com 2024-02-29 23:06:23.000 -06:00 a70d464c
│  (empty) create hello and goodbye functions
~

◉  yykpmnuq steve@steveklabnik.com 2024-02-29 23:03:22.000 -06:00 210283e8
│  (empty) add better documentation
~
```

This shows both of our heads. But what is that `heads(all())` stuff?
It's called a "revset," and it's what we're going to talk about next.
