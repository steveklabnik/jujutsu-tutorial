# Real-world workflows

We can use `jj` at this point, but I wouldn't say our workflow is great. In this
chapter, we're going to explore two different workflows that are popular with
`jj` users. Like `git`, `jj` is extremely flexible, and so you can customize
your workflow in many ways, but I'd like to show you two examples. For me,
understanding the basics we've talked about wasn't tough, but when I actually
sat down to *use* `jj`, I found myself tripping up a bit. With a few more
commands, we can have nicer workflows that let us work a bit more naturally.

We'll start with the "squash workflow," as it is the workflow that Martin, the
creator of `jj`, prefers. We'll then talk about the "edit workflow," which is
popular among people who don't like the squash workflow. Finally, we'll cover
three commands you'll want whichever workflow you settle on: `jj split`, which pulls
one change apart into two, `jj absorb`, which files a pile of small fixes back
into the commits they belong to, and `jj diffedit`, which reaches into a single
commit and fixes what it contains.
