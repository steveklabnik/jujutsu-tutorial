# What is jj and why should I care?

`jj` is the name of the CLI for [Jujutsu][jj]. Jujutsu is a DVCS, or
"distributed version control system." You may be familiar with other DVCSes,
such as `git`, and this tutorial assumes you're coming to `jj` from `git`.

So why should you care about `jj`? Well, it has a property that's pretty rare
in the world of programming: it is both *simpler* and *easier* than `git`, but
at the same time, it is *more powerful*. This is a pretty huge claim! We're
often taught, correctly, that there exist tradeoffs when we make choices. And
"powerful but complex" is a very common tradeoff. That power has been worth it,
and so people flocked to `git` over its predecessors.

What `jj` manages to do is create a DVCS that takes the best of `git`, the best
of Mercurial (`hg`), and synthesize that into something new, yet strangely
familiar. In doing so, it's managed to have a smaller number of essential tools,
but also make them more powerful, because they work together in a cleaner way.
Furthermore, more advanced `jj` usage can give you additional powerful tools in
your VCS sandbox that are very difficult with `git`.

I know that sounds like a huge claim, but I believe that the rest of this
tutorial will show you why.

There's one other reason you should be interested in giving `jj` a try: it has
a `git` compatible backend, and so you can use `jj` on your own, without requiring anyone
else you're working with to convert too. This means that there's no real
downside to giving it a shot; if it's not for you, you're not giving up all of
the history you wrote with it, and can go right back to `git` with no issues.

[jj]: https://github.com/martinvonz/jj
