# Typing less at the CLI

`jj` has a bunch of ways to not type so much. For tutorials I like to use the
long names, but you may want to take advantage of some of the aliases that
exist. For example:

```console
> jj checkout main
> jj co main
```

Same thing. But one of the coolest shortcuts is hard to represent here, because
I am too lazy to take screenshots. But let's say you have CLI output like
this:

```console
> jj log
@  tqxkxrro steve@steveklabnik.com 2024-02-03 12:20:07.000 -06:00 1fec9fde
│  front matter, references, etc
◉  voznqwxp steve@steveklabnik.com 2024-02-03 11:32:57.000 -06:00 main 0262ab43
│  updating working with github
~
```

You can't see it here, but the first character in all of the various IDs are
highlighted: the `t` in `tqxkxrro`, or the `0` in `0262ab43`. This is something
shared with git, but the output of the `jj` CLI makes it so much more obvious:
you only need to share the amount of the ID that's unique in the repository to
commands. So these are all equivalent:

```console
> jj checkout main
> jj co main
> jj co 0262ab43
> jj co 0
```

If there's another ID starting with `0` in the repository, it would show `02` in
a different color instead of only `0`, so it feels very obvious and safe to
use the shorter ID. I know with `git` sometimes I haven't always felt sure I
wasn't going to accidently use too few characters, so I rarely do this there.
I'm sure `git` would probably tell me that the ref is ambiguous, but I'd just
rather not stress it.
