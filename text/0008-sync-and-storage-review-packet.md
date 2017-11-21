---
title: Roadmap Review: Sync and Storage
layout: text
---

# Roadmap Review: Sync and Storage

**Firefox should wow users with a smart personalized experience that's available seamlessly across services, apps, and devices. User data storage and syncing are vital to this vision.**

We propose to reframe user profile data as a pillar of the Firefox ecosystem, rather than continuing to allow our storage and syncing story to emerge piecemeal from the implementation details of each new product feature.

User data becomes a deliberately designed service layer on which many different product experiences are built. We foster through design reviews the long-term importance of user data. And we invest technically in a more unified, extensible, flexible, cross-platform suite of storage and sync solutions that support a coherent set of approaches to storage.

## Review question

Is this roadmap proposal congruent with Firefox's strategy?

## Document status

* 2017-11-20: Go-ahead given.
* 2017-11-09: Review successfully completed. Follow-up questions underway.
* 2017-11-03: Sufficient contributions received; sent to the Reviewer.

## Roles

* **Reviewer**: dcamp
* **Chair**: jwalker
* **Proposers**: 
    * rnewman, Browser Architecture
    * rfkelly, Engineering Lead, Firefox Accounts
    * markh, Sync engineering
    * emily, Browser Architecture
    * adavis, Product Manager, Sync & FxA

# Lay Summary

Firefox, and the other new experiences that we want to create, rely on easily recording, combining, syncing, extending, and repurposing user data.

