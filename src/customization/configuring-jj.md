# Configuring `jj`

`jj` stores its configuration in TOML files, and we can work with them through
`jj config`. Let's start by seeing where they live.

## Where settings live

There are four layers. Settings later in this list take precedence over earlier
ones:

1. built-in defaults
2. your user config
3. the repository's config
4. `--config` on the command line

We can ask `jj` where the files are rather than guessing:

```console
$ jj config path --user
/home/steve/.config/jj/config.toml

$ jj config path --repo
/home/steve/.config/jj/repos/07d925992ceecfee7f5a/config.toml
```

The repository config is worth a second look. It lives in *our* config
directory, keyed by repository, rather than inside the repository itself. So we
get per-repository settings that stay private to us. We can't accidentally
commit them, and they won't turn up in anyone else's checkout.

There are three ways to change a setting. We can set one key:

```console
$ jj config set --user ui.default-command log
```

Open the file in your editor:

```console
$ jj config edit --user
```

Or we can override a setting for one command, which is handy when trying
something out:

```console
$ jj --config ui.graph.style=ascii log
@  oqtwrqnv steve@steveklabnik.com 2024-03-25 09:14:44 main* 07effb17
|  (empty) local commit on main
+  quvlnmky steve@steveklabnik.com 2024-03-25 09:12:37 main@origin edd53bde
|  upstream commit 2
```

We can also ask what is currently in effect, including settings we've never
changed ourselves:

```console
$ jj config list                       # your settings
$ jj config list --include-defaults    # and the built-in ones
$ jj config get ui.editor
```

`jj config unset --repo ui.default-command` removes a key and lets the layer
underneath show through again.

`--user`, `--repo`, and `--workspace` each name a whole layer. When a layer is
spread across several files — a user config plus a `conf.d/` directory, say —
`--user` writes to the first file `jj` loads. To pick one file exactly, give
`--file <PATH>`; it works with `set`, `edit`, and `unset`, and points at any
file `jj` reads.

## Some useful settings

### What bare `jj` does

```toml
[ui]
default-command = "log"
```

Out of the box, typing `jj` with no arguments prints the help. I prefer to have
it run `log` or `status` instead.

### Editors

```toml
[ui]
editor = "nvim"  # writing commit descriptions
diff-editor = ":builtin"  # picking hunks in jj split / jj squash -i
merge-editor = "meld"  # resolving conflicts in jj resolve
diff-formatter = ["difft", "--color=always", "$left", "$right"]
```

These are four different jobs, so they have four different settings. The
built-in TUI we've used for interactive splits is `:builtin`, and it's the
default. We can change `diff-editor` if we prefer something else.

### Paging

```toml
[ui]
pager = "less -FRX"
paginate = "never"
```

`paginate = "never"` is the one to set when you're driving `jj` from a script,
or anywhere a pager would hang waiting for a keypress nobody's there to make.

### Movement

```toml
[ui.movement]
edit = true
```

Back in the edit workflow chapter we kept typing `jj next --edit`. Set this and
`jj next` and `jj prev` edit by default; `--no-edit` gets the other behavior
when you want it. If you've settled on the edit workflow, set it and stop
typing the flag.

### Remotes

```toml
[git]
fetch = "origin"
push = "origin"
```

These settings choose which remotes `jj git fetch` and `jj git push` use when we
don't name one. They're especially useful on a fork, where we may fetch from
`upstream` and push to `origin`.

### Keeping work off the remote

```toml
[git]
private-commits = "description(glob:'wip:*')"
```

Any commit matching that revset — and anything descended from it — is refused by
`jj git push`. A safety net for the "checkpoint" commits you make for yourself
and never mean to publish. `jj git push` already refuses commits with no
description and commits containing conflicts; this lets you add your own rule.

### Signing your commits

Some projects require every commit to be signed. `jj` does it for you, given a
backend and a key:

```toml
[signing]
behavior = "own"
backend = "ssh"
key = "~/.ssh/id_ed25519.pub"
```

`behavior` is the interesting setting. `own` signs commits we author whenever
`jj` writes them, which is what we usually want. The default, `keep`, only
re-signs our commits that were already signed before a rewrite. `force` signs
every commit it writes, even when somebody else authored it, while `drop`
removes a signature when a commit is rewritten. The backends are `ssh`, `gpg`,
and `gpgsm`.

`jj sign` signs a revision after the fact, and `jj unsign` removes one. To be
sure nothing unsigned escapes:

```toml
[git]
sign-on-push = true
```

Signatures aren't shown by default, since most of the time they're noise.
`ui.show-cryptographic-signatures = true` puts them in the log.

### Stale workspaces

```toml
[snapshot]
auto-update-stale = true
```

From the workspaces chapter: apply `jj workspace update-stale` automatically
instead of stopping to tell you about it.

### Large repositories

```toml
[fsmonitor]
backend = "watchman"
```

Snapshotting the working copy means checking every tracked file for changes.
On a big repository that walk gets expensive. Watchman replaces it with a query
against a filesystem watcher, and needs the `watchman` binary on your `PATH`.

## Aliases

We can use aliases to save whole command lines:

```toml
[aliases]
l = ["log", "-r", "trunk()..@"]
```

```console
$ jj l
@  oqtwrqnv steve@steveklabnik.com 2024-03-25 09:14:44 main* 07effb17
│  (empty) local commit on main
~
```

The value is an array of arguments, not a string to be parsed by a shell.
Anything we type after the alias is appended, so `jj l --limit 3` works.

## Revset aliases

Revset aliases are a little more interesting. Revsets are a language, and we
can add our own words to it:

```toml
[revset-aliases]
"stack()" = "trunk()..@"
```

```console
$ jj log -r 'stack()'
```

Now `stack()` is available anywhere we can use a revset, including inside other
aliases and revset expressions. One note on naming: `jj` will happily let an
alias shadow a built-in function — remember `mine()` from the revsets chapter —
and it won't warn you when it does, so pick names that aren't already taken.
`trunk()` itself is one of these aliases. `jj git clone`
writes `revset-aliases."trunk()" = "main@origin"` into our repository config,
which is why the built-in defaults can refer to `trunk()` without knowing what
our project calls its main branch.

One alias worth understanding governs what we're allowed to rewrite:

```toml
[revset-aliases]
"immutable_heads()" = "builtin_immutable_heads() | remote_bookmarks()"
```

The built-in set is
`trunk() | tags() | untracked_remote_bookmarks() | untracked_remote_tags()`:
trunk, tags, and the remote bookmarks and tags you don't track. Everything at or below those heads
is frozen, which is why `jj log` draws them as `◆` and why `jj rebase` won't
touch them. The line above widens it to every remote bookmark — nothing that has
reached a remote can be rewritten. Some teams want that; if you use stacked PRs,
you don't, because rewriting pushed branches is the entire workflow.

Note the `builtin_immutable_heads() |` at the front. Assign to
`immutable_heads()` without it and you *replace* the default set rather than
adding to it, which unfreezes trunk. Always union with the builtin.

## A starting point

If you want something to paste and adjust:

```toml
[user]
name = "Your Name"
email = "you@example.com"

[ui]
default-command = "log"
editor = "nvim"

[ui.movement]
edit = true

[aliases]
l = ["log", "-r", "trunk()..@"]

[revset-aliases]
"stack()" = "trunk()..@"
```

This is deliberately small, but it removes a bunch of repeated typing. I add to
it when I notice myself reaching for the same option again and again.
