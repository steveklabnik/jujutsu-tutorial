# Absorbing changes with `jj absorb`

We've seen that `jj squash` moves our work into one commit, while `jj split`
breaks one commit into two. Both need us to say where the work should go.
`jj absorb` can work that out for us.

The situation it's built for turns up all the time. Let's say we have a stack
of finished commits and, while reading over them, we spot small problems in
several commits at once: a typo here, a rename we missed there. We fix them all
in our working copy, and now we have one change whose pieces belong in several
different commits.

Here's a small version. Two finished commits, each having touched one file:

```console
$ jj log
@  wmtwzxmv steve@steveklabnik.com 2024-03-26 09:14:48 e9cfed95
│  (no description set)
○  towkwvlp steve@steveklabnik.com 2024-03-26 09:14:48 013493d0
│  add an evaluator
○  wumvwlxp steve@steveklabnik.com 2024-03-26 09:14:48 8fa7ccba
│  add a parser
○  rlrtxyru steve@steveklabnik.com 2024-03-26 09:14:48 26dcb6b5
│  initial
◆  zzzzzzzz root() 00000000
```

We've gone back and touched up both of them from the working copy:

```console
$ jj st
Working copy changes:
M a.txt
M b.txt
Working copy  (@) : wmtwzxmv e9cfed95 (no description set)
Parent commit (@-): towkwvlp 013493d0 add an evaluator
```

We could sort this out by hand with two `jj squash --into` commands and the
right paths, or with a `jj split` followed by two rebases. Instead, let's try
`jj absorb`:

```console
$ jj absorb
Absorbed changes into 2 revisions:
  towkwvlp 012a265e add an evaluator
  wumvwlxp 4891076c add a parser
Working copy  (@) now at: ttoyvukr 4c383532 (empty) (no description set)
Parent commit (@-)      : towkwvlp 012a265e add an evaluator
```

Each hunk went where it belonged, the descendants were rebased, and the working
copy is empty again:

```console
$ jj log
@  ttoyvukr steve@steveklabnik.com 2024-03-26 09:14:48 4c383532
│  (empty) (no description set)
○  towkwvlp steve@steveklabnik.com 2024-03-26 09:14:48 012a265e
│  add an evaluator
○  wumvwlxp steve@steveklabnik.com 2024-03-26 09:14:48 4891076c
│  add a parser
○  rlrtxyru steve@steveklabnik.com 2024-03-26 09:14:48 26dcb6b5
│  initial
◆  zzzzzzzz root() 00000000
```

## How it decides

For each hunk, `jj` looks at which commit last touched those lines, and puts the
hunk there. That's the whole rule. It's the same question `git blame` answers,
used to route your edit rather than to assign blame.

This has two useful consequences:

* A hunk whose lines nobody has touched has no home, so it stays in your working
  copy. Add a brand new file and `jj absorb` will tell you `Nothing changed`.
  Nothing gets guessed at.
* It only writes to commits you're allowed to rewrite. Lines whose last change
  is in an immutable commit — anything on trunk — are left where they are.

So `jj absorb` moves the parts it can place confidently and leaves the rest for
us. If the working copy still has something in it afterwards, that's the part
we need to decide about.

## Narrowing it

Paths limit it to part of the tree:

```console
$ jj absorb src/
```

and `--from` / `--into` set the source and the candidate destinations
explicitly:

```console
$ jj absorb --from @ --into 'mutable()'
```

The default source is `@`, and the default destinations are the mutable
ancestors, which is what you want almost every time.

## Where this fits

`jj squash`, `jj split`, and `jj absorb` are the three commands that move work
between commits after the fact. We can squash to combine, split to separate,
and absorb when the destination is clear from the history. Between them, we can
tidy up a stack before review without rebuilding it by hand.
