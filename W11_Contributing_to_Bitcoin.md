# Contributing to Bitcoin

This is simply the story of my first contributions to Bitcoin, and what I learned from them.
Written during the last week of my BTrust Fellowship of 2026, It's also a look back on this whole year for me.

## Before Contributing: learning!

Prior to this year, I had been more of a 'user' of the Bitcoin ecosystem. My contributions to Bitcoin tools were limited to submitting fixes to messages or documentation. Or creating issues on the projects Github pages, but for submitting my problems, never for submitting a fix.
I had already developped tools around Bitcoin ([analysing the Lightnin Network's graph to find which nodes to open channels to](https://lnshortcut.xyz)), but it was more tools for me to help use existing applications, not contributions to the applications themselves.

But this year, I took a step forward. I wanted to actually learn how it all works underneath.
It all started at the end of last year, when a SN post annnounced the opening of Boss Challenge at the start of 2026.
To enter, you have to first complete [Saving Satoshi game](https://savingsatoshi.com/). Even if just a game, I already learnt a lot more about various aspects of Bitcoin.<br>
Finishing it gave me access to ChainCode Labs' BOSS Challenge, where, through resources and exercises, I doved deep into the tech of Taproot and Lighting Network for example.
At some point, the Challenge openned doors to continuing with partner programs.<br>
I then joined a Bitshala cohort on the Lightning Network, where we studied all chapters of the Mastering Lightning Network book.<br>
I followed by following a program from BTrust on programming in C++ for Bitcoin.

And after getting high grades in this one, I was offered to take part of BTrust's 2026 Fellowship, which aims to teach people how to become contributors in the Bitcoin ecosystem. This is a 3 month program, where the 2 first months were again concentrated on Learning, and researching a particular subject, which for me, was Silent Payments.
Then came the 3rd month, finally, about contributing!

## Time to show Proof of Work

### Selecting a project

I had done some early searches before starting the Fellowship, for a good project to work on. I'm interested in Lightning node management tools, and am mostly programming in C++.<br>
So Ended up finding CLBOSS, which is kind of an autopilot plugin for CLN nodes. I got really interested in the project, and started attending some of their weekly meetings. But the project was probably too small. It is now in th elist pof projects I would like to contribute to, but only after the Fellowship is ended.

I naturally went towards the most known, and biggest C++ proojecft related to Bitcoin : Bitcoin Core.

### What I worked on

Bitcoin Core is a big project, touching various aspects, and using multiple implementation technologies. It was not obvious how to find what to work on at first. I used multiple approaches:

#### A General C++ code review

In order to start getting familiar with the code, I started by just reviewing the C++ code, as I would have reviewed a PR at my job. When you don't fully master the feature, you can at least check for some common C++ errors / innefficiencies.

But this turned out to be less fruitfull than I expected : the code was actually pretty clean. Which is good news for such a critical project!

This at least led me to find material for my first PR: [PR #36501](https://github.com/bitcoin/bitcoin/pull/36501).

The idea here is that the type CBlockIndex is composed of various members, of different sizes. Due to structure member alinment rules, sometimes padding bytes have to be added between members of the structure, so members always respect a certain alignment.

By reordering the members, you can sometimes reduce that loss of bytes, and reduce the size needed for an instance of that type.

The interesting thing is that this particular type is generally instantiated once per block in the chain. So at current block height (~976k), that mean we can potentially reduce memory usage of most Bitcoin nodes by 8 MB without doing anything else!

#### Reviewing areas I Know

At some point, I needed to go deeper than just checking 'code quality'. In order to enter more in the codebase, I starrted looking areas of the code related to functions I know well enough.

The obvious entry point was the subject I had picked for the research presentation of the fellowship : Silent Payments.

I had dug deep into the formulas behing Silent Payments, and had had to lookup the current state of the implementation in Bitcoin Core.

One potential PR candidate was the BIP 352 unit test. Which in my opinion did not test one explicit thing. But in the end, a more generic test was also testing that case.

An issue was also published on Github, by one of the devs working on the implementation of BIP 352 : [Issue #36605](https://github.com/bitcoin/bitcoin/issues/36605).<br>
I really spent time on this one. Trying to understand the problem, trying to see what is needed,..
The problem was that this is a real problem, but even deciding what should be the default behaviour is tricky. So for now, there still needs discution before any work can be started. So this one is still in my watch list, but I kepyt looking elsewhere

I kept digging around BIP352 code. Stumbled upon the changes to Bech32 for Silent Payments. Indeed, for Silent Payments, where the silent payment address is encodes in Bech32m, the maximum supported bech string length for them increases from 90 to 1023 bytes. So I looked at the Bech32 unit tests, to see if they had updates to test for this change.<br>
I found out that there were no new tests for the new bech32 length limit. I actually also found out that we did not really test the 'guarantees' of error detection count of bech32 advertised in the code.<br>
So this lead to my second PR : [PR #36690](https://github.com/bitcoin/bitcoin/pull/36690)

Finally, the knowledge I acquired on Silent Payments also allowed me to provide review on another existing PR related to them. 

#### Simply using the project

Finally, there is a limit to what you can find to improve by just reading the code. Another way you find things you could work on is.. to ise the project!

For me, it was not tecnically while using it, but simply by keeping exploring the code that I found another thing that would gain from a modification:

At some point after a git pull in order to test some branch, then my project stopped building.<br>
After a few hours of debugging, I ended up finding the issue : the PR removing support for Visual Studio's default compiler had been merged. And it did not only 'stop supporting' the compiler. Tee changes is now assuming that if you are building for MSVC, you WILL use clang-cl. The problem is, cmake still lets you build a MSVC solution for cl by default!<br>
So my 3rd PR was simple, but hopefully save those hours of debugging to other windows developpers : adding a fatal error when cmake is called if the target is MSVC and the selected compiler is cl.exe ( [PR #36839](https://github.com/bitcoin/bitcoin/pull/36839) )


### What Feedback i received

During this work, I received various kinds of feedback, which gave me multiple point of views on contributing.

#### Respecting the review procedures

Very early during the fellowship, I tried to review a pull request. I just provided some code comments with brief sentences, on minor remarks. Unfortunately,  this, probably associated with my lack of hostory in the repo until now, made some algorithm identify me as a bot/AI, and made me get banned from the Bitcoin repository. Forntunately, this got sorted out by contacting a maintainer, but this hard introduction to respecting the rules anchored how they are important. 

#### Proving what my PR brings

Another kind of feedback I got was that my PR submission needed to assess properly what it improves.<br>
In the case of PR #36501 for example, I had claimed that memory usage of all nodes would be reduced by 8MB without doing anything. This is because I had simply multiplied the size reduction per object by the number of those object that have to be in memory of a node.<br>
But I had not considered allocation constraints of the container of those object. In that case, a std::map; which, on Linux platforms, overallocates more memory than what is needed per node of in the map. reducing thae type size by 8 bytes did not reduce the memory per node of the map that was allocated.<br>
In the end, I had to provide some additional tests to at least assert that the gain was present on other platforms. Windows for example.

#### Assessing the risks of a change

Another feedback was that my risk assessments were not complete. It was not that there was an actual risk in the end. It was more the fact that I had claimed there was no risk, but that maintainers clearly saw potential issues, and that my PR submission did not clarify that I hade checked them. Everyone's time is precious, so it's up to the PR author to prepare the review work of the maintainers.

#### Lack of feedback, but my idea gets reused

This one, I'm still a little bitter about. One of my PRs, which added some tests around bech32, did not recceive any feedback in the Bitcoin Core project. After 3 weeks of being submitted, there has not even been a comment saying the PR is not interesting.
One lesson from this is that it's not because you find what you think is an issue, and even find, then prepare a fix for submission, that there are people that have available time and interest in that PR to review it.

But it did not really stop there for that PR. Actually, within 1 day of that PR having been open for review, another project had already used AI to apply the same principle and to add the same kind of test to their project.<br>
It kind of proved the interest of what my contribution in a certain way. But something in the way the idea was taken by another project, and in exchange I didn't even get a review or feedback in the original PR, hurt a little. But that's the price for making your work public I guess. You can't control how the world will use it.

### What i would do differently

I think two things particularly frustrated me during this work, that maybe I exagerated because of wrong expectations.

#### Finding what to work on

I spent a lot of time browsing existing issues of the Bitcoin Core project.. and not finding which one to claim. There were multiple reasons, among which:
- the issue is a user submission of theyr problem, but not an actual problem
- The issue already has open PRs
- The issue still needs discussion before work can start
- The issue is too advanced for what I know right now

So between not findinng issues, or waiting for feedback, my inability to 'claim issues' to work on took a lot of stress for nothing.

In the end, by looking at parts of the code you know, trying to reproduce issues, reviewing ther people's work, you will naturally find issues to submit, or things you can work on!

#### The objective shouldn't be the number of merged PRs

The other frustrating part was the speed (or lack of) to get a PR merged.
I was one of the first in the fellowship to open PRs. But then they stagnated and others got theirs merged first.
Which then introduced stress of not reaching the Fellowship goals.

While in the end, I have submitted 3 additions to this big project. One is approved, the other at least has ACKs, and the last, even if ignored in this project, at least received some 'acknowledgment' by being copied by another project.
I also spent time on the other side, reviewing other people's work, and helping it getting merged or improved.

And that's also part of contributing!

## What now?

Writing this article will probably be one of the last things I do during the Fellowship, since we are just a few days from the end of the last week when this is written.

Over this last month, I finally started contributing to the Bitcoin ecosystem, as a conclusion for a full year of studying. Now, I have one project I already started contrivbuting to, and another project I also know I want to work on in the future. During this year, I also matured an idea of my own project in this ecosystem, for which I can't wait to have more time to allocate to.

This is just the beginning!