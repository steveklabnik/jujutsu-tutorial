# Customizing the output of various `jj` commands with templates

Just like revsets are a functional language that allow you to describe a set of
commits, templates are a typed functional language that allows you to customize
the output of commands in `jj`.

A bunch of commands support `-T` or `--template` to allow you to customize their
output. For example, right now `@` is on an empty commit, so let's refresh
ourselves on what 

```text
$ jj log -r @-
◉  yzlysuwt steve@steveklabnik.com 2024-02-29 00:35:12.000 -06:00 main f80a73c1
│  Fill out a table of contents
```

Let's give it a template instead:

```console
$  jj log -r @- -T 'separate(" ", change_id, description.first_line())'
◉  yzlysuwtylswszxknppsyqxqoktqpqpz Fill out a table of contents
│
```

That's quite different! We've got a few things going on here:

* `separate()` is a function that takes a separator, a bunch of other templates,
  and produces the contents of those templates separated by the separator.
* `change_id` and `description` are keywords that resolve to what you'd expect.
* `first_line()` is a method on strings that returns the first line.

What if we wanted a nicer change ID? We can do that:

```console
$  jj log -r @- -T 'separate(" ", change_id.shortest(8), description.first_line())'
◉  yzlysuwt Fill out a table of contents
│
```

The `shortest` method returns the shortest prefix of an ID, and the `8` we pass
to it says "show at least eight letters even if the shortest is shorter."

Templates are powerful, and let you do a lot of interesting things. I would
suggest reading [the documentation on templates][templates] to learn all of the
details.

[templates]: https://martinvonz.github.io/jj/v0.14.0/templates/
