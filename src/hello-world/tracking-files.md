# Which files `jj` tracks

You may have noticed something missing. We've created a bunch of files, and
they've shown up in our commits, but at no point did we run anything like
`git add`. Let's talk about why.

New files are tracked automatically. If we make one, it will be part of our
working copy commit:

```console
$ echo "secret" > secrets.env

$ jj st
Working copy changes:
A secrets.env
Working copy  (@) : ywnkulko 0c334409 (no description set)
Parent commit (@-): puomrwxl 7a096b8a it's important to comment our code
```

There's no staging area, so there's nothing to add things to. Remember, `@` is
a commit, and its contents are whatever is in our working directory. This is
the same snapshotting we've relied on all along, even for files that didn't
exist before.

This is convenient, but that file has a suspicious name! We probably didn't
want `secrets.env` in a commit. Let's see how to keep it out.

## Ignoring files

`jj` reads `.gitignore`, with the same syntax you already know. We already have
one, from back when we created the project — it keeps `cargo`'s build output
out of our repository. Let's add a rule to it:

```console
$ echo "*.env" >> .gitignore

$ jj st
Working copy changes:
M .gitignore
A secrets.env
Working copy  (@) : ywnkulko 6ac6112a (no description set)
Parent commit (@-): puomrwxl 7a096b8a it's important to comment our code
```

`.gitignore` shows up as `M`odified, which makes sense: it's a normal, tracked
file, and you want it committed so that everyone working on the project ignores
the same things. But `secrets.env` still shows up too! The previous `jj st`
snapshotted it into `@`, so it's already tracked. Adding an ignore rule never
removes a tracked file, exactly as in `git`. That's what `jj file untrack` is
for:

```console
$ jj file untrack secrets.env

$ jj st
Working copy changes:
M .gitignore
Working copy  (@) : ywnkulko f77fddd4 (no description set)
Parent commit (@-): puomrwxl 7a096b8a it's important to comment our code
```

The file is out of the commit and still on disk, which is what you want for
something like a local config file:

```console
$ ls
Cargo.lock    Cargo.toml    secrets.env    src
```

`jj` insists you ignore it first:

```console
$ jj file untrack Cargo.toml
Error: 'Cargo.toml' is not ignored.
Hint: Files that are not ignored will be added back by the next command.
Make sure they're ignored, then try again.
```

This may seem fussy, but it makes sense. If the file wasn't ignored, `jj` would
track it again the next time it took a snapshot. Requiring both steps keeps us
from thinking we ignored something when we didn't.

One thing this doesn't do: untracking removes the file from `@` going forward,
not from history. A password you committed five changes ago is still in those
changes, and still in what you push. Getting it out of the past is a rewrite,
not an untrack.

## Looking inside files

There are a handful of other useful commands under `jj file`. We can read a
file as of a particular change without touching our working copy:

```console
$ jj file show -r @- src/main.rs
/// A "Hello, world!" program.

fn main() {
    println!("Hello, world!");
}
```

Listing what a revision contains:

```console
$ jj file list
.gitignore
Cargo.lock
Cargo.toml
src/main.rs
```

Searching the tracked files:

```console
$ jj file search --pattern "Hello" .
src/main.rs:/// A "Hello, world!" program.
src/main.rs:    println!("Hello, world!");
```

And here's one worth remembering, because the name may not suggest it at first.
`jj file annotate` is the equivalent of `git blame`:

```console
$ jj file annotate src/main.rs
puomrwxl steve@st 2024-02-28 20:38:13    1: /// A "Hello, world!" program.
puomrwxl steve@st 2024-02-28 20:38:13    2: 
yyrsmnoo steve@st 2024-02-28 20:24:56    3: fn main() {
yyrsmnoo steve@st 2024-02-28 20:24:56    4:     println!("Hello, world!");
yyrsmnoo steve@st 2024-02-28 20:24:56    5: }
```

We get each line along with the change that last touched it. The first column is
a change ID, so we can pass it straight to `jj show` to see what that change
was doing.

That's it for our tour of the basics. Before we recap, let's clean up after
this experiment: delete `secrets.env`, and take the `*.env` line back out of
`.gitignore`. That leaves our working copy empty again, ready for the next
section.
