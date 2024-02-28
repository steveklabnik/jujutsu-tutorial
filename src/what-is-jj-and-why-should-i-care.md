# What is jj and why should I care?

`jj` is the name of the CLI for [Jujutsu][jj]. Jujutsu is a DVCS, or
"distributed version control system." You may be familiar with other DVCSes,
such as `git`, and this tutorial assumes you're coming to `jj` from `git`.

The reasons I find Jujutsu interesting are varied. First of all, `jj` is
compatible with `git`. This means you can use `jj` to manage a `git` repository,
so your project can still live on GitHub, other people on your team do not have
to use `jj`. This makes it much easier to adopt than similar tools.

At the same time, `jj` is its own VCS. It works differently than `git`. I like
learning new things. It focuses around a concept called "changes," rather
than commits. The team is interested in making `jj` easy to use, which is a
big challenge in this space. It's written in Rust, so that's a plus: if I ever
want to read the internals, or contribute back, I won't have to worry. I hear
it has an `undo` commmand.

And moreover, people I know who try it keep saying I should check it out. Yes
that is a VCS joke. But it's also true.

Oh yeah, and this, from the project's README:

> I (Martin von Zweigbergk, martinvonz@google.com) started Jujutsu as a hobby
> project in late 2019, and it has evolved into my full-time project at Google,
> with several other Googlers (now) assisting development in various capacities.
> That said, this is not a Google product.

Very interesting.

[jj]: https://github.com/martinvonz/jj
