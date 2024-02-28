## A basic `jj` workflow

New tools often require a change of mental model. This is very true of `jj`.
With `git`, the most basic workflow looks like this:

1. Make some modifications to the contents of the repository.
2. Use `git add` to add these modifications to the "index."
3. Use `git commit` to create a new commit from the index.
4. GOTO 1.

Now you may be saying "Steve, I don't use `git add` I just `git commit -a`!"
I actually think that this is very significant! You're already kind of thinking
like `jj`. I am a big fan of `git`'s index, even though I don't really use `-a`
often. Anyway, a lot of times, people's `git` workflow is:

1. Make some modifications to the contents of the repository.
2. Use `git commit -am <message>` to create a new commit with our modifications.
3. GOTO 1.

We only bust out more complicated tools when we're doing something more
complicated. But this is the core VCS workflow: modifiy, save modifications,
modify, save modifications.

`jj`'s basic workflow works like this too! Except the steps are re-arranged,
and it's better that way.

Let me explain.

### Changes

There is no "dirty state" in `jj`. When we do work, we are doing it within
the context of a "change," and then do things with these changes. For
example, we haven't comitted anything yet, but we're already inside a change!
We can use `jj st` (short for `jj status`) to see the current state of things:

```console
> jj st
Working copy changes:
A .gitignore
A book.toml
A src\SUMMARY.md
A src\gitignore.md
A src\jj-describe.md
A src\setting-up-a-repository.md
Working copy : ovruwlst 4450e3d5 (no description set)
Parent commit: zzzzzzzz 00000000 (empty) (no description set)
```

This kinda looks like `git status` but a bit different. But note one thing:
we haven't run any sort of `git add`-like command, yet all of our changes are
being added (`A`).

Let's talk about those bottom two lines real quick:

```text
Working copy : ovruwlst 4450e3d5 (no description set)
Parent commit: zzzzzzzz 00000000 (empty) (no description set)
```

This output is a little strange since this is a fresh repository, but it's fine.
The first line describes our working copy, that is, what we are currently
working on. We then have some gibberish: `ovruwlst 4450e3d5`. The first one is
a "change id." The second is a "commit id." Why two things? What's the
difference?

Well, coming from `git`, a `jj` "change id" is kind of like a `git`
"commit id" and a `jj` "commit id" is kind of like a `git`... sub-commit id.
Basically, in `jj`'s humble opinion, a commit is too low-level of a concept
to be working with. Commits are how computers keep track of changes. We aren't
computers. Doing mental tedious work is a computer's job. We want to think about
the *logical* change that we are making, not the specific details of how it's
tracked by the version control system.

Okay, but what does that *mean*? First, we have to do some shifting of our
mental model of what a VCS is and what we do with it. Please forgive the
extended conceptual discussion below.

### Why git messed up my brain

