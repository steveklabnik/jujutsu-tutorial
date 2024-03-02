# Introduction

Hi there, I'm [Steve][steve]. This is a tutorial for [Jujutsu—a version control
system][jj].

This tutorial exists because of a particular quirk of mine: I love to write
tutorials about things as I learn them. This is the backstory of
[TRPL](https://doc.rust-lang.org/stable/book/), of which an ancient draft was
"[Rust for Rubyists][r4r]." You only get to look at a problem as a beginner
once, and so I think writing this stuff down is interesting. It also helps me
clarify what I'm learning to myself.

Anyway, I have been interested in version control for a long time. I feel like
I am the only person alive who doesn't mind `git`'s CLI. That's weird. I also
heard friends rave about "stacked diffs" but struggled to understand what exactly
was going on there. Nothing I read or conversations I had have clicked.

One time I was talking with [Rain](https://github.com/sunshowers) about this and
she mentioned `jj` being very cool. I looked at it but I didn't get it. I decided
to maybe just come back to this topic later. A very common thing people have when
learning Rust is, they'll try it, quit in frustration, and then come back months
later and it's easy and they don't know what they were struggling with. Of course,
some people never try again, and some who try don't get over the hump, but this
used to happen quite often, enough to be remarkable. So that's what I decided
with `jj`. I'd figure this stuff out later.

Well, recently [Chris Krycho wrote an article about jj][chris]. I liked it. I
didn't fully grok everything, but it at least felt like I could understand this
thing finally maybe. I didn't install it, but just sat with it for a while.

I don't know if that is why, but I awoke unusually early one morning. I
couldn't get back to sleep. Okay. Fine. Let's do this. I opened up [the official
jj tutorial][jj-vevo], installed `jj`, made this repository on GitHub, followed
the "cloning a git repo" instructions to pull it down, and started this mdBook.

What follows is exactly what the title says: it's to teach me `jj`. I am
publishing this just in case someone might find it interesting. I am very excited
about `jj`. I feel like I'm getting it. Maybe your brain will work like mine,
and this will be useful to you. But also, it may not. I make no claim that this
tutorial is good, complete, or even accurate.

You can find the source for this content [here][github]. Feel free to open
issues or even send me pull requests for typos and such. Zero guarantees I will
respond in a timely fashion though.

Anyway, let's get on with it.

[steve]: https://steveklabnik.com/
[jj]: https://github.com/martinvonz/jj
[r4r]: https://github.com/steveklabnik/rust_for_rubyists
[chris]: https://v5.chriskrycho.com/essays/jj-init/
[jj-vevo]: https://martinvonz.github.io/jj/v0.13.0/tutorial/
[github]: https://github.com/steveklabnik/jujutsu-tutorial
