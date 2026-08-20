# Configuring `jj`

Configuration lives in TOML files, and you get at them through `jj config`.

## Where settings live

There are four layers, and later ones win:

1. built-in defaults
2. your user config
3. the repository's config
4. `--config` on the command line

Ask `jj` where the files are rather than guessing:

```console
$ jj config path --user
/home/steve/.config/jj/config.toml

$ jj config path --repo
/home/steve/.config/jj/repos/07d925992ceecfee7f5a/config.toml
```

The repository config is worth a second look: it lives in *your* config
directory, keyed by repository, not inside the repository itself. So it's
per-repository but private to you — you can't accidentally commit it, and it
won't turn up in anyone else's checkout.

Three ways to change something. Set one key:

```console
$ jj config set --user ui.default-command log
```

Open the file in your editor:

```console
$ jj config edit --user
```

Or override for a single command, which is the right tool for trying something
out:

```console
$ jj --config ui.graph.style=ascii log
@  oqtwrqnv steve@steveklabnik.com 2024-03-25 09:14:44 main* 07effb17
|  (empty) local commit on main
+  quvlnmky steve@steveklabnik.com 2024-03-25 09:12:37 main@origin edd53bde
|  upstream commit 2
```

To see what's actually in effect, including everything you've never set:

```console
$ jj config list                       # your settings
$ jj config list --include-defaults    # and the built-in ones
$ jj config get ui.editor
```

`jj config unset --repo ui.default-command` removes a key and lets the layer
underneath show through again.

## Settings worth knowing about

### What bare `jj` does

```toml
[ui]
default-command = "log"
```

Out of the box, typing `jj` with no arguments prints the help. Most people set
this to `log` or `status` within a week.

### Editors

```toml
[ui]
editor = "nvim"  # writing commit descriptions
diff-editor = ":builtin"  # picking hunks in jj split / jj squash -i
merge-editor = "meld"  # resolving conflicts in jj resolve
diff-formatter = ["difft", "--color=always", "$left", "$right"]
```

Four different jobs, four different settings, because the right tool differs for
each. The built-in TUI we've used for interactive splits is `:builtin`, and it's
the default; `diff-editor` is how you swap in something else.

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
`jj next` and `jj prev` edit by default; `--no-edit` gets the other behaviour
when you want it. If you've settled on the edit workflow, set it and stop
typing the flag.

### Remotes

```toml
[git]
fetch = "origin"
push = "origin"
```

Which remote `jj git fetch` and `jj git push` use when you don't say. Useful on
a fork, where you fetch from `upstream` and push to `origin`.

### Keeping work off the remote

```toml
[git]
private-commits = "description(glob:'wip:*')"
```

Any commit matching that revset — and anything descended from it — is refused by
`jj git push`. A safety net for the "checkpoint" commits you make for yourself
and never mean to publish. `jj git push` already refuses commits with no
description and commits containing conflicts; this lets you add your own rule.

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

Aliases save whole command lines:

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
Anything you type after the alias is appended, so `jj l --limit 3` works.

## Revset aliases

This is the more interesting kind. Revsets are a language, and you can add words
to it:

```toml
[revset-aliases]
"mine()" = "trunk()..@"
```

```console
$ jj log -r 'mine()'
```

Now `mine()` is available anywhere a revset is, including inside other aliases
and other revset expressions. `trunk()` itself is one of these — `jj git clone`
writes `revset-aliases."trunk()" = "main@origin"` into your repository config,
which is why the built-in defaults can refer to `trunk()` without knowing what
your project calls its main branch.

The alias worth understanding is the one governing what you're allowed to
rewrite:

```toml
[revset-aliases]
"immutable_heads()" = "builtin_immutable_heads() | remote_bookmarks()"
```

The built-in set is `trunk() | tags() | untracked_remote_bookmarks()`: trunk,
tags, and remote bookmarks you don't track. Everything at or below those heads
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
"mine()" = "trunk()..@"
```

Small, but it removes several dozen keystrokes a day. Add to it when something
irritates you twice.
