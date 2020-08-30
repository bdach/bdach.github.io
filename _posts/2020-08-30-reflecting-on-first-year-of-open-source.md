---
layout: post
title: "Reflections on the first year of open source"
sub_title: "The good, the bad, and the unforeseen parts."
image:
    path: "/assets/images/lazer/hero.png"
    thumbnail: "/assets/images/lazer/thumb.png"
categories: open-source
comments: true
---

On August 31, 2019, I submitted [my first pull request](https://github.com/ppy/osu/pull/5928) to the [osu!lazer](https://github.com/ppy/osu) project.
It was supposed to be a trivial change, but I stupidly over-thought it - not sure if it was nerves, or just plain lack of experience.
After a couple of rounds of review feedback and fixing the issues pointed out, on September 3, I was overjoyed to see the change merged upstream.

The PR [was pretty much reverted the next day](https://github.com/ppy/osu/pull/5983), due to regressing parts of the code I obviously hadn't checked before.

Not the ideal way to get started, is it?

I have a very dumb philosophy of life; it's "either do it perfectly, or don't do it at all".
In retrospect, it would have been extremely on-brand for me to drop the topic then and there, and not continue pressing on as I had clearly failed the first time around, and was wasting not only my time, but the maintainers' time, as well.
But that time, for *some* reason, I refused, and tried again.
[The second effort](https://github.com/ppy/osu/pull/5979) came much easier - as I was writing the code for it, things just... clicked, and I've stuck around ever since.

Since then, over two repositories (both osu!lazer and its associated [bespoke game framework](https://github.com/ppy/osu-framework/)) I have:

- left comments in 885 issue threads,
- submitted 88 pull requests (all merged, save for one which was closed and re-submitted again, and another one which is pending review at the time of writing),
- created 367 commits in both repositories,
- reviewed 580 pull requests in both projects (467 of which were merged).

I'm bringing these numbers up not to brag; all of this was on top of an actual day job programming.
In light of this, the statistics above are clearly a sign of a diseased, restless mind.

I'm bringing these numbers up to put some perspective on them, both for me and for anyone that might be reading this - you can't succeed outright at everything.

# The what and the why

I had previously had very short-lived attempts at joining open source projects.
I would comb through "get involved in open source!" websites with vast, barren collections of code that needed more contributors.
I'd even submitted a few pulls before - but it never stuck, because I was never really interested in what *the project* was.

osu!lazer is a project that's special to me, in several respects.
It's an entirely open-sourced project to rewrite the client for [osu!](https://osu.ppy.sh/home), a free rhythm game, whose inspiration goes back [to the Nintendo DS](https://en.wikipedia.org/wiki/Osu!_Tatakae!_Ouendan), that over time [branched out into other gameplay styles](https://osu.ppy.sh/help/wiki/Game_mode) and got very popular on the Internet.
In fact, I had played it previously during my younger years (my [profile](https://osu.ppy.sh/users/3035836) dates back to July 2013) and remembered enjoying it quite a lot.

Open-sourced game projects are *very* rare to come across, as far as I can tell - on top of that, lazer uses .NET Core, which I've very much come to like and invest time into, and the codebase has been kept extremely clean through years of development, to the level that many enterprise projects could envy.

From glimpses of other commercially-released games that get posted every now and then on the Internet, I sincerely believed I would never be able to release or even be a part of a team working on a game, if those examples were to go by; thousand-line functions and globalised state are only a couple of the major sins that could be seen in those examples.
Trying to develop something like that into a workable game would make me go insane.
My brain is way to small for that sort of thing.

In comparison, seeing that osu!framework has in-built support for [visual tests](https://github.com/ppy/osu-framework/wiki/Development-and-Testing) (as a quick summary, think Selenium, but less clunky and for games), I was instantly hooked.
This is how I'd write a game, if I had infinite time and way more perseverance than I actually do.

So once I saw what the project was, I didn't really want to let it go that easily.
As time went on, I would participate in and experience several facets of the project.

# The fun stuff, or submitting code

In comparison to everything else, my code submission numbers are relatively meagre.
This was partially due to the fact that at the time of my arrival, the project was (and still is) missing people who would be willing to review code (more about that later).
My primary mission was to help the project out as a whole, and therefore I didn't really consider me writing code all that useful in comparison.

Because review efforts were limited, I set out a primary directive for each one of my submissions, and that is: **"it is my sole responsibility to provide the maintainers with everything they need to be comfortable merging"**.

On my second attempt at a PR, I devised a sort of template for a pull request, whose main goal is to explain to the maintainers *what* the change is, *why* it is needed (in case of a bug fix, for example, that would be a root cause analysis), and *why* I opted to write what I wrote in the particular way I wrote it.
All of this was to make reviewing go smoother - to help the reviewers have all of the context they need immediately in the description, without having to go look themselves or to yank it out of me.

Strangely, for a thing that I just thought about for like five minutes, it has stood the test of time, and I'd like to think it's one of the big reasons why my PRs nowadays regularly get merged quickly.
I am using this same basic template to this day, with no major alterations; funnily enough I have seen other community contributions adapt it as well, but after reading those descriptions I'm not entirely sure whether they understood the intentions behind it.

If there's one downside to this part, it's that I haven't really done much novel stuff.
I have root-caused and fixed a lot of bugs, that's for sure, but I have rarely ever broken new ground and made architectural changes or added new features.
Initially, this was intentional, as I was clearly new, but as time went on, I found that more and more of my time was consumed by...

# The less fun stuff, or reviewing code

After a while, I also picked up reviewing other changes, for a few reasons.
As mentioned before, the project is limited by review throughput, so that was an area that definitely needed help.
In addition, reviewing code had the nice side-effect of letting me see more and more of the code in the project, and the structure of osu!framework.
Reading code "just to read code" is not a philosophy I'd ever go for - it feels a bit aimless.
It's not like a book - it isn't intrinsically interesting to read for me without the larger context of a functionality or a goal to achieve.

The unfortunate thing about the organisational structure of lazer is that at current there are two members of the core team working on the game full-time, and everything else comes from community contributions.
The bad part is that a significant number of those contributions comes from obviously well-meaning, but not necessarily overly experienced programmers.
It might sound hypocritical to say this as a person with barely two years of professional experience (even if I am a lunatic workaholic), but I could clearly see shortcomings in the code submissions that the submitters themselves failed to spot.

At the start, this was not much of a problem; I would go out of my way to explain the issues in the code, and the processes and ways in which the project did things.
This was probably the key part of my contributions, which by February of this year got me an invite to be a collaborator on the project.
However, at some point, explaining the same things over and over multiple times, started getting tiring and old.

The truth about software maintainership, which I guess reviewing code is a part of, is that you *need* to be the person to say "no" a lot.
It is your job to object to each shortcut taken, to each unseen bug, to each spotted shortcoming, for the good of the project.

Contributors don't really like you for it, and it's not a very healthy thing to be doing all of the time, as it wears you down and makes you cynical, which is especially bad given it's a really asymmetric relationship.
You might interact with 50 PRs a month, but the average contributor submits maybe one or two of them, so being snippy makes you just seem like an unpleasant person in their eyes.

Two months ago, I set out to try and make this better.
[I wrote up a document with contributing guidelines](https://github.com/ppy/osu/pull/9188) to help new people get involved without making the same mistakes over and over again.

In my view, this has spectacularly failed to work.
Several new people have come along since then, and most of those people have repeated the mistakes explicitly listed in the contributing guidelines.
Admittedly that fact has worsened my feelings of cynicism, and I'm not sure whether they'll ever go away.

I don't know whether this can be fixed, but I do know that this can't continue forever.
"Open source should be accessible to all", is the big idea - but is that really a feasible thing to achieve, given that maintainership is a thankless task done by the few, whose resources are additionally usually limited by wanting to do their own things?

# The unfun stuff, or issue triage

In terms of "toll taken on patience", code review is nothing in comparison to handling and triaging issues.
Despite enforcing issue templates, a significant number of issues filed is initially missing the necessary details or does not have enough information to be acted upon.

It is very hard to keep a calm head after

- handling the thirty-fifth issue of the month that mentions unspecific "lag," without listing hardware specifications, attaching a video of the behaviour or even screenshot of the (built-in!) FPS counter or performance statistics, or the slightest context of when the performance drop happens,
- reading a tenth duplicate of an existing issue,
- reading an issue about a problem with a particularly arcane hardware setup that maybe less than 0.1% of end users will be using,
- reading an issue with only the title filled out and an entirely blank issue template,
- reading a comment on a 2-year-old issue marked as low priority trying to bump it up while the game isn't yet entirely functionally complete,
- reading a comment proposing a code change without actually having written said code or attempted to use it in the context the proposal applies to (what happened to the DIY spirit of open source?),
- reading a comment that guesstimates what the cause of a bug might be, clearly not having even tried to read the code in question,
- reading an issue with a "fun" emoji in the title because the issue number is round.

While the list above may be a little hyperbolic (although individual events on it have actually happened), the sad reality is that issue trackers of public-facing end user applications are a source of almost constant noise.
Competent and full feature requests and bug reports are by far the minority, and are outweighed by non-specific complaints that take up time and mental energy to process.
Users seem unaware that to be actionable, an issue has to be replicable - we're not doing magic here; each issue has to be explicitly investigated.

This is again something that I don't have an answer to.
Increasing the barrier-to-entry for issue submissions seems like a wrong solution, but a couple of attempts to make users aware of what they should be doing to make handling issues easier by fiddling with the issue templates have yielded very little.

# The question of focus and direction

Looking back, while I don't regret any of the time I've spent on the project, I do wonder if I have misallocated the time I put into it on things that aren't really necessary.
Over time I have been shifting my focus toward fixing major regressions and adding features necessary for the game's release, but I still have the distinct feeling of having done very little for how much time I've put in.

Somewhere in the last two weeks, which I've had off work, I decided to hold off on reviewing low priority pull requests, to take some time off from reviewing and to try and catch a better focus on the priorities going forward.
I'm not sure whether that will be temporary or long-lasting, but I want to see how much of an impact prioritising in that manner will have.
However, I'm afraid that in the long run I might have to go back to reviewing whether I like it or not, as not many people seem to both want to and have ample time to help out in the process of code review.

# Open source isn't a silver bullet

In the past, I have overly idealised open source as this perfect meritocracy, wherein anybody can pick up the code and do with it whatever they want to.
It was clearly a failed premise, but this year has shown that explicitly to me.
What is gained by lowering the barrier of entry to contribute, is offset by figuring out how to help the contributors attain the level of competency which the project actually requires.

This is by no means universal to this one project; as a contrasting example, the Linux kernel has an insane barrier of entry.
Partially it's justified, as the kernel is nothing to mess with, and [issues and pull requests on the kernel's code mirror hosted on GitHub](https://github.com/torvalds/linux/pulls) shows that if the kernel ever *were* to move to it, the maintainers would likely be absolutely be swamped by a torrent of absolute noise.
On the other side, I imagine that finding appropriate successors as the current maintainers come and go might become an increasingly troublesome task as time goes on.

On some level, it does feel a little unjust to give little time to contributions that aren't necessarily of the highest quality.
On my personal scale, at some point, it inevitably boils down to a time management problem: is my time better spent trying to advance the project forward toward a release on my own, or to train others to be able to assist in that process?

# The money talk

The osu! ecosystem in a push for open source has established a [bounty program](https://docs.google.com/spreadsheets/d/1jNXfj_S3Pb5PErA-czDdC9DUu4IgUbe1Lt8E7CYUJuE/view?&rm=minimal#gid=523803337) to compensate contributors for their work.
I have never sent in a bounty request for anything I've done, as I'm doing well for myself in my day job and don't need more resources than I already have.
I save plenty as-is and don't want to be hogging resources that others could need more.

As much as I have considered using the bounty program to do open source full time, I also did the math, and it is ruthless.
At the rates usually awarded, it would still be above average wage in Poland, but I would still be taking a pay cut to do it compared to my current compensation, and that's pre-tax and insurance/retirement fund payments, which I would also have to be handling myself entirely.
I wouldn't really want to ask for more; I don't really think that my time is inherently worth more or less than other contributors'.

Doing this would mean taking on more risk, and potentially limiting my possibilities to buy a flat or a house, which I do want to do at some point, instead of renting and living on essentially borrowed ground for my entire life.
I am aware that a house means selling my soul for over a decade to a bank, too.
It's just a sad reality of our current economical system - for now, I'm putting the thought of doing this as a day job away as a pipe dream.

# The end notes

While the future is very much uncertain *for reasons that shall remain unmentioned*, the last year has been a blast and I still want to put as much of my free time into lazer development as I can feasibly give.
The technical results of the project and its foundations, laid down years ago now by people smarter than me, are all incredibly impressive, and the future plans for it, which I haven't even scraped the surface of in this post, are very ambitious.
On some level I wish I'd found the project earlier (although I'm not sure I would have finished my MSc degree if I had).

I would like to thank the osu!lazer team of [@peppy](https://github.com/peppy/) and [@smoogipoo](https://github.com/smoogipoo/) for all the help, and for the trust placed in me in granting me the collaborator role.

Huge shout-outs to other regular contributors I've interacted with throughout the year: [@swoolcock](https://github.com/swoolcock/), [@frenzibyte](https://github.com/frenzibyte/). [@jorolf](https://github.com/jorolf/), [@Joehuu](https://github.com/Joehuu) - you're all doing great work.

Into another year!
