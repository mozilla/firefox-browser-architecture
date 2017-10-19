---
title: Browser Architecture Newsletter 4
layout: text
description: A newsletter on architecture review, XBL Conversion, Storage and Sync, Workflow Improvements and a developer survey
mailinglist: https://groups.google.com/forum/#!topic/firefox-dev/p9rTlfFUXlQ
---

# Browser Architecture Newsletter 4

[Link to mailing-list archive]({{ page.mailinglist }})

Welcome to the fourth Browser Architecture Newsletter!

A lot of the issues that the Browser Architecture team are digging into are
larger than our team. Our goal is to discuss and review entire product level
architecture issues and build consensus around solutions. We’re interested in
engaging with engineers around the organization. We’re actively reaching out to
folks but if you want to talk to us we’d sure love to hear from you. You can
find us in `#browser-arch` on IRC or Slack.

## XBL Removal

Since our last update, we've completed the design review for [a plan to move
away from XBL in Firefox](https://mozilla.github.io/firefox-browser-architecture/text/0007-xbl-design-review-packet.html).
Thanks to the long list of people who guided the creation of that plan, and the
chair and panel for putting it through its paces. We're working through some
follow-up investigations; expect a separate email with more details soon.

## Storage and Sync

We've been doing lots of writing on this topic. Emily has [documented](https://github.com/fluffyemily/cross-platform-rust) and
blogged about getting Rust libraries up and running on multiple platforms ([iOS](https://mozilla.github.io/firefox-browser-architecture/experiments/2017-09-06-rust-on-ios.html),
[Android](https://mozilla.github.io/firefox-browser-architecture/experiments/2017-09-21-rust-on-android.html)), which will pave the way for shared storage implementations across
mobile and desktop products. Richard has published the [first in a series of
blog posts about sync](https://medium.com/@rnewman/thinking-about-syncing-part-1-timelines-7f758e2bd676).

A [roadmap review](https://mozilla.github.io/firefox-browser-architecture/text/0006-architecture-review-process.html) is planned for this quarter; thanks to everyone — a dizzying
array of engineers, managers, product folks, and Deep Syncers — who've been
contributing to that process. Let rnewman know if you're not involved and would
like to be.

## Front-end Developer Survey

We sent out a survey for front-end developers to try to understand how to
target our architecture and workflow improvements. The results will be
published soon, but one thing we found was that a lack of documentation is
holding developers back. We'll be digging into this more in the future.
