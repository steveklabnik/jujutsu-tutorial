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

## Creating a pull request

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

### Adding commits on to a PR

Let's add another commit. Do some more work, and then run this:

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

### Squashing commits in a PR

Let's make some modifications and then squash them into the previous change:

```console
> <do work>
> jj squash
Working copy now at: voznqwxp 01743a61 (empty) (no description set)
Parent commit      : wuowlzov 17b52582 push-wzqkzmwoxpmn* | continue to work on the 'working with github' chapter
> jj git push
Branch changes to push to origin:
  Force branch push-wzqkzmwoxpmn from 51fb2507a939 to 17b52582c6fc
```

`jj squash` will take the contents of our current change and squash it into
the previous one. This is kind of like amending the previous commit in `git`.
And as we see, that means a force push. But what if we don't want to just squash
new changes in, but squash the whole thing down to one commit?

We can tell it to squash the previous revision:

```console
> jj squash -r @-
```

This will pop up a window with the previous two changes' descriptions, allowing
you to merge them:

```text
JJ: Enter a description for the combined commit.
JJ: Description from the destination commit:
Working with GitHub

JJ: Description from the source commit:
continue to work on the 'working with github' chapter

JJ: Lines starting with "JJ: " (like this one) will be removed.
```

I changed mine to this:

```text
JJ: Enter a description for the combined commit.
JJ: Description from the destination commit:
new chapter: Working with GitHub
```

Yeah I could have deleted the first two lines, I'm lazy. Save and close.

```console
Rebased 1 descendant commits
Working copy now at: voznqwxp 7f7c2786 (empty) (no description set)
Parent commit      : wzqkzmwo 0dd38f9d push-wzqkzmwoxpmn* | new chapter: Working with GitHub
> jj log
@  voznqwxp steve@steveklabnik.com 2024-02-03 10:56:36.000 -06:00 468fe8d3
│  (no description set)
◉  wzqkzmwo steve@steveklabnik.com 2024-02-03 10:56:15.000 -06:00 push-wzqkzmwoxpmn* 0dd38f9d
│  new chapter: Working with GitHub
◉  kkonqyyr steve@steveklabnik.com 2024-02-03 10:10:32.000 -06:00 main b1dbd9ed
│  one little extra note
~
> jj git push
Branch changes to push to origin:
  Force branch push-wzqkzmwoxpmn from 17b52582c6fc to 0dd38f9de670
```

We've squashed it down, and pushed it up. Our PR now only has one commit. Nice.
I am sure there is a way to do this more than one at a time, but for now, this
is good enough for me.

Oh, now that we're done, we may want to not keep that branch around anymore. We
can delete it like this:

```console
> jj branch delete push-wzqkzmwoxpmn
```

Incidentally, by default it helps you keep `origin` clean too:

```console
> jj branch delete --help
Delete an existing branch and propagate the deletion to remotes on the next push
```

Nice.

### Pulling down changes from a merged PR

Our pull request has been merged. This means our `main` branch locally is out
of date from the remote. Let's fetch our changes:

```console
> jj git fetch
```

This will fetch changes from our `origin`. `jj` knows what that is because we
cloned the repository down ourselves. But did it do anything?

```console
~/Documents/GitHub/jujutsu-tutorial> jj log
◉    uyovmwoq steve@steveklabnik.com 2024-02-03 11:00:40.000 -06:00 main 41f8c46c
├─╮  (empty) Merge pull request #2 from steveklabnik/push-wzqkzmwoxpmn
│ │
│ ~
│
│ @  voznqwxp steve@steveklabnik.com 2024-02-03 11:01:25.000 -06:00 85023bec
├─╯  (no description set)
◉  wzqkzmwo steve@steveklabnik.com 2024-02-03 10:56:15.000 -06:00 push-wzqkzmwoxpmn 0dd38f9d
│  new chapter: Working with GitHub
~
```

Oh this is cool! See how this is a bit different than `git log` already? Instead
of `HEAD` being at the top, `@` is in the middle, because it is no longer the
newest commit. We can see that there's a merged PR up there, on `main`. Yep,
even though making new changesets does not update branches, pulling new changes
down from GitHub *will* update our branch. We can check even:

```console
~/Documents/GitHub/jujutsu-tutorial> jj branch list
main: uyovmwoq 41f8c46c (empty) Merge pull request #2 from steveklabnik/push-wzqkzmwoxpmn
```

Nice.

### Updating `@`

So, at this point, you may think you're done. But you're not. And that's because
of something subtle. If you had done the same actions in `git`, what would happen
if you made some changes right now? `HEAD` is still on our pull request branch,
which we have deleted, right? `git` would have warned us about a detatched `HEAD`
by deleting the branch (I hope, I think so, I didn't bother checking.)

We can use `jj checkout` to move `@`:

```console
> jj checkout main
Working copy now at: pqtmqtnu 5c04adb7 (empty) (no description set)
Parent commit      : uyovmwoq 41f8c46c main | (empty) Merge pull request #2 from steveklabnik/push-wzqkzmwoxpmn
Added 0 files, modified 1 files, removed 0 files
```

Now we're at a new change on top of `main`.
