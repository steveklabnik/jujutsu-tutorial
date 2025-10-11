# Using `jj new` to create new changes

We're done with our first commit, and we're ready to do more work. Let's start
that work by using `jj new`:

```console
$ jj new
Working copy now at: puomrwxl 01a35aad (empty) (no description set)
Parent commit      : yyrsmnoo ac691d85 hello world
```

It's that easy! We now have a new change, `puomrwxl`, that's empty and has
no description. But its parent is our previous change, `yyrsmnoo`.

Let's check out `jj st`:

```console
$ jj st
The working copy is clean
Working copy : puomrwxl 01a35aad (empty) (no description set)
Parent commit: yyrsmnoo ac691d85 hello world
```

Nice, a clean working copy: all of our changes were made in `yyrsmnoo`, and
we're starting this change fresh.

We now technically have a very primitive, but near-complete, workflow. That's
really all you need to know to get started. To practice, let's make another
change. This time, I'm going to describe things first, before I make any changes:

```console
> jj describe -m "it's important to comment our code"
Working copy now at: puomrwxl a0f0bc71 (empty) it's important to comment our code
Parent commit      : yyrsmnoo ac691d85 hello world
```

Just what we expected, still an empty change, but with a description, and our
commit ID has updated while the change ID stays the same.

Let's modify `src/main.rs`:

```rust
/// A "Hello, world!" program.

fn main() {
    println!("Hello, world!");
}
```

We can double check that `jj` has noticed our change:

```console
$ jj st
Working copy changes:
M src\main.rs
Working copy : puomrwxl 7a096b8a it's important to comment our code
Parent commit: yyrsmnoo ac691d85 hello world
```

Excellent, `src\main.rs` has been `M`odified, we have a new commit ID. Since
we're done with this change, let's start a new one:

```console
$ jj new
Working copy now at: ywnkulko 46b50ed7 (empty) (no description set)
Parent commit      : puomrwxl 7a096b8a it's important to comment our code
```

Wonderful.

Just seeing the parent change is very restrictive, though. It would be nice
if we could look at all of the work we've done. Let's tackle that next.
