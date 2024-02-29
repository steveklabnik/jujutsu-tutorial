# The Edit Workflow

While I like the previous workflow, some people just don't. They use a workflow
that adds a different command, `jj edit`, along with a second new command,
`jj next`, as well as a new flag to `jj new`. Lots to learn!

The worklow goes like this:

1. We create a new change to work on our feature.
2. If we end up doing exactly what we wanted to do, we're done.
3. If we realize we want to break this up into smaller changes, we do it by
   making a new change before the current one, swapping to it, and making that
   change.
4. We then go back to the main change.

Let's see how to use `jj` this way.

## Create a new change to work on our feature.

Let's create a feature that's un-doing our previous feature: we'll revert to
`Hello, World!` only.

Now, our previous workflow left `@` at an empty change. But if you use this
workflow, `@` will often be on an existing change. So in the real use of this
workflow, we'd start by:

```console
$ jj new -m "only print hello world"
```

But since we have an empty change, what we'll actually do is:

```console
> jj describe -m "only print hello world"
Working copy now at: ootnlvpt bb06f041 (empty) only print hello world
Parent commit      : ywnkulko ed71bb54 print goodbye as well as hello
```

We are now ready to do some work.

Let's change our file to:

```rust
/// A "Hello, world!" program.

fn main() {
    println!("Hello, world!");
}
```

Cool. We're done. In the best case, we're happy with this change, and we're done.
When we begin more work we start it with `jj new -m ""` and get to work.

But sometimes, when we're working on something, we realize we also want a
different change, and maybe it relies on this one. For example, let's say that
we were working on undoing this goodbye feature, but we realized we wanted
to refactor printing out into its own function, because that's a terrible idea
in practice and so makes for a good example to play around with.

What we want to do is make a new change before this one. So let's do that.

## Make a new change and edit it

Let's try this:

```console
$ jj new -B -m "add more comments"
Rebased 1 descendant commits
Working copy now at: nmptruqn 30a1f33b (empty) add more comments
Parent commit      : ywnkulko ed71bb54 print goodbye as well as hello
Added 0 files, modified 1 files, removed 0 files
```

We have a new flag to `jj new`, `-B`. This says to create the new change
*before* the current one. That's exactly what we asked!

The first line of the output should raise some eyebrows:

```text
Rebased 1 descendant commits
```

That's right, because we have created a change before the one we're on, it
automatically rebased our original change. How can it do that? What if there
are conflicts? Relax, we'll get there. All I'll say is something that's probably
hard to believe: this operation will *always* succeed, and we will have our
working copy at the commit we've just inserted. You won't learn how this works
in this chapter, but in a future one.

In the meantime, let's examine our log:

```console
$ jj log
◉  ootnlvpt steve@steveklabnik.com 2024-02-28 22:59:46.000 -06:00 be40656e
│  only print hello world
@  nmptruqn steve@steveklabnik.com 2024-02-28 22:59:46.000 -06:00 30a1f33b
│  (empty) add more comments
◉  ywnkulko steve@steveklabnik.com 2024-02-28 22:09:40.000 -06:00 ed71bb54
│  print goodbye as well as hello
◉  puomrwxl steve@steveklabnik.com 2024-02-28 20:38:13.000 -06:00 7a096b8a
│  it's important to comment our code
◉  yyrsmnoo steve@steveklabnik.com 2024-02-28 20:24:56.000 -06:00 ac691d85
│  hello world
◉  zzzzzzzz root() 00000000
```

We can see that `@` is at our new empty change, and that we have our original
change, `ootnlvpt`, is after us. Some of you may recognize `ootnlvpt`: even
though we rebased it on top of our current change, `nmptruqn`, the change ID
is the same. The commit changed from `bb06f041` to `be40656e`, though. The
change ID is stable, but we can keep track of how the change changes over time.
Neat.

Anyway, now we can edit `@`. Let's change `src/main.rs`. When you first open
up the file, you'll see this:

```rust
/// A "Hello, world!" program.

fn main() {
    println!("Hello, world!");
    println!("Goodbye, world!");
}
```

Remember, this change is before the one where we removed the goodbye message,
so that has returned. Here's what we want to end up with:

```rust
/// A "Hello, world!" program.
/// 
/// This is the best implementation of this program to ever exist.

fn main() {
    println!("Hello, world!");
    println!("Goodbye, world!");
}
```

This is very silly. Regardless, we have finished. Let's see our current status:

```console
$ jj st
Rebased 1 descendant commits onto updated working copy
Working copy changes:
M src\main.rs
Working copy : nmptruqn 90a2e97f add more comments
Parent commit: ywnkulko ed71bb54 print goodbye as well as hello
```

Yet again, a rebase. Because we have changed the contents of our change,
all of the changes that depend on it must be rebased. But again, this happens
all the time, without fail. So it's not something you'll get stuck on at this
stage.

## Return to our main change

Now that we're done, we're going to go back to editing our original commit. To
do that, we could use `jj edit`, which is where this workflow gets its name from.
`jj edit` sets the working copy to the contents of a change, and now changes
you make will update that change.

Doing that would look like this:

```console
$ jj edit o
```

Since `o` is the unique prefix of `ootnlvpt`, our original feature change.
However, looking up that revision is kind of annoying. Therefore, we can use a
simpler command:

```console
$ jj next --edit
Working copy now at: ootnlvpt e13b2585 only print hello world
Parent commit      : nmptruqn 90a2e97f refactor printing
Added 0 files, modified 1 files, removed 0 files
```

`jj next` will move `@`, the working copy change, to the child of where it is
now. The `--edit` flag means we're now going to be editing that change, whereas
if you leave it off, it works more like a variant of `jj new`, making a new
change based on top of that change.

Let's double check with `jj log`:

```console
$ jj log
@  ootnlvpt steve@steveklabnik.com 2024-02-28 23:26:44.000 -06:00 b5db7940
│  only print hello world
◉  nmptruqn steve@steveklabnik.com 2024-02-28 23:09:11.000 -06:00 90a2e97f
│  add more comments
◉  ywnkulko steve@steveklabnik.com 2024-02-28 22:09:40.000 -06:00 ed71bb54
│  print goodbye as well as hello
◉  puomrwxl steve@steveklabnik.com 2024-02-28 20:38:13.000 -06:00 7a096b8a
│  it's important to comment our code
◉  yyrsmnoo steve@steveklabnik.com 2024-02-28 20:24:56.000 -06:00 ac691d85
│  hello world
◉  zzzzzzzz root() 00000000
```

That's correct, `@` is at our original change.

## Recap and thoughts

This workflow is also a good alternative. If your brain thinks this way better
than the other way, that's great! A nice thing about the flexibility of these
tools is you can work with them how you'd like!