Desktop Firefox’s data is spread across [45+ different stores using 10+ different storage technologies](https://github.com/mozilla/firefox-data-store-docs), and we keep adding more.

In general, these stores and technologies:

* Were not designed for syncing, for reuse on other platforms, or for their data to be repurposed.
* Are not straightforward to extend, particularly those connected to Firefox Sync. Migrations are risky and time-consuming, and [downgrades are a huge concern](https://bugzilla.mozilla.org/show_bug.cgi?id=1404344).
* Are silos: it’s hard to connect them to each other to provide insights into data, which is the motivation for initiatives such as Activity Stream and Context Graph.
* Have no baseline of expected storage capabilities: documentation, full-text search, backup/import/export, basic versioning/migration, asynchrony, atomicity, caching, performance measurements and optimizations, _etc._ are all built from scratch, if they are built at all. Each store repeats the same fundamental mistakes and has its own quirks.
* Provide inconsistent support for security and privacy UI settings, leading to an [inconsistent and non-intuitive user experience](https://docs.google.com/document/d/1QJxHQ4GqziUGUiyaF5oP7-cA3hkyTamY33VvHY3vBrM).
* Are limited to a single platform. Desktop’s storage investments (_e.g._, Places, form history, session store), are hard to use elsewhere in the ecosystem, which reduces the value of those investments. No code or storage formats at all are shared between our iOS and NMX projects and desktop; even Fennec reimplements (for good reason, but at some cost) stores that exist in Gecko.

Firefox Sync itself is incomplete, in every sense:

* It doesn’t sync enough data types.
* Types that sync aren't complete, and/or lose data when synced.
* We have almost no consistency across platforms: each implementation syncs different data, in different ways, to different storage.
* Access control is an all-or-nothing affair; the auth tokens and encryption keys needed to read synced bookmarks will also grant read/write access to synced passwords.
* It is hard to improve the protocol itself to remedy these.

Our current technologies are ill-suited to even our current sync and extensibility needs, let alone our predicted future needs.

Moreover, we have no off-the-shelf solution to adopt for new products or features that want to avoid these issues: the status quo is maintained. Teams either do significant amounts of redundant work to bootstrap new storage for each feature and for each platform, to add sync support, and to reach a baseline level of functionality, or they neglect platforms and functionality.

These problems are already limiting factors to product development. Recent examples:

* Form autofill is shipping desktop-only. Desktop-only sync took two months of work and an entire work week to build. Android will, at some point, blindly round-trip this data but not use it. There are no plans for iOS work.
* AS found it extremely challenging to add data to Places, and adding even one additional field to Firefox Sync took 9+ months to trickle through three platforms and negotiate backwards compatibility constraints. More ambitious data recording, particularly for speculative features/experiments, is considered unreasonable, time-wise, within the framework of Firefox Sync: time-to-market is too long.
* The nascent support for containers in Firefox — partitioning of history visits into work, personal, _etc_. — has no clear path to integration with Sync.
* Few features are added to Sync itself: they need to be implemented three times (once for each major platform) and significant changes run into backward compatibility issues.
* New mobile and other experiences that want to integrate with FxA-stored data face significant data/sync costs:
    * The NMX team forked a [separate simple read-only Android library](https://github.com/mozilla-mobile/FirefoxData-android), and [an unfinished iOS equivalent](https://github.com/liuche/FirefoxAccounts-ios), to pull data directly from the Sync server; Sync is complex, and extracting the existing Firefox code into separate libraries required expertise that was not present in the team. (We now have **five** separate FxA and Sync implementations.)
    * Firefox Rocket doesn't reuse Firefox for Android's storage, and has no path forward to syncing.
    * A Servo-based Android AR/VR browser is in need of data storage solutions, but we don't have a solution that we can give them.
    * New mobile and other experiences that generate their own user data have no established patterns or guidelines for storing it, and no existing path toward making that data available to the broader Firefox ecosystem.
    * The Lockbox project burnt a lot of early development cycles trying to decide what to use for backend storage — Sync, Kinto, or something else entirely. Their MVP currently does purely local storage.
    * Project Hopscotch will store its collected user data in Google's [Firebase](https://firebase.google.com/), where it is siloed away and unable to be repurposed or shared with other Firefox experiences.

Stakeholders are:

* Sync, both client and server, across all platforms. The Sync team is motivated to improve Sync and to build better systems, but is only staffed for incremental work, and has neither time to assist with more than a handful of one-off integrations (_e.g._, form autofill) nor significant leverage to direct the design of storage systems to reduce that workload.
* Activity Stream, across all platforms. AS is a significant short-term new consumer of user data, and a long-term generator of reusable data. Delivering a good AS experience requires capturing new data and going far beyond the current capabilities of Sync and Places, but the team lacks the leverage or expertise to make those changes.
* Existing storage teams, responsible for maintaining and implementing existing stores. Some stores lack owners. Most of these engineers don't directly experience the costs borne by the Sync team and engineers working on other platforms.
* Existing product teams. Product managers own features across platforms, but the implementations and capabilities of those features differ widely.
* New product teams and ET explorations wishing to use and collect user data. These teams must build storage and syncing from scratch. Extending and integrating with existing data is challenging.

## Brief

Altering the course of 10+ years will be a multi-step process.

_We propose expending effort to build replacement tools and systems that meet current and known future needs, supporting in-development Firefox features, and migrating existing Firefox data stores as it makes sense to do so._

Rather than just building single-platform MVPs, we hope to encourage engineering and product managers to consider the total cost of cross-platform features and integration, and shift incentives to encourage engineers and product managers to drive towards consolidation — make the right thing easy, so that features get syncing, backup, extensibility, and fast querying without significant new work.

Mozilla needs a _culture that places importance on user data_ — more forward-looking data modeling, implementations that are built to sync across platforms, and data that can be repurposed. One path to doing so will be the architecture review process itself.

We expect technical work to include **designing and building a durable and scalable end-to-end sync and storage system for user data.** This system should handle evolving data without the involvement of Sync engineers, without locking out clients on other platforms or releases, and without losing server data. [Log-structured data](https://engineering.linkedin.com/distributed-systems/log-what-every-software-engineer-should-know-about-real-time-datas-unifying) is the industry standard for distributed systems with growing data and multiple consumers, so we expect this to be a key part of the solution.

We've [captured some detailed requirements for sync and storage](https://docs.google.com/document/d/1XOJhW-qf4iIyGHwiSLgWEfXYDWLy44So5L8CdwH6pcg), and will continue to do so.

This is a relatively large piece of work. Mozilla’s needs are fairly unique: _e.g._, client-side crypto, disconnected writes, years of activity data stored on the client, and working without an account. This means that although the conceptual framework has been well understood for many years, off-the-shelf solutions tend not to be suited to our needs. (Chrome Sync itself has a similar implementation to Firefox for iOS or 2017/2018 desktop Firefox, and is presumably staffed to match the costs of this high-overhead approach. Google defaults to non-private approaches.)

Getting this right unlocks the future of user data.

Technical work will involve:

* Designing client-side APIs to encourage features to record data in a way that meets the needs of other components, including Sync.
* Designing one or more storage systems (including client, server, and documented protocol), supporting those APIs, that meets the captured requirements for sync and storage, including straightforward evolution of data.
* Building SDKs and other tooling to help new mobile experiences quickly integrate with existing user data, and store and share new user data of their own.
* Building client-side data pipelines to also meet the needs of application features. New storage systems must meet the performance requirements of the product.
* Exploring technologies and patterns that make it easier to connect UIs through to storage.

The Sync and BA teams have already begun exploring prototyping parts of this puzzle in an incremental way: in Q4, building a proof of concept as an example of an event observation and capturing infrastructure, materializing a view for querying. We expect this POC to inform our implementation plan.

We also expect to build cross-platform **storage and sync implementations partly or wholly in Rust**, to build a future in which products **throughout the Firefox ecosystem** can share fast, reliable code instead of reinventing the wheel. With current staffing levels we have a hard time keeping our multiple codebases interoperable, let alone growing features at a healthy rate. Emily has [already made progress](https://github.com/fluffyemily/cross-platform-rust) on the groundwork for cross-platform Rust libraries.

We will **continue work on documenting Firefox’s user data and publishing** [**our findings**](https://github.com/mozilla/firefox-data-store-docs). In the course of preparing this proposal we discovered that essentially none of our storage systems are documented, sometimes to the point of not knowing what a file or database was for, and often not being able to tell whether a file was obsolete. We don’t even know which prefs will exist in about:config. It is hard to make good decisions without information.

We will **continue to help guide new data-centric systems** — including Lockbox and ET work like Foxy — towards a cooperative future that builds capabilities for our ecosystem strategy.

And we will guide the **incremental consolidation of existing Firefox data stores into more capable systems**. This might begin by building or buying replacements or abstractions, so the number of stores will grow before it shrinks. For example, libpref is ill-suited for storing data for front-end features; we should provide a durable, syncable store that meets their needs, and migrate features away from libpref, ultimately allowing it to be replaced by a simpler and more performant system that better meets its “pref-shaped” needs. It’s not yet reasonable to pin a number on how many stores we _should_ have, but it’s almost certainly less than 45.

Some of this work will involve measurement work on existing systems or product features; Firefox currently lacks a holistic view of its storage footprint and performance characteristics. We expect to work with product management and engineering to improve this.

We expect initial staffing for these efforts to be low. Research and prototyping work (much of it not parallelizable) needs to be done for new storage, more exploration of Firefox needs to be documented, and the groundwork for Rust components needs to be completed. 3–4 FTEs is appropriate.

In 3–9 months we should be in a position to evaluate a number of more concrete directions, gradually committing more headcount to accelerate progress as opportunities become actionable.

We can call out some representative product end states of this effort:

* New mobile apps can import a published library, with **shared code across platforms**, and use a standard FxA OAuth flow to get access to all or part of a user's synced data **without reinventing the wheel**.
* New product features and **experiments** can store new data through a simple, homogeneous API, and have that data automatically accessible from other Firefoxes and in mobile apps, **without writing significant new syncing code** and without worrying about version compatibility. Product managers should feel that they can confidently explore and iterate on product experiences.
* Users can alternate between release channels or move through a series of experiments using the same profile without losing data. **Most changes will be downgrade-safe**.

Effort subsequently invested in consolidation should gradually reduce ongoing cross-platform maintenance burden, which — after the initial expense — should free up engineering resources for more useful product work.

## Alternatives

### Do Nothing

The main alternative to this roadmap proposal is to do nothing.

There’s no immediate additional cost to doing so, beyond, as Alex puts it, "shipping shitty products, really slowly!"

We will continue to bear the high cost of maintaining and extending Sync, we will continue to have a broad array of undocumented and inconsistent storage systems, Sync will continue to be a lower-quality feature than we want, and as a result we will continue to offer inconsistent and incomplete experiences across our products, harming mobile adoption.

We will be largely unable to offer Context Graph-like features on top of existing user data. Telemetry data and Pocket will thus be the foundation of Context Graph. Activity Stream will soon face significant difficulties in storing and syncing new data; ultimately they will end up building an _ad hoc_ event storage system, and will collapse under the effort of replicating that work on iOS and Android where staffing is limited. New agent projects and mobile browsers will reinvent their own non-syncing storage, slowing development and convergence. Prototyping and experimentation will continue to be costly, particularly when integrating with or extending existing data.

### Bottom-Up

Another alternative is to allow this work to happen bottom-up. We are skeptical of this possibility: evidence suggests that engineers have neither incentive nor leverage to tackle cross-component work like this. [Conway’s Law](https://en.wikipedia.org/wiki/Conway's_law) applies. We are trying to make our architecture decision making clearer, not force it under the radar. Worse, this encourages unsustainable heroism — if we think improvement is necessary, we should produce a supportive environment for it to occur.

### Rebuild Whole Products

A more ambitious alternative is to construct new products in the right way — up to and including constructing a new browser. Mozilla has no plans to ship a Servo-based browser as a replacement for Firefox, so this strategy certainly doesn’t fly on desktop. It doesn’t avoid cross-platform costs, as discussed above. And — more significantly — simply building a new product is no guarantee that the work will be done differently. Servo is a very modern renderer, but without incentives to make other choices, it has already started down Firefox’s path with its prefs storage. Zerda/Firefox Rocket's initial data stores are bespoke and not syncable, despite having access to Fennec's code, because that team's incentives and the costs of reuse don't align.

### Improve Firefox Sync 1.5

We could attempt to invest in Firefox Sync 1.5 to try to make it incremental, extensible, durable, etc., rather than exploring alternative solutions. In the last 7+ years there's been enough architectural motivation to design Sync 2.0, Kinto, QueueSync and TreeSync (the latter two during the short-lived PiCL project). This, and recent experiences with adding data to Sync 1.5, suggest that we’ve improved the old Weave architecture as far as it can go. Additionally, we are not staffed to grow all five Sync client implementations in parallel.

## Competitive analysis and simplifying opportunities

Some of the requirements we've captured, and some aspects of the situation in which we find ourselves, might be malleable.

Some of these requirements constitute strategic choices: _e.g._, end-to-end encryption offers some strong privacy guarantees, but impacts recoverability and access by other services. The set of such strategic choices makes up a space of solutions. We expect the outcome of this roadmap to make progress towards defining a complementary set of points in this space.

In this section we briefly examine some of the choices available to us.

### Encryption and servers

Storing data in a way that prevents Mozilla from seeing cleartext means that the server can't help much with conflict resolution, versioning, data access, _etc_.

Chrome makes a different strategic choice, and is thus able to show your history on the web. [Realm](https://realm.io), a cross-platform database that competes with Core Data, offers automatic synchronization of its 'realms' between clients via a central Realm Object Server, with all data in cleartext.

A solution that could rely on cleartext data would be able to shift logic from the client to a single server infrastructure, which would increase agility and simplify the client.

The end state of this shift is something like Pancake or Facebook: a thin client with a rich server application and a server-canonical data model. This avoids synchronization entirely.

A number of systems use encryption at rest, but not end-to-end encryption. This allows for simpler web access, sharing, and recovery. Dropbox uses this model for files. This approach is not resistant to subpoena or intrusion.

This proposal assumes that end-to-end client-side encryption is table-stakes for some or all of Mozilla's applications, and so we can't disregard this requirement.

### Data-rich client applications

If we were to commit to storing less data on clients, and changing the set of stored data less frequently, then the cost-benefit analysis of this problem would shift, perhaps to the point that significant engineering investment should be avoided. To an extent this is similar to the alternative "Do Nothing": we currently try to avoid growing or changing the data we store, and we change Sync very infrequently.

Chrome is an example of this: there is no evidence of Chrome moving towards anything more than its rudimentary history and bookmarks storage.

This proposal assumes that there is product desire to leverage more user data on clients, not less, and eventually we will have to tackle the issues outlined in this proposal.

### Offline writes

If we were to make significant changes to the relationship between users and accounts, and between devices and servers, we could simplify the problem of syncing. For instance, if we linked all profile data to a Firefox Account from first opening Firefox, then repeated large-scale merges can be avoided. If we require all devices to be connected in order to record data (providing high availability for server storage), then we can turn synchronization into a distributed write. These are still non-trivial engineering problems, but they're different problems.

Most database systems we've examined, including Realm, Dropbox Datastore, CouchDB/PouchDB, and others, allow for offline writes.

This proposal assumes that our ability to tie writes to server resources is limited, for both technical and product experience reasons, but we have been thinking about this possibility.

### The definition of ‘data’

Storage systems might store some or all of documents, small blobs (_e.g._, icons), large blobs (_e.g._, downloaded files), independent structured data (_e.g._, emails), interrelated structured data (_e.g._, social graphs or bookmark trees), measurements/events (_e.g._, sensor data), and more.

A system optimized for document-structured data will typically scale horizontally but lack features like cross-document transactions. A system optimized for graph traversal might not handle blobs well.

Choosing carefully which kinds of data to support in which systems makes it easier to meet requirements.

Tools like CouchDB and MongoDB aim to provide scale and simplicity in syncing by deferring conflict resolution, identity, and validation to application code, themselves working with a more simplistic data model. Dropbox's [decommissioned](https://blogs.dropbox.com/developers/2015/04/deprecating-the-sync-and-datastore-apis/) Datastore API treats datastores independently and [merges records field-wise to resolve conflicts](https://blogs.dropbox.com/developers/2013/08/how-the-dropbox-datastore-api-handles-conflicts-part-two-resolving-collisions/); that's a good set of tradeoffs, but it leaves identity merging, data evolution, and validation to the app.

## Strawman Roadmap

We expect to gradually firm up our understanding of concrete solutions and shipping vehicles over the course of the next two quarters: Phase 1.

### Phase 1: building confidence (2017Q4–2018Q1)

We have a good idea of the characteristics of a solution for structured user data that allows for the data evolution, experimentation/iteration, encrypted sync support, and reuse that our products need.

We have identified two main remaining puzzle pieces that the Browser Architecture team will explore: the client-side data pipeline that preserves existing query interfaces, and the concrete sync protocol that moves data around.

If we can't tackle those, then there is no mainstream path forward for this part of the solution. If we're wrong, we want to find out sooner rather than later.

Our first step, then, is to exercise those two parts by prototyping each, iterating as needed. If we fail, then we will reassess our approach. This is the cheapest and best route to failure.

The Sync team has a related plan for this timeframe to reduce uncertainty around Rust-based cross-platform dedicated storage APIs that wrap generic storage. This will build confidence in our ability to consolidate implementations across desktop and mobile while meeting the needs of product engineering teams, and help to determine the developer experience linking application and service.

In parallel, we can move forward with independent cleanup work: making APIs async (post 57 freedom), incrementally improving and teasing apart existing problematic stores (_e.g._, libpref), work being proposed elsewhere around process separation for storage, etc. These efforts will set the stage for productization, but also make the world a better place regardless of the success of this particular effort.

A successful outcome for this phase is one of: fast failure; a revised roadmap or strategy that reflects new knowledge; or a proposed set of design options that can be reviewed with an eye to tackling Phase 2.

### Phase 2: attack the hardest thing, fail fast (2018Q1–Q3)

The largest single point of risk beyond basic functionality is addressing Places.

Places' history store is a large source of value, Places-linked data is core to Sync, and it's also a good test case: a high-volume store of structured user data with existing complex query consumers.

If we can't tackle Places, then there's much less value in proceeding.

We think the quickest and lowest-risk approach is to intermediate some writes to Places, turning parts of the Places database into a read-only materialized view. This stress-tests the client-side data pipeline, and allows us to explore syncing new data (_e.g._, containers, alternative representations for keywords and tags), recovering Places data from the log, _etc_., without disturbing existing consumers.

At this stage we hope to find and fix scaling and performance issues before doing further work. If we fail here, the process has been both informative and relatively cheap.

As this phase proceeds we'll design and build production server capacity for the sync system, begin to address fit-and-finish concerns like push notification integration, and continue to assess requirements from new product efforts.

### Phase 3: consolidate (2018Q3–)

If we're confident in our new sync implementation, can use it from apps, and can handle Places-level scale, then we can proceed with consolidation and simplification to add value to Firefox. We can fan out: pick suitable storage (_e.g_., data that's high priority for syncing, an old store that's not owned by an existing team, …), make sure it has an async API, and port it on top of the log, following the example of Places.

Early in this phase our documentation will go from adequate to great, we will place heavier emphasis on the developer experience and UI programming integration (_e.g._, GraphQL), and we expect to see further adoption in mobile apps and on the server as the cost of accessing Firefox data decreases.

We can also assess the urgency of exposing this kind of storage — both access to Firefox's data, and dedicated storage — to WebExtensions.

These additional efforts will likely be assessed and planned closer to the time; it would be unrealistic to estimate resourcing or timing when priorities are likely to shift.

## Resources

[Requirements capture for Sync](https://docs.google.com/document/d/1XOJhW-qf4iIyGHwiSLgWEfXYDWLy44So5L8CdwH6pcg)

[Scoped encryption keys and access control for FxA](https://docs.google.com/document/d/1IvQJFEBFz0PnL4uVlIvt8fBS_IPwSK-avK0BRIHucxQ)

Vision statements

* [Ryan Kelly](https://docs.google.com/document/d/1MrXNcVzDDYyGSvud6CvVSoPGNyvpJvHGSDpRwIcalQc)

Blog posts

* [Thinking about Syncing, part 1](https://medium.com/@rnewman/thinking-about-syncing-part-1-timelines-7f758e2bd676) (2017)
* … [part 2](https://medium.com/@rnewman/thinking-about-syncing-part-2-timelines-and-change-90a0964f10f3)
* [Syncing and storage on three platforms](https://160.twinql.com/syncing-and-storage-on-three-platforms/) (2015)
* [Different kinds of storage](https://160.twinql.com/different-kinds-of-storage/) (user agent service) (2016)

## Thanks

Thanks to our readers: myk, asuth, mak, grisha, tspurway, snorp, overholt, ckarlof, wennie, selena, gps, trink, hildjj, and plenty more.

## Questions

**Are you going to require this everywhere?**

No; it's not applicable to all problems, nor does the cost-benefit calculation always play out. Decisions should be made based on suitability and value: _e.g._, to make some data available to other apps or platforms, to bring a store up to a feature baseline, or to integrate two data sources to support a new feature.

**Does this mean we’re using Mentat?**

Not necessarily. Our roadmap so far involves answering questions about how to provide capabilities that we think are important for Firefox's strategy. It's possible that parts of Mentat will evolve into parts of the solution — after all, it's an embedded Rust log-structured store — but a solution will be integrated with other parts of the Firefox ecosystem, not just a database.

**Does this restrict the design of Sync and Storage at Mozilla?**

Not really. We expect FxA identity-attached services to proliferate in time, and we expect there to be justified divergent approaches to storage, too — here we are mainly focused on structured user profile data, but elsewhere we are also _e.g._, helping to shape a redesign of libpref. What we hope to happen is that we can provide a small spectrum of solutions that are better choices than building yet another store from scratch, and that are dramatically _less_ restrictive than the current situation, achieving convergence at a healthy rate.

**Does this help ET?**

We hope so. Having reusable, portable components and data opens doors to exploration and development.

**How do you plan to scope out the problem of scaling the service?**

In short: by finding sets of properties that are beneficial for both the requirements we've captured for storage and syncing and also for management of a hosted service — including scalability — and designing a solution that reflects those properties.

This is something of a design question, but we have some idea of where we might go (we're initially proposing log-structured storage) so we can answer in brief. In short:

* Independent users and divisible data. There is little need to coordinate between encrypted storage for each user. Moreover, by making the log, entities, and other attributes meaningful concepts in the system, we can make the data for each user amenable to sharding along these axes. This allows us to scale horizontally.
* Mostly immutable data. Unlike a write-in-place system like Firefox Sync, which supports its constantly growing set of data by churning in place, we would like the majority of our storage work to be appends. The exceptions are well-defined operations that introduce barriers (like rollups, snapshots, or account deletion) and well-defined operations that rewrite parts of history (like Forget). Even with those exceptions, that's a healthier workload than we currently support in Firefox Sync, where a big pile of records is both constantly added to and constantly subject to random replacements — it makes the server simpler and the workload more predictable, and allows for cheaper long-term storage.
* Exploiting modern advances like push to reduce costs for common cases: improved responsiveness can eliminate a subset of conflict scenarios, which reduces load for both client and server.

Questions from ckarlof (not a lot of time to answer these, so a little brief, but better than nothing!):

**What are the outcomes we're trying to achieve here and how do you know we've achieved them? What are your key metrics of success?**

_I see some "representative product end states", but I feel these need to be more prominent in communicating your vision of success._

In addition to the concrete definitions of success in each phase, we'll know the overall effort has been successful if:

* The organization displays a culture of holistic thinking around user data across the Firefox ecosystem.
* Product managers feel more empowered to drive experiences that rely on new, integrated user data.
* The baseline of storage capabilities is higher, with consequences for the consistency of Firefox's UX (see, e.g., Wennie's doc).
* The cost of integrating new syncable data is markedly lower than it is today.

Figuring out how to concretely _measure_ our success is a task for after this review.

**How many different complete options are you considering to achieve your proposed outcomes?**

_I feel there's really only one under consideration, but I feel having one or two alternative approaches to reach your goal, even if they're straw dogs, would help folks understand the merits of your favored approach better. Your competitive analysis is close to this, but some of things you're comparing against aren't solutions to reach your vision. They're compromises. Sell me on your vision._

We're concerned with agreement that the situation is as described, that our vision is desirable, and that the situation can be reasonably improved. The alternatives listed above are 'roadmap alternatives', rather than design alternatives, precisely because those are different paths that the organization could take other than investing in a suite of technical solutions.

We have some understanding of characteristics that we feel are important to forward-looking technical solutions, and the next steps on the roadmap are focused on exploring those characteristics. The results of those explorations will dictate the concrete approaches we consider.

**For the approach you're suggesting, what are they key choices in that approach that you want us to understand? Using Rust to implement a shared library is one such strategic choice. Are there others worth surfacing?**

The proposal to use Rust is essentially an assertion of two things: that supporting multiple applications on multiple platforms is part of Mozilla's future; and that native code is the right way to do that in this instance. Rust is, we will admit, an opinionated choice here.

There are other strategic/organizational opinions that lie beneath this document:

* That some organizational coordination of storage — including centralized reusable implementations — is valuable, both to individual features and to the organization as a whole.
* That totally separating syncing from storage doesn't work, and that pushing onto feature developers the burden of understanding syncing and other consumers, without tool support, is costly. Abstractions are needed.

**What would have to be true for your approach to be a winning one? What are your riskiest assumptions here?**

_I'm looking for a prioritized list of questions that need answers and assumptions that need to validated, an order to tackle them in, and how you're going to tackle them (ideally, as cheaply and as early as possible)._

Phase 1 and Phase 2 capture what we think are the most significant technical risks: can we build a system that more closely meets [the requirements we've captured for sync and storage](https://docs.google.com/document/d/1XOJhW-qf4iIyGHwiSLgWEfXYDWLy44So5L8CdwH6pcg/edit), in a way that serves our strategic needs (_e.g._, around reusability), and maintain syncability, performance, and the developer experience (including the preservation of existing interfaces for as long as makes sense) — can we build it? can we make it sync? can we make it scale? can we get people to understand it?

We'll break these down into more detailed work items and questions as we go about working on those phases.

From a product perspective, most (if not all) of the following would have to be true for this to be a success and to meet the needs of the engineering teams we’ve spoken to:

* We have a sufficiently accurate understanding of the needs of Mozilla engineering teams.
* We have a sufficiently accurate understanding of our users’ needs and problems.
* Teams have a desire/need to store new data.
* Teams have an increased need to more easily access existing stored data from other teams.
* We can successfully collaborate across departments.
* Users trust us with their data.
* Engineering teams trust us with their data.
* Existing storage is insufficiently easy to extend or scale.
* Engineers agree with the shortcomings of the current situation.
* Engineers would stop using _ad hoc_ solutions to create new products.
* Engineers are OK with having less control of their integration due to being limited to APIs
* There is enough of an overlap between all teams that would allow us to have solutions that fit all.
* New smaller mobile products won’t need to pull down all of the data locally since that can be a battery and storage drain.
* Mozilla wants to make personalized data-centric features (Firefox eco-system?)
* Users want a personalized browsing experience.
* In the case of making it ourselves, nobody has a working solution that we could use or fork which would allow us to achieve similar aspirations.
* We have the technical expertise to build a new storage system and cloud storage infrastructure.
* Our solution would provide increased flexibility to engineering teams in terms of adding new data types and changing existing ones.
* For new projects, that we are faster and easier to integrate than _ad hoc_ solutions, ideally enabling new projects to more rapidly pop-up across the company.
* We still want to encrypt all of our users data and work within those constraints.
* That we can satisfy the desire for experimentation within product teams and their current lack of data storage solutions that enable them to do this.
