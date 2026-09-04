# CLAUDE.md

An mdBook tutorial for Jujutsu. Prose lives in `src/`; `book.toml` is the config.
Built output goes to `book/` and is not committed.

## Version control

Run `jj st` at the start of each session — this may be a jj repo on one machine
and plain Git on another.

**IMPORTANT:** if this is a jj repo, manage it with jj, not Git.

**jj repo (`jj st` succeeds):** use jj commands only. Do not run state-modifying
`git` commands (`commit`, `push`, `rebase`, `reset`, `branch`, …) without explicit
permission for that command. Read-only `git log`/`git status` are fine, but prefer
`jj log`/`jj st`. Push with `jj git push` — it forces with lease natively, so no
`git push --force`.

**Plain Git (`jj st` fails):** use normal `git` commands.

Confirm `jj --version` matches the version the book documents in
`src/hello-world/how-to-install.md` (currently 0.45.1). Flag any mismatch —
behaviour and prose may diverge.

## Build and test gate

jj snapshots the working copy into `@` automatically — there is no Git-style
commit step. So the gate is finishing a change: before you move off `@` with
`jj new` or `jj commit`, or publish with `jj git push`, the book must build and
test clean:

```
mdbook build
mdbook test
```

Don't call a change done until both pass.

After finishing and pushing a change, run `jj new` to leave `@` as a fresh empty
change. This keeps `describe`d work from being confused with new edits — the
current change is always either empty or clearly in progress.

## Writing

Match the voice of the existing prose — a relaxed, first-person walkthrough.
Read nearby chapters before adding text so new sections read as part of the same
work.

Use the original prose at change ID `wtzppvkn` (`main@origin`), commit ID
`8c4b3784`, as the reference point for the book's tone. The change ID is useful
with `jj`, while the commit ID identifies the same revision through Git. Compare
new chapters directly with that revision rather than inferring the voice from
later additions.

Use `we` and `our` while walking through examples with the reader, and use `I`
for personal preferences or judgments. Introduce a concrete situation before
the command that solves it, then explain the output as the next step in the
same story. Prefer patient transitions such as "Let's see" and "You may have
noticed" over clipped, reference-manual prose. Keep examples continuous with
nearby chapters when possible.

Avoid slogans, punchy fragments, and glib comparisons with Git. Comparisons are
useful when they explain a specific difference in behavior, but the tutorial's
voice should stay curious and conversational rather than adversarial. Wrap prose
at about 80 columns, following the surrounding Markdown.

**IMPORTANT:** this is a tutorial — confirm everything against jj in practice.
Describe jj's real behaviour, not assumed behaviour: run any command in a
disposable jj repo (`jj git init /tmp/jj-scratch` or similar) and match the prose
to its actual output before documenting it. Never use this repo's own history as a
test bed.
