# Dealing with conflicts

When you're merging or rebasing, if your changes are incompatible with each
other, you may introduce a conflict. Conflicts are often regarded as painful
by users of version control systems. `jj` can't make that pain go away entirely,
but it can help a lot.

Let's deliberately introduce a conflict. First, we make a new change:

```console
$ jj new -m "remove goodbye message"
Working copy now at: povouosx e2c9628c (empty) remove goodbye message
Parent commit      : yykpmnuq 2b93da0c (empty) add better documentation
```

And then update `src/main.rs` appropriately:

```rust
/// A "Hello, world!" program.
/// 
/// This is the best implementation of this program to ever exist.

fn main() {
    print_hello();
}

fn print_hello() {
    println!("Hello, world!");
}
```

Let's also make a new change off of the previous head:

```console
$ jj new yykpmnuq -m "refactor printing"
Working copy now at: vvmrvwuz 44205653 (empty) refactor printing
Parent commit      : yykpmnuq 2b93da0c (empty) add better documentation
Added 0 files, modified 1 files, removed 0 files
```

And if we open `src/main.rs` again, we'll see that of course, it's back to the
state it was before we made our other change:

```rust
/// A "Hello, world!" program.
///
/// This is the best implementation of this program to ever exist.

fn main() {
    print_hello();
    print_goodbye();
}

fn print_hello() {
    println!("Hello, world!");
}

fn print_goodbye() {
    println!("Goodbye, world!");
}
```

Let's make a very silly change: our own print function. Edit `src/main.rs` to
look like this:

```rust
/// A "Hello, world!" program.
/// 
/// This is the best implementation of this program to ever exist.

fn main() {
    print("Hello, world!");
    print("Goodbye, world!");
}

fn print(m: &str) {
    println!("{m}")
}
```

Excellent:

```console
$ jj log --limit 3
@  vvmrvwuz steve@steveklabnik.com 2024-03-01 17:29:12.000 -06:00 5f858c15
│  refactor printing
│ ◉  povouosx steve@steveklabnik.com 2024-03-01 17:27:14.000 -06:00 28010506
├─╯  remove goodbye message
◉  yykpmnuq steve@steveklabnik.com 2024-03-01 17:07:36.000 -06:00 2b93da0c
│  (empty) add better documentation
```

Everything looks to be in order. Let's rebase our goodbye message change onto
our refactor printing change:

```rust
$ jj rebase -r povouosx -d @
New conflicts appeared in these commits:
  povouosx 793ce8e0 (conflict) remove goodbye message
To resolve the conflicts, start by updating to it:
  jj new povouosxlror
Then use `jj resolve`, or edit the conflict markers in the file directly.
Once the conflicts are resolved, you may want inspect the result with `jj diff`.
Then run `jj squash` to move the resolution into the conflicted commit.
```

Wait a minute, I thought I told you that rebases always succeed. Well... it did:

```console
> jj log --limit 3
◉  povouosx steve@steveklabnik.com 2024-03-01 17:30:32.000 -06:00 793ce8e0 conflict
│  remove goodbye message
@  vvmrvwuz steve@steveklabnik.com 2024-03-01 17:29:12.000 -06:00 5f858c15
│  refactor printing
◉  yykpmnuq steve@steveklabnik.com 2024-03-01 17:07:36.000 -06:00 2b93da0c
│  (empty) add better documentation
```

Remember, `@` stays where it is, and we moved a commit ahead of us, so we're
good. Go ahead, check out `src/main.rs`, you'll see that it's still just like
it was before, with our printer refactoring.

However, you'll notice that in our log output, it says that `povouosx` is now
*conflicted*. This is why rebases always succeed in `jj`: if there's a conflict,
it doesn't make you stop and fix it, it records that there's a conflict and
still performs the rest of the rebase. This is *very* powerful. Even in this
case with one change, it lets us handle the conflict when we're ready. We can
keep making changes to our current change if we want to:

```rust
/// A "Hello, world!" program.
/// 
/// This is the best implementation of this program to ever exist.

fn main() {
    print("Hello, world!");
    print("Goodbye, world!");
}

// a function that prints a message
fn print(m: &str) {
    println!("{m}")
}
```

And then we check our log again:

```console
$ jj log --limit 3
Rebased 1 descendant commits onto updated working copy
◉  povouosx steve@steveklabnik.com 2024-03-01 17:49:07.000 -06:00 a912c809 conflict
│  remove goodbye message
@  vvmrvwuz steve@steveklabnik.com 2024-03-01 17:49:07.000 -06:00 d41c079b
│  refactor printing
◉  yykpmnuq steve@steveklabnik.com 2024-03-01 17:07:36.000 -06:00 2b93da0c
│  (empty) add better documentation
```

`jj` automatically rebased `povouosx` again. It's still in conflict. But that's
totally okay. We only need to handle it when we're ready. This automatic
rebasing behavior only works because `jj` is okay with commits being in
conflict. And if we had even more children commits, they'd *all* be rebased,
automatically.

## Resolving the conflict

The output we got back when the conflict was created gave us some advice:

```text
To resolve the conflicts, start by updating to it:
  jj new povouosxlror
Then use `jj resolve`, or edit the conflict markers in the file directly.
Once the conflicts are resolved, you may want inspect the result with `jj diff`.
Then run `jj squash` to move the resolution into the conflicted commit.
```

