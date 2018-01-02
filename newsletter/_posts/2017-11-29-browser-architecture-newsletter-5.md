---
title: Browser Architecture Newsletter 5
layout: text
description: A newsletter on Storage and Sync, XBL Removal and Workflow Improvements
mailinglist: https://groups.google.com/forum/#!topic/firefox-dev/XKp3EthdJ60
---

# Browser Architecture Newsletter 5

[Link to mailing-list archive]({{ page.mailinglist }})

Welcome to the exciting fifth installment in our series of Browser Architecture Newsletters!

## Storage and Sync

We are pleased to report that the Sync and Storage roadmap proposal successfully passed review by dcamp. Thanks to the 20+ people from around the organization who helped us build consensus.

The roadmap establishes that storage and syncing of user data is a pillar of the Firefox ecosystem, warranting holistic and long-term attention, and outlines both where we’d like to end up and some ideas for how to get there.

We expect this roadmap to lead to a number of subsequent concrete design reviews, as we begin with prototyping, further competitive analysis, and more coordination across Mozilla. The roadmap doc itself has been lightly cleaned up and [published](https://mozilla.github.io/firefox-browser-architecture/text/0008-sync-and-storage-review-packet.html) alongside [our other docs](https://mozilla.github.io/firefox-browser-architecture/).

If you’re working on improving storage or syncing of data in any of our products or projects, on any platform, or if you’re currently struggling to work with what currently exists, then we’d like to hear from you and align our efforts.

You can find us in Slack or IRC (#browser-arch), or come find rnewman, etoop, rfkelly, markh, or adavis.

We’ll be meeting to talk in more detail about the roadmap, the present, and the future [at the All Hands](https://austinyallhands2017.sched.com/event/CyOK/sync-and-storage). We would welcome your participation.

## XBL Removal

Our work to remove XBL is proceeding apace. We’ve started by removing bindings that are never or only rarely used. Work to convert XBL bindings to custom elements is currently stalled while we get custom elements working in XUL.

You can track how this project is progressing with some [lovely graphs](https://bgrins.github.io/xbl-analysis/graph/). If you’d like to help with the effort, you can grab a bug tagged with [[xbl-available]](https://mzl.la/2AyTxCG).

## Workflow Improvements

Running mochitest-plain in headless mode is now supported on Windows, MacOS, and Linux. Use the “--headless” flag with “./mach test” to try it out.

## Austin All Hands

We have two meetings planned for the All Hands. You can see these in Sched here:

* [Austin Y'all Hands 2017: Sync and Storage](https://austinyallhands2017.sched.com/event/CyOK/sync-and-storage)
* [Austin Y'all Hands 2017: XBL and XUL Removal](https://austinyallhands2017.sched.com/event/CyLC/xbl-and-xul-removal)
