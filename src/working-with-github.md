# Working with GitHub

Okay, we know how to push our code up as `main`, but what about pull requests?

Most projects I am involved in follow a "trunk based development" style
workflow. What this means is:

* `main` is the shared canonical history of the project
* people make branches off of `main` and submit them for review
* when accepted, these branches are merged into `main`

This is a pretty basic and straightforward workflow, and tools like `jj` seem
to prefer it for reasons I don't fully understand the implications of just yet.
But given its basic-ness, I feel like that's a natural next step. So let's
create, update, and then merge a pull request.

Let's modify the contents of our project, describe this change, and then push
that change as a `git` branch:

```console
> <do work>
> jj describe
Working copy now at: wzqkzmwo 2d641227 Working with GitHub
Parent commit      : kkonqyyr b1dbd9ed main | one little extra note
> jj git push -c @
Creating branch push-wzqkzmwoxpmn for revision @
Branch changes to push to origin:
  Add branch push-wzqkzmwoxpmn to 40ba4a3277a9
> jj new
Working copy now at: lqplrmtz 05854f85 (empty) (no description set)
Parent commit      : wzqkzmwo 40ba4a32 push-wzqkzmwoxpmn | Working with GitHub
> jj branch list
main: kkonqyyr b1dbd9ed one little extra note
push-wzqkzmwoxpmn: wzqkzmwo 40ba4a32 Working with GitHub
```

We used the `-c` option to `jj git push` to create our branch. We passed `@` as
the change because we wanted to use the current change as the one we wanted to
use. A cool thing about `-c` is that it will generate a branch name for you,
in this case, we got `push-wzqkzmwoxpmn`. If you wanted to choose a name
for your branch, we could have done what we did earlier and use `jj branch
create <name> -r @` and then `jj git push`. But I think if you're going "`jj`
native" the names of the branches don't really matter. But if you need to
interoperate with some sort of convention in the upstream `git` repo, this
can be useful.

Anyway, as you can now see, we made a new empty changeset so that we didn't
continue updating the changeset we want to push up. It might seem a bit weird
to be typing `jj new` all the time, and it is: we will improve this workflow in
a moment. But for now, we can see that we have our new branch.

And we can make a PR with that remote branch.
[Here](https://github.com/steveklabnik/jujutsu-tutorial/pull/2) is the one
you'll see from me writing this. At the moment, there's just the one commit
there, from our one changeset, but when you look at it, there'll be more.
Let's add another commit, for example.

Do some more work, and then run this:

```console
> jj commit -m "continue to work on the 'working with github' chapter"
Working copy now at: xlnpzmxz a8abb7ef (empty) (no description set)
Parent commit      : wuowlzov 51fb2507 (empty) continue to work on the 'working with github' chapter
```

Wait what? `jj commit`? I thought commits were auto-saves only!

Well... it's kinda annoying to type `jj new` all the time. And so, here's how
`jj commit` describes itself:

> Update the description and create a new change on top

That sounds super wierd coming from `git`, but I hope you're used enough to `jj`
for this to make sense: it's kind of like running `jj describe && jj new`. See:

```console
> jj log
@  xlnpzmxz steve@steveklabnik.com 2024-02-03 10:43:47.000 -06:00 a8abb7ef
│  (empty) (no description set)
◉  wuowlzov steve@steveklabnik.com 2024-02-03 10:43:47.000 -06:00 51fb2507
│  (empty) continue to work on the 'working with github' chapter
◉  wzqkzmwo steve@steveklabnik.com 2024-02-03 10:43:33.000 -06:00 push-wzqkzmwoxpmn* 52caf08e
│  Working with GitHub
◉  kkonqyyr steve@steveklabnik.com 2024-02-03 10:10:32.000 -06:00 main b1dbd9ed
│  one little extra note
~
```

`jj log` is like `git log` but fancier. I am not gonna talk about all the options
here yet because I barely understand them. But we can see that we are now in an
empty change, and that we added a new change on top of our branch. Let's update
the branch and push:

```console
~/Documents/GitHub/jujutsu-tutorial> jj branch set push-wzqkzmwoxpmn -r @-
~/Documents/GitHub/jujutsu-tutorial> jj log
@  xlnpzmxz steve@steveklabnik.com 2024-02-03 10:43:47.000 -06:00 a8abb7ef
│  (empty) (no description set)
◉  wuowlzov steve@steveklabnik.com 2024-02-03 10:43:47.000 -06:00 push-wzqkzmwoxpmn* 51fb2507
│  (empty) continue to work on the 'working with github' chapter
◉  wzqkzmwo steve@steveklabnik.com 2024-02-03 10:43:33.000 -06:00 52caf08e
│  Working with GitHub
◉  kkonqyyr steve@steveklabnik.com 2024-02-03 10:10:32.000 -06:00 main b1dbd9ed
│  one little extra note
~
```

That... is annoying with the generated name. I bet there's a cool trick here,
I just don't know it yet. Maybe not. Anyway.

Time to push:

```console
> jj git push
Branch changes to push to origin:
  Force branch push-wzqkzmwoxpmn from 40ba4a3277a9 to 51fb2507a939
```

Now my PR has two commits. If our reviewer wants us to add more commits,
we can keep doing that with `jj commit` and `jj branch set` and `jj git push`.
But sometimes people want pull requests squashed. We can do that too!

```console
> <do work>

```
