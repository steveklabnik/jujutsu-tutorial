# Seeing what changed with `jj diff` and `jj show`

So far, `jj st` has told us *which* files we've touched, and `jj log` has told
us which changes exist. Neither tells us what's actually in those changes. For
that, we can use `jj diff`:

```console
$ jj diff
Modified regular file main.rs:
   1    1: fn main() {
   2    2:     println!("Hello, world!");
        3:     println!("Goodbye, world!");
   3    4: }
```

With no arguments it shows what's in `@`: the difference between your working
copy and its parent. Since your working copy is a commit like any other, that's
the same thing as "what this commit contains".

This format can take a moment to read if you're used to `git`. There are two
columns of line numbers: the line's position before, and after. A line present
in both gets both numbers. A line only in the new version, like our `Goodbye`,
has an empty left column. Deleted lines are the other way round:

```console
$ jj diff
Modified regular file main.rs:
   1    1: fn main() {
   2     :     println!("Hello, world!");
        2:     println!("Hi!");
   3    3: }
```

Line 2 went away and a new line 2 arrived. In a terminal this is colored, and
`jj` highlights the words that differ rather than just the lines, which is why
it's called a "color words" diff. It's good at showing small edits inside long
lines, where `git`'s line-at-a-time view makes you hunt.

If you'd rather have the format every other tool understands:

```console
$ jj diff --git
diff --git a/main.rs b/main.rs
index e7a11a969c..4e787b01da 100644
--- a/main.rs
+++ b/main.rs
@@ -1,3 +1,4 @@
 fn main() {
     println!("Hello, world!");
+    println!("Goodbye, world!");
 }
```

This is handy when we want to paste a patch somewhere. We can also use `-s` to
get just the summary, which is the same list of names `jj st` shows:

```console
$ jj diff -s
M main.rs
```

## Any change, not just this one

We can use `-r` to show a different change:

```console
$ jj diff -r @-
Added regular file main.rs:
        1: fn main() {
        2:     println!("Hello, world!");
        3: }
```

An added file has nothing in the left column at all, which makes sense — none of
it was there before.

`--from` and `--to` compare two revisions that needn't be adjacent:

```console
$ jj diff --from @-- --to @
Modified regular file main.rs:
   1    1: fn main() {
   2     :     println!("Hello, world!");
        2:     println!("Hi!");
        3:     println!("Goodbye, world!");
   3    4: }
```

This is the combined effect of two changes. I find this useful before pushing:
`jj diff --from trunk` shows everything our branch does.

## `jj show`

`jj diff` shows us the contents of a change. `jj show` adds who made it, when
they made it, and why:

```console
$ jj show
Commit ID: 8ad74e791c831f2f39cdde695fce5e7553b85f4c
Change ID: wvokwvnpzwqkmmxzsruqvmwpzkxuupom
Author   : Steve Klabnik <steve@steveklabnik.com> (2024-03-26 11:04:04)
Committer: Steve Klabnik <steve@steveklabnik.com> (2024-03-26 11:04:05)

    add a goodbye

Modified regular file main.rs:
   1    1: fn main() {
   2    2:     println!("Hello, world!");
        3:     println!("Goodbye, world!");
   3    4: }
```

We get both IDs in full, both timestamps, the description, and the diff.
`jj show -r` takes a revision in the same way. I tend to use `jj show` when I'm
inspecting somebody else's commit, and `jj diff` when I'm checking my own work.

Note the two timestamps. Author is when the change was first made; committer is
when this version of it was written. Rewrite a commit — amend it, rebase it —
and the second moves while the first stays put.

## Comparing two versions of the same change

There's a third command for a situation `git` doesn't handle very well. Let's
say we've pushed a pull request, someone has reviewed it, and then we've
rewritten the change. We may want to know what changed between those two
versions, rather than what the change itself does.

```console
$ jj interdiff --from 64fe802a --to @
Modified commit description:
   1     : feature v1
        1: feature v2
Modified regular file feat.txt:
   1     : one
        1: one and two
```

That's `jj interdiff`, and it compares the *effects* of two commits rather than
their contents, so the work underneath them doesn't get in the way. The old
commit ID comes from `jj evolog`, which we'll meet properly later — it keeps
every version of a change, so the thing the reviewer saw is still there to
compare against.

I find it surprisingly useful to be able to answer "what did you change since
my review?" precisely. This is another benefit of `jj` keeping old versions of
our changes rather than overwriting them.
