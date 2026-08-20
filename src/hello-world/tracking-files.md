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
Working copy  (@) : rsrkkypy 5709764b (no description set)
Parent commit (@-): pqxxuvwy 8ed64743 second line
```

There's no staging area, so there's nothing to add things to. Remember, `@` is
a commit, and its contents are whatever is in our working directory. This is
the same snapshotting we've relied on all along, even for files that didn't
exist before.

This is convenient, but that file has a suspicious name! We probably didn't
want `secrets.env` in a commit. Let's see how to keep it out.

## Ignoring files

`jj` reads `.gitignore`, with the same syntax you already know:

```console
$ echo "*.env" > .gitignore

$ jj st
Working copy changes:
A .gitignore
A secrets.env
Working copy  (@) : rsrkkypy 6b873e2a (no description set)
Parent commit (@-): pqxxuvwy 8ed64743 second line
```

`.gitignore` shows up because it's a normal file, and you want it committed so
that everyone working on the project ignores the same things. `secrets.env`
shows up too: the previous `jj st` snapshotted it into `@`, so it's already
tracked. Adding an ignore rule never removes a tracked file, exactly as in
`git`. That's what `jj file untrack` is for:

```console
$ jj file untrack secrets.env

$ jj st
Working copy changes:
A .gitignore
Working copy  (@) : rsrkkypy 1c7d8fe0 (no description set)
Parent commit (@-): pqxxuvwy 8ed64743 second line
```

The file is out of the commit and still on disk, which is what you want for
something like a local config file:

```console
$ ls
notes.txt    secrets.env
```

`jj` insists you ignore it first:

```console
$ jj file untrack notes.txt
Error: 'notes.txt' is not ignored.
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
$ jj file show -r @- notes.txt
line one
line two
```

Listing what a revision contains:

```console
$ jj file list
.gitignore
notes.txt
```

Searching the tracked files:

```console
$ jj file search --pattern "line two" .
notes.txt:line two
```

And here's one worth remembering, because the name may not suggest it at first.
`jj file annotate` is the equivalent of `git blame`:

```console
$ jj file annotate notes.txt
mtvqnnvv steve    2024-03-27 14:31:39    1: line one
pqxxuvwy steve    2024-03-27 14:31:39    2: line two
```

We get each line along with the change that last touched it. The first column is
a change ID, so we can pass it straight to `jj show` to see what that change
was doing.
