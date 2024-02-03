When I first made the repo, I was like "I'm not using git, so no mdbook
don't create a `.gitignore` for me."

But then the `book` directory of generated files ends up in the repo! And
since there's no staging concept in `jj`, the stuff was now in my commit.

So here's what I did:

First, I created a `.gitignore` with the contents `book`, just like usual.

I then ran `jj untrack book` to tell `jj` to stop tracking that path. Now
things look good:

```console
> jj st
Working copy changes:
A .gitignore
A book.toml
A src\01-setup.md
A src\SUMMARY.md
A src\gitignore.md
Working copy : ovruwlst 3acaf585 (no description set)
Parent commit: zzzzzzzz 00000000 (empty) (no description set)
```
