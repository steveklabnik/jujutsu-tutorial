# Non-colocated repositories

Every repository in this book has been a colocated one. As we saw in chapter
one, `jj git init` gives us a `.jj` directory and a real `.git` directory over
the same working copy. `git` can see everything we do, and tools such as `gh`
and our editor can work without knowing that `jj` exists.

There's a second arrangement. Even if we never switch to it, looking at it will
help us understand where the line between `jj` and `git` actually sits.

```console
$ jj git init --no-colocate
Initialized repo in "."

$ ls -a
.
..
.jj
```

Now we have one directory and no `.git`:

```console
$ git status
fatal: not a git repository (or any of the parent directories): .git
```

## The `git` repository is still there

We are still using the `git` backend. Its store has moved inside `.jj`, where
other tools won't find it by accident:

```console
$ jj git colocation status
Workspace is currently not colocated with Git.
Last imported/exported Git HEAD: (none)
Hint: To enable colocation, run: `jj git colocation enable`

$ jj git root
/home/steve/src/my-project/.jj/repo/store/git
```

If we point `git` at that path explicitly, it works normally:

```console
$ GIT_DIR=$(jj git root) GIT_WORK_TREE=$(jj workspace root) git status
```

`gh` is the same story — `GIT_DIR=$(jj git root) gh pr list`. Nothing is
unavailable. It just has to be asked for.

Remotes are entirely unaffected. We can use `jj git fetch`, `jj git push`, and
`jj git clone --no-colocate` exactly as we did in a colocated repository. These
commands talk to the store directly, so they never needed a top-level `.git` in
the first place.

## Exports stop being automatic

Here's the difference that matters in practice. Our colocated repositories
imported `git` state and exported their own on *every* command. That's what kept
the two views in step. Without colocation, there is no top-level `git` view to
keep in step, so `jj` doesn't do that automatically:

```console
$ jj bookmark create second -r @-
Created 1 bookmarks pointing to okxrzpsx 66ac5baa second | first

$ GIT_DIR=$(jj git root) git branch
  feature
```

The bookmark exists in `jj` and the `git` store hasn't heard about it. Run more
`jj` commands and it still hasn't. You have to say so:

```console
$ jj git export

$ GIT_DIR=$(jj git root) git branch
  feature
  second
```

`jj git import` goes the other way. In a colocated repository, we rarely need
either command. Here, they are how we update the `git` view at all. If we find
ourselves exporting every few minutes, that's a good sign that colocation would
fit our workflow better.

## Keeping `git` out of the working copy

This also changes what `git`-based tools see.

In a colocated repository, `git`'s `HEAD` is detached and points at a *parent*
of `@`, because the working copy being a commit is an idea `git` can't state.
So after `jj edit` on a finished commit, `git status` reports its already
recorded patch as an uncommitted modification:

```console
$ git status --short
 M f.txt
```

That's `git` accurately describing the gap between `HEAD` and the files, and
it's harmless — but it surprises people, and some editors and pre-commit tooling
behave oddly around a tree they think is dirty. In a non-colocated repository
nothing is looking, so nothing is confused.

## Choosing between them

Colocation gives us access to the `git` ecosystem, with one important rule: let
`jj` make the changes. Read-only `git` commands are fine and are often the
quickest way to inspect something. Mutating commands such as `git commit` and
`git rebase` can be imported afterwards, but they can race `jj`'s own import and
export if run concurrently. `jj` also can't take over one of these operations
halfway through, and moving a bookmark in both tools can produce the `name??`
conflict from an earlier chapter.

Non-colocation enforces that discipline instead of asking for it. There's no
`.git` for an editor plugin to act on, no `git commit` typed out of habit, no
script that discovers a repository you didn't mean it to find. Some people want
that, particularly on a machine running tooling they don't control.

I think the default is right for most people. If we work on GitHub and use
`gh`, colocation lets those tools work normally. We aren't locked into either
choice:

```console
$ jj git colocation disable
Workspace successfully converted into a non-colocated Jujutsu/Git workspace.
```

`jj git colocation enable` converts back, and `git.colocate = false` in your
config makes `--no-colocate` the default for new repositories.

## Native repositories

Both arrangements keep our commits in a `git` repository. The only difference
is where it sits and who else can see it. We can confirm that in any repository:

```console
$ jj util backend name
git
```

`jj` does have a storage format of its own, and chapter one mentioned it in
passing. It is still a work in progress, and `jj` 0.44.0 does not provide a
command for creating a repository that uses it. For now, both arrangements in
this chapter use the `git` backend; colocation only decides whether the Git
repository is exposed at the top of our working copy.

## Unsupported `git` features

A few `git` features aren't implemented by `jj`, whether or not the repository
is colocated:

* **Submodules** are not materialized in the working copy. They survive, but
  `jj` won't check them out or update them.
* **Git LFS** is unsupported. Pointer files are ordinary files.
* **`git` hooks don't run.** No pre-commit, no pre-push. `jj fix`, which we
  covered a couple of chapters ago, replaces the formatting ones; CI has to
  cover the rest.
* **`.gitattributes` is ignored** — no clean/smudge filters, no line-ending
  normalization.
* **Annotated tags** can be read and checked out, but not created.
* **`git config` is mostly ignored**, except remote configuration and
  `core.excludesFile`.
* **Shallow clones** work, but deepening one doesn't; partial clone is
  unsupported.

In a colocated repository, `jj` also puts an ignore file inside `.jj` so a
`git add -A` does not add its internal state. We don't need to add `.jj/` to the
project's own `.gitignore`.
