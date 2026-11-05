# Creating a repository with `jj git init`

Let's make a new repository! First, we need a project to track. I am going
to use a Rust project in this example, since it is my favorite language, but
you can use whatever you'd like: we won't be writing complex code here, just
giving ourselves something to work with.

We can use `cargo new` to make a new Rust project, and we'll tell it not to
create any version control repository for us, so we can do it ourselves.

```console
$ cargo new hello-world --vcs=none
     Created binary (application) `hello-world` package
$ cd hello-world
```

In Cargo projects, the main source file is stored in `src/main.rs`, whose
contents look like this:

```rust
fn main() {
    println!("Hello, world!");
}
```

Perfect. Now, this is kind of funny, but `jj` doesn't have an equivalent of
`.gitignore`, and instead, just supports `.gitignore`. So let's put this in
a `.gitignore` file:

```console
/target/
```

If you're using another language, you may want to add something like
`node_modules` if you're in JavaScript, or the equivalent of whatever language
you're using.

Now that we've got a project, let's initialize our repository:

```console
$ jj git init
Initialized repo in "."
```

Now, you may be wondering, "why not just `jj init`?" The deal is this: the
native repository format is still a work in progress. So we're creating a
repository that's backed by a real `git` repository, because in practice, this
early in `jj`'s life, that's the right thing to do. There's no `jj init`
command at all; if you type it, `jj` will tell you that you probably wanted
`jj git init`.

If you peek at the directory listing, you'll notice something: `jj git init`
made *two* directories.

```console
$ ls -d .git .jj
.git
.jj
```

This is called a "colocated" repository: a `.jj` directory and a real `.git`
directory side by side, over one working copy. It's the default, and it's what
we'll use throughout this tutorial. `jj` imports from and exports to the `git`
repository on every command, so `git`, `gh`, your editor, and CI all see a
perfectly normal `git` repo. Mind you, if you run `git log` right this second,
it will complain that there aren't any commits yet — we haven't made any! Once
we have some history, `git log` will show the commits we make with `jj`, and
we'll see exactly that later in the tutorial.

Because both tools share one store, we'll let `jj` make changes to the
repository and use `git` for reading it. We'll look at the details later in the
tutorial. There is an escape hatch, `--no-colocate`, for workflows that need to
keep Git-aware tools away from the working copy. Unless we have a specific
reason to make that tradeoff, the colocated default is the useful place to
start.

Anyway, now we've got a repository! In the next section, we'll take a peek
inside.