Personally, I think this is why a lot of people struggle with `git`, while
others find it easy. I use the term "`git`-brained" for a specific reason: `git`
is born out of a specific ideology, a particular way of structuring how you
think about things. Here is an [interesting quote from
Linus](https://lwn.net/Articles/193245/):

> [...] git actually has a simple design, with stable and reasonably
> well-documented data structures. In fact, I'm a huge proponent of designing
> your code around the data, rather than the other way around, and I think it's
> one of the reasons git has been fairly successful (*).

This repeats a similar sentiment expressed first by Fred Brooks:

> Show me your flowchart and conceal your tables, and I shall continue to be
> mystified. Show me your tables, and I won't usually need your flowchart; it'll
> be obvious.

Which I learned because it is cited by Rob Pike as an inspiration for his fifth
rule of programming:

> Data dominates. If you've chosen the right data structures and organized
> things well, the algorithms will almost always be self-evident. Data
> structures, not algorithms, are central to programming.

Do I think this sentiment is wrong? Of course not! I'm a big fan, actually!
But note these quotes are about understanding how a system works, not about
how to design a usable version control system.

I think `git` is fantastic because of exactly what Linus says: it's a nice data
structure. When I use `git`, I visualize the data structure in my head. I see
what it looks like, where I want it to go, and then map that to a `git` command,
and then do that. See, isn't that easy? Lol. I feel like I sound... "oh yeah
look how easy this is just visualize a data structure, diff it, no big deal.
Why do people find `git` hard?"

Look, I know I'm spending a lot of time on this. But this has been my struggle
to learn other forms of VCS. A ton of people I know really like "stacked diffs,"
and I have struggled to understand them, not because the words make no sense,
but because my brain wants to understand how it's implemented, to figure out
how it changes the data structure. And once I did, I was overwhelmed. But
that's a Me issue.

So to get my brain in the right headspace to understand `jj`, I think it's
crucial to get the mental model right from the start. And it's different in
`jj` than `git`.

## So what is a change anyway?

A change is a particular *logical* change we are recording in our repository.
Like commits in `git`, changes have a parent change, and they form a tree.
(okay, okay, a DAG. Nerd.) So why aren't they commits? And why is there *also*
a commit id?

`git`-brain time: okay, how would you track changes if there was no index? How
does `jj` do this? Well, what if every time you ran a `jj` command it did
a `git commit -a` for you? That way, there's never any dirty state. Sure, but
wouldn't that make a super messy history? Sure, so why not add an abstraction
here to hide it? That's what a change is. If you squint, it almost serves
like a branch, and commits are just autosave in the backgrouund. That doesn't
mean it's literally a `git branch`, I may have strained this analogy too far.

Think about it this way: a repository is kind of like a folder. Not a computer
folder, like a physical folder with real pieces of paper in it. Am I too old?
Is this what getting old feels like?

Changes are kind of like documents, pieces of paper placed in the folder.
They have an order. Turning the page is kind of like following a child pointer
but instead we know they're parent pointers so really it's like reading the book
backwards. Dammit there goes the limits of this analogy again. Back on track: I
can edit a piece of paper, but that won't change that the second page is the
second page. Commits are kind of like the various revisions of each piece.

And when examining the papers stored in a folder, you don't want to read
every revision of every page. That's too much. You just want to read the pages.
Sure, in some contexts, you'd want to dig that deep, but it's details. Not
super important.

I got up and walked around my room several times while writing those last
paragraphs, overthinking all of this way too much, and had a bit of a
realization about this: this is already how I use git, just more manual. I am
a big fan of `git rebase`. I never understood why people didn't like it. Why?
Because I could just commit whatever and whenever, and then go back and sculpt
my work into beautiful commits with good messages. You know, terminal sessions
like this:

```console
> <do work>
> git add .
> git status
> git commit -m "starting this thing"
> <do work>
> git commit -am "thing's almost working"
> cargo fmt
> git commit -am "oops forgot to run cargo format"
> <do more work>
> git commit -am "one more failing test left"
> <do more work>
> git commit -am "hell yeah it works let's fucking goooooo"
> git rebase -i HEAD~6
> git log -n1
commit ac30cdef63ddb5f8b67577d48db1a1aedbcabc27 (HEAD -> outline-register)
Author: Steve Klabnik <steve@steveklabnik.com>
Date:   Sat Feb 3 08:17:03 2024 -0600

    Manually outline ApiDescription::register

    This function often gets called in generic contexts, and so a *lot* of
    monomorphization happens. By making this small change, you can help the
    compiler not do as much work.

    In my tests, this results in five seconds being knocked off of Omicron's
    build times.
> git push origin outline-register
```

You know what I'm doing here? I'm manually auto-saving. I know I'm just gonna
rebase this into something nice later, so why bother even giving it a real commit
message. That's going away. You may notice earlier I said I don't really use
`-a`, but that's basically all I use here? Yeah, it's because I am a hypocrite.
I understand that I am working on a logical revision. And when I am manipulating
those things, I will often use the index, more fine-grained tools... but when
I'm actually working on a thing, it's "commit 'em all and let squash sort 'em
out."

So yeah, anyway. Changes. They're the logical thing you're working on, and
commits are just whatever happens to get saved while you're working on stuff.
The state at the latest commit represents the current state of the change.

So now we understand most of 

```text
Working copy : ovruwlst 4450e3d5 (no description set)
Parent commit: zzzzzzzz 00000000 (empty) (no description set)
```

but not all. The first thing is easy: the `zzzzzzzz` change and the `00000000`
commit are that way because this is a brand new repository: there is no
parent commit. They're empty. We'll see if this changes once we create our
second change.

But the part we haven't talked about yet: descriptions?

### Descriptions

Descriptions are kind of like a commit message. But again, we're not thinking
in commits, we're thinking in changes. Higher level. A change has an id,
so we can refer to it easily with tools. But it's nice for humans to have some
sort of description of what the change is. So we can add one to our change
if we'd like:

```console
> jj describe
```

This will bring up an editor. I'm on Windows, so that's Notepad. I should
change that. Anyway, just like `git commit`, we can edit this file, giving
our change a description.

```text
Hello, world!

This is the beginning of this book existing!

JJ: This commit contains the following changes:
JJ:     A .gitignore
JJ:     A book.toml
JJ:     A src\SUMMARY.md
JJ:     A src\gitignore.md
JJ:     A src\jj-describe.md
JJ:     A src\setting-up-a-repository.md

JJ: Lines starting with "JJ: " (like this one) will be removed.
```

Save and quit to finish the process.

Now we can look at `jj st` again:

```text
> jj st
Working copy changes:
A .gitignore
A book.toml
A src\SUMMARY.md
A src\gitignore.md
A src\jj-describe.md
A src\setting-up-a-repository.md
Working copy : ovruwlst 4450e3d5 Hello, world!
Parent commit: zzzzzzzz 00000000 (empty) (no description set)
```

We can see the first line of our description in the log output now.
Decoupling "when you save changes" and "when you describe changes" is very cool.
I will often be examining my beautiful sculpted commit, and realize I have
missed something, so then it's time for `git commit --amend`. With `jj` I would
just make the change. Done. Easy.

Okay, so... what next? It's time to make our second change.

# jj new

How does `jj` know when we are done? If it just saves things every time we make
changes... how do we start working on a different change? Making a new
one is very easy:

```console
> jj new
Working copy now at: tmlnvvtr 89b6b019 (empty) (no description set)
Parent commit      : ovruwlst fc8c588d Hello, world!
```

There we go! It's that easy. Now we can make more changes, and they'll get
saved as commits in this change. We also didn't give our change a
description yet. We can do that with `jj describe` like we just did with
our last change, but if we know what description we want when we
create the change, `jj new` accepts a `-m <MESSAGE>` parameter that will let
you set a description from the command line.

After setting the description, we'll see it like we would in `jj st`:

```console
> jj describe
Working copy now at: tmlnvvtr 0ff64c6d jj-new chapter
Parent commit      : ovruwlst fc8c588d Hello, world!
```

So we can see that the workflow has the same pieces as the git workflow, but
in a different order:

| tired                                | wired                                                                 |
|--------------------------------------|-----------------------------------------------------------------------|
| making changes, then committing them | creating a new change, then making changes                            |
| commits require a message            | describing a change with some human-readable text if you feel like it |

Now that we can create changes, how do we share them with others? What
do we do about pushing to GitHub?
