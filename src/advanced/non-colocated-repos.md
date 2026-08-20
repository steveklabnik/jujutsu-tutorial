# Non-colocated repositories

Every repository in this book has been a colocated one: a `.jj` directory and a
real `.git` directory over a single working copy, which is what `jj git init`
gives you and what chapter one described. `git` can see everything you do, and
`gh`, your editor, and any other tool that reads `.git` work without knowing
`jj` exists.

There's a second arrangement, and it's worth knowing about even if you never
switch to it, because it shows you where the line between `jj` and `git`
actually sits.

```console
$ jj git init --no-colocate
Initialized repo in "."

$ ls -a
.
..
.jj
```

One directory. No `.git`:

```console
$ git status
fatal: not a git repository (or any of the parent directories): .git
```

## The `git` repository is still there

`jj` is still backed by `git`. The store just moved inside `.jj`, where nothing
finds it by accident:

```console
$ jj git colocation status
Workspace is currently not colocated with Git.
Last imported/exported Git HEAD: (none)
Hint: To enable colocation, run: `jj git colocation enable`

$ jj git root
/home/steve/src/my-project/.jj/repo/store/git
```

Point `git` at that path and it works normally:

```console
$ GIT_DIR=$(jj git root) GIT_WORK_TREE=$(jj workspace root) git status
```

`gh` is the same story — `GIT_DIR=$(jj git root) gh pr list`. Nothing is
unavailable. It just has to be asked for.

Remotes are entirely unaffected. `jj git fetch`, `jj git push`, and
`jj git clone --no-colocate` behave exactly as they do in a colocated
repository, because they talk to the store directly and never needed a
top-level `.git` in the first place.

## Exports stop being automatic

Here's the difference that matters in practice. A colocated repository imports
`git` state and exports its own on *every* command. That's what keeps the two
views in step. Without colocation there's nothing to stay in step with, so `jj`
doesn't bother:

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

`jj git import` goes the other way. In a colocated repository you rarely type
either of these; here they're how the `git` view gets updated at all. If you're
reaching for `git` in a non-colocated repository often enough to keep exporting,
that's an argument for colocating.

## Nothing to explain to `git`

The other consequence is a nuisance you stop having.

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

## Choosing

Colocation buys you an ecosystem for free and asks for one discipline in return:
let `jj` do the mutating. Read-only `git` commands are fine and often the
quickest way to see something. Mutating ones — `git commit`, `git rebase`,
moving a branch — can be imported afterwards, but they race `jj`'s own
import and export if run concurrently, they can't be handed over half-finished,
and moving a bookmark in both tools gets you the `name??` conflict from an
earlier chapter.

Non-colocation enforces that discipline instead of asking for it. There's no
`.git` for an editor plugin to act on, no `git commit` typed out of habit, no
script that discovers a repository you didn't mean it to find. Some people want
that, particularly on a machine running tooling they don't control.

Most people, most of the time, want the default. If you work on GitHub and use
`gh`, colocation costs you nothing and saves you a prefix on every command.
Either way you're not committed:

```console
$ jj git colocation disable
Workspace successfully converted into a non-colocated Jujutsu/Git workspace.
```

`jj git colocation enable` converts back, and `git.colocate = false` in your
config makes `--no-colocate` the default for new repositories.

## Native repositories

Both arrangements keep your commits in a `git` repository. The only difference
is where it sits and who else can see it. You can confirm that in any repository
you have:

```console
$ jj util backend name
git
```

`jj` does have a storage format of its own, and chapter one mentioned it in
passing. It is still a work in progress, and `jj` 0.44.0 does not provide a
command for creating a repository that uses it. For now, both arrangements in
this chapter use the `git` backend; colocation only decides whether the Git
repository is exposed at the top of our working copy.

## What `git` gives up either way

A few `git` features aren't implemented by `jj` at all, colocated or not:

* **Submodules** are not materialized in the working copy. They survive, but
  `jj` won't check them out or update them.
* **Git LFS** is unsupported. Pointer files are ordinary files.
* **`git` hooks don't run.** No pre-commit, no pre-push. Use `jj fix` or CI.
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