This advice is good, but also more complex than we need to do right now. Doing
this is a great way to handle a complex resolution, where you want to double
check what you've done before you apply the changes. But we are just using a
small example to make a point. Therefore, we can just edit `povouosxlror` and
remove the conflict markers directly:

```console
> jj edit povouosx
Working copy now at: povouosx a912c809 (conflict) remove goodbye message
Parent commit      : vvmrvwuz d41c079b refactor printing
Added 0 files, modified 1 files, removed 0 files
```

Here's `src/main.rs`:

```rust
/// A "Hello, world!" program.
/// 
/// This is the best implementation of this program to ever exist.

fn main() {
<<<<<<<
+++++++
    print("Hello, world!");
    print("Goodbye, world!");
%%%%%%%
     print_hello();
-    print_goodbye();
>>>>>>>
}

// a function that prints a message
fn print(m: &str) {
    println!("{m}")
}
```

`git` uses a combination of `<<<<`, `=====`, and `>>>>` to mark conflicts. `jj`
has more rich conflict markers. It still uses the `>>>` and `<<<`s to indicate
the start and end, but has two other markers: `+++++++` and `%%%%%%%`. The `+`s
indicate the start of a snapshot, and `%`s mark the start of a diff. So we can
see that we have two `print` lines, but our changes wanted to remove one of them,
but since that's not the same thing, conflict. To resolve this, we apply our own
take on the diff to the snapshot:

```rust
/// A "Hello, world!" program.
/// 
/// This is the best implementation of this program to ever exist.

fn main() {
    print("Hello, world!");
}

// a function that prints a message
fn print(m: &str) {
    println!("{m}")
}
```

Let's take a look:

```console
$ jj st
Working copy changes:
M src\main.rs
Working copy : povouosx 7647f7a0 remove goodbye message
Parent commit: vvmrvwuz d41c079b refactor printing
$ jj log --limit 3
@  povouosx steve@steveklabnik.com 2024-03-01 18:08:23.000 -06:00 7647f7a0
│  remove goodbye message
◉  vvmrvwuz steve@steveklabnik.com 2024-03-01 17:49:07.000 -06:00 d41c079b
│  refactor printing
◉  yykpmnuq steve@steveklabnik.com 2024-03-01 17:07:36.000 -06:00 2b93da0c
│  (empty) add better documentation
```

Conflict resolved!

## Automatic rebasing conflict resolution

A wild thing about this though is the combination of conflicted changes and
automatic rebasing. Here, I'll show you. First we need to undo our resolution,
and then we'll make a new change on top of our conflicted change:

```console
> jj undo
New conflicts appeared in these commits:
  povouosx a912c809 (conflict) remove goodbye message
To resolve the conflicts, start by updating to it:
  jj new povouosxlror
Then use `jj resolve`, or edit the conflict markers in the file directly.
Once the conflicts are resolved, you may want inspect the result with `jj diff`.
Then run `jj squash` to move the resolution into the conflicted commit.
Working copy now at: povouosx a912c809 (conflict) remove goodbye message
Parent commit      : vvmrvwuz d41c079b refactor printing
Added 0 files, modified 1 files, removed 0 files
> jj new povouosxlror --no-edit
Created new commit mlzwmxzs 07bb727d (conflict) (empty) (no description set)
> jj log --limit 4
◉  mlzwmxzs steve@steveklabnik.com 2024-03-01 18:10:08.000 -06:00 07bb727d conflict
│  (empty) (no description set)
@  povouosx steve@steveklabnik.com 2024-03-01 17:49:07.000 -06:00 a912c809 conflict
│  remove goodbye message
◉  vvmrvwuz steve@steveklabnik.com 2024-03-01 17:49:07.000 -06:00 d41c079b
│  refactor printing
◉  yykpmnuq steve@steveklabnik.com 2024-03-01 17:07:36.000 -06:00 2b93da0c
│  (empty) add better documentation
```

We have our conflict, and then our new change, `mlzwmxzs`, is also in conflict.
So fix the conflict in `main.rs` again, and then let's see what happens:

```console
> jj log --limit 4
Rebased 1 descendant commits onto updated working copy
◉  mlzwmxzs steve@steveklabnik.com 2024-03-01 18:12:43.000 -06:00 9a4ad229
│  (empty) (no description set)
@  povouosx steve@steveklabnik.com 2024-03-01 18:12:43.000 -06:00 f68d1623
│  remove goodbye message
◉  vvmrvwuz steve@steveklabnik.com 2024-03-01 17:49:07.000 -06:00 d41c079b
│  refactor printing
◉  yykpmnuq steve@steveklabnik.com 2024-03-01 17:07:36.000 -06:00 2b93da0c
│  (empty) add better documentation
```

Not only did we fix our issue, but after we did, `jj` automatically rebased
`mlzwmxzs`, and the fix propagated correctly. `mlzwmxzs` is no longer in
conflict.

By the way, let's abandon that change, as we don't intend to use it for anything
right now:

```console
$ jj abandon mlzwmxzs
Abandoned commit mlzwmxzs 9a4ad229 (empty) (no description set)
```

Great, we've cleaned that up.

These behaviors, namely recording conflicts and automatic rebasing, form the
behaviors necessary for a very cool `jj` workflow, and that's stacking pull
requests. Before we talk about that though, we have to talk about how to
use `jj` with GitHub in the first place! Let's go over that next.
