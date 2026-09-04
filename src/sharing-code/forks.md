# Working with a fork

When we contribute to a project we can't push to directly, we usually have two
remotes: we fetch from the project's repository and push to our own fork. `jj`
handles this much like `git`, but there are a couple of details worth knowing.

Let's add the project's repository as our second remote:

```console
$ jj git remote add upstream https://github.com/someone/theproject.git

$ jj git remote list
origin https://github.com/steveklabnik/theproject.git
upstream https://github.com/someone/theproject.git
```

By convention `origin` is your fork, the one you can push to, and `upstream` is
the real project. `jj` attaches no meaning to either name.

## Fetching from both

```console
$ jj git fetch --remote upstream
bookmark: main@upstream [new] untracked
```

Notice the word `untracked`. A bookmark from a remote we added by hand doesn't
become one of our local bookmarks. We get `main@upstream`, which we can refer
to, but no local `main` follows it around.

That's usually right for a fork. You don't want your local `main` chasing two
different remotes. Refer to `main@upstream` explicitly when you want it:

```console
$ jj rebase -b @ -o main@upstream
```

It also means the upstream bookmark is immutable, since the default immutable
set includes `untracked_remote_bookmarks()`. You can't accidentally rewrite the
project's history, which is a reasonable default when it isn't yours.

If you do want a local bookmark following it:

```console
$ jj bookmark track main@upstream
Started tracking 1 remote bookmarks.
```

and `jj bookmark untrack main@upstream` undoes that.

`--all-remotes` fetches from everything at once, and `jj bookmark list
--all-remotes` shows you where each name sits on each remote:

```console
$ jj bookmark list --all-remotes
main: oqtwrqnv 07effb17 (empty) local commit on main
  @git: oqtwrqnv 07effb17 (empty) local commit on main
  @origin (behind by 1 commits): quvlnmky edd53bde upstream commit 2
main@upstream: quvlnmky edd53bde upstream commit 2
my-feature: tqmvurqz 71ade80c my feature
```

## Fetching from one, pushing to the other

Typing `--remote` every time gets old, so let's set the defaults for this
repository:

```console
$ jj config set --repo git.fetch upstream
$ jj config set --repo git.push origin
```

Now a bare `jj git fetch` reads from the project and a bare `jj git push` writes
to your fork, which is the fork workflow in two settings. Because we used
`--repo`, this applies to this checkout only, and it lives in your own config
directory rather than in the repository, so there's nothing to accidentally
commit.

The rest is what we already know. We can rebase onto `main@upstream` to catch
up, push a bookmark to our fork, and open the pull request from there.
