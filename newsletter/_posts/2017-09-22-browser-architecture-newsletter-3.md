---
title: Browser Architecture Newsletter 3
layout: text
description: A newsletter on architecture review, XBL Conversion, Storage and Sync, Workflow Improvements and a developer survey
mailinglist: https://groups.google.com/forum/#!topic/firefox-dev/p9rTlfFUXlQ
---

# Browser Architecture Newsletter 3

[Link to mailing-list archive]({{ page.mailinglist }})

Welcome to the third Browser Architecture Newsletter! Since our [last
update](https://mozilla.github.io/firefox-browser-architecture/posts/2017-08-24-browser-architecture-newsletter-2.html),
we've continued to investigate moving away from XBL and started to
document what we're talking about when we talk about XUL problems. We're
also working with members of the Sync, AS, and Lockbox teams to figure
out what the future of storage and syncing looks like for Mozilla.

A lot of the issues that the Browser Architecture team are digging into
are larger than our team. Our goal is to discuss and review entire
product level architecture issues and build consensus around solutions.
We're interested in engaging with engineers around the organization.
We're actively reaching out to folks but if you want to talk to us we'd
sure love to hear from you. You can find us in \#browser-arch on IRC or
Slack.

## Architecture Review

We are asking for comments on our [proposed
process](https://mozilla.github.io/firefox-browser-architecture/text/0006-architecture-review-process.html)
for when our team performs architecture reviews. The process is in draft
right now so if you have suggestions for how to improve it please let us
know.

If you've got ideas, questions, or concerns, talk to
[jwalker](https://mozillians.org/en-US/u/jwalker/).

## XBL Removal

We've filed a [meta
bug](https://bugzilla.mozilla.org/show_bug.cgi?id=1397874) for our
de-XBL work, which includes a variety of prototypes and other
investigations, including: moving XBL bindings to Custom Elements,
moving XBL bindings to JS modules, and performance comparisons of XBL
bindings to a custom elements polyfill. You can view all the prototypes
and investigations we're working on by viewing the meta bug's
dependencies.

We're getting ready to run through a design review for our XBL
replacement plans. The review will be chaired by Dave Townsend and
include a panel of experts on both Gecko and Firefox. The review process
itself is a work in progress, and our XBL Removal review is a trial run
to help us refine it. Look for more information on the review itself
soon when we get further along in the process.

If you've got ideas, questions, or concerns, talk to
[bgrins](https://mozillians.org/en-US/u/bgrinstead/).

## Storage and Sync

We've [captured](https://github.com/mozilla/firefox-data-store-docs) a
bunch of knowledge about Firefox's data stores. That effort is feeding
into some roadmapping work with the Sync and Activity Stream teams. We
hope to have some concrete roadmap documentation underway by our next
newsletter, as well as some stage-setting blog posts.

We expect that the future will include some cross-platform Rust storage
services, so we've been researching how this looks in practice.
[Deploying an Rust library on
iOS](https://mozilla.github.io/firefox-browser-architecture/experiments/2017-09-06-rust-on-ios.html)
is a short tutorial describing how to build and deploy a Rust library
for use inside an iOS app.

If you've got ideas, questions, or concerns, talk to
[rnewman](https://mozillians.org/en-US/u/rnewman/) or
[fluffyemily](https://mozillians.org/en-US/u/etoop/).

## Workflow Improvements

*   [You can
    now](https://groups.google.com/d/msg/firefox-dev/L5D3YSBxqh8/pjonQLutCwAJ)
    open Browser Toolboxes on more than one instance of Firefox at the
    same time
*   The ‘no popup autohide' mode that can be toggled in the Browser
    Toolbox to inspect popups [now automatically
    resets](https://bugzilla.mozilla.org/show_bug.cgi?id=1251658) when
    the toolbox closes if you clicked the button to enable it. You can
    also flip this pref yourself with \`ui.popup.disable\_autohide\`
*   dump() is [now enabled by
    default](https://bugzilla.mozilla.org/show_bug.cgi?id=1395711) in
    local builds - kudos to :kmag for doing this

If you've got ideas, questions, or concerns, talk to
[bgrins](https://mozillians.org/en-US/u/bgrinstead/) or
[nalexander](https://mozillians.org/en-US/u/nalexander/). If you have
suggestions or pain points about doing Gecko development
(non-front-end), please contact
[jesup](https://mozillians.org/en-US/u/jesup/).

## Front-end Developer Survey

We've sent out a survey to front-end developers asking them about their
workflow and productivity issues. The results of this survey will help
us target future workflow and development improvements.

If you're an employee working on front-end Firefox code and you haven't
received the survey, please let
[mossop](https://mozillians.org/en-US/u/Mossop/) or
[nalexander](https://mozillians.org/en-US/u/nalexander/) know so we
can get it to you! We're still discussing if we should open this survey
up to contributors at this time. If you've got other ideas, questions,
or concerns, [mossop](https://mozillians.org/en-US/u/Mossop/) and
[nalexander](https://mozillians.org/en-US/u/nalexander/) are also the
right people to contact.
