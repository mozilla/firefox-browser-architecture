---
title: Extracting Necko
layout: text
---

# Extracting Necko

## Proposal

We should extract Necko for reuse in background services and on mobile platforms.

### Motivation

We expect that Mozilla will need to build consistent native applications and background services across platforms, including services and applications that don't run Gecko atop a profile directory. To do this it would be beneficial (in theory) to leverage our existing investment in our network layer, avoiding the confusing situation of having multiple ways to configure network settings, and avoiding the possibility of different behaviors in different parts of an application.

Furthermore, we expect the question of reuse to recur. Necko is sometimes characterized as being a relatively self-contained piece of mozilla-central's compiled code, so it should be one of the easiest to reuse. Also, its interdependencies are probably applicable to other components, so it's a good opportunity to learn.

Breaking out a component for reuse is, for our purposes, equivalent to embedding: it's taking the component and driving it in a different context. Specifically, we are interested in those contexts being (a) Rust, (b) not necessarily desktop. An alternative approach is to build a new Gecko application on top of a slimmed-down libxul. In the case of Necko, that might be something like a cURL clone. This kind of reuse is a known quantity, so we won't discuss it here.

## References

* Lots of very old documents:
  * <https://developer.mozilla.org/en-US/docs/Mozilla/Projects/Necko/Embedding_Necko>
  * <https://developer.mozilla.org/en-US/docs/Mozilla/Projects/Necko>
  * <https://developer.mozilla.org/en-US/docs/Mozilla/Projects/Necko/Architecture> 
* Discussions with rhelmer and jduell
* Examination of `/netwerk`

## Analysis: obstacles to embedding

* In its early days, 15+ years ago, Necko was intended to be embeddable; in recent years that has not been an aim of the component leads. Meeting the needs of Gecko and Firefox is more important. (The definition of 'embeddable' here still carries the obligations of Gecko-style initialization.)
* As noted in the "Embedding Necko" document, Necko depends on Gecko features:
  - XPCOM and NSPR.
  - mDNS, `NetUtil.jsm`, and some HTTP functionality requires a JS runtime. This is feasible to eliminate if needed.
  - Docshell.
  - If cookies are desired, they must be provided via the nsICookieService interface, which is baked into Necko. One can build Necko without, via `--disable-cookies`, but this is a build-time decision. One could conceivably provide one's own implementation of the cookie service, but the interface requires a stateful synchronous global singleton.
  - Necko depends on the Mozilla preferences service (libpref) for configuration, which is a privileged singleton that itself depends on the observer service (for startup and profile events) and the directory service (for profile directory and app locations).
* Because of its dependency on the profile (both for reading state and as a key for the disk cache) and flushing files to disk for prefs, it's not safe to use Necko from two distinct processes simultaneously for the same application.
* Because of its extensive use of XPCOM singleton patterns, it's not possible to have two independent Neckos in the same process; they need separate processes and separate profile directories. Some independence can be achieved by carefully configuring channels, but it's not isolation.
* It's not possible to use Necko without a profile service (*e.g.*, to make a network request without touching disk).
* Parts of Necko are still single-threaded, requiring new channels to be created on the main thread, and dispatching callbacks to the main thread.
* Necko's init and deinit are managed by observer notifications, which is not ideal for embedding outside of Gecko.

# Proposal
## Proposal
# Conclusion

Necko is a very feature-rich C++ networking library that is directly targeted at meeting the needs of single-user, single-main-process, single-profile, interactive applications built on a persistent profile directory in the traditional Mozilla model. It is not well-suited for use outside of the Gecko lifecycle — indeed, almost every aspect is designed for the Gecko world — and would require substantial effort in decoupling to be suited for embedded use. Even if so decoupled, it would still mandate the use of XPCOM and the NSPR.

To extract Necko for reuse outside of Gecko, by non-XPCOM-based applications, would — almost by definition — require coming up with a plan to deal with XPCOM. That plan might look a lot like continuing historical efforts on deCOMtamination.

Necko's data structures, interface definitions, error codes, macros, etc. *are* XPCOM's, and so eliminating XPCOM dependency from the module is likely to involve touching almost every line of code. [Automated tools to do so have been proposed](https://brendaneich.com/2006/10/mozilla-2/).

(Going further to eliminate NSPR and mfbt data structures, yielding a more standard C++ library, would be further work still.)

This is particularly relevant when we consider oxidation — the replacement of C++ and JS components of Firefox with Rust — and even more so when we consider the use of such components both outside and inside Firefox *from* Rust (rather than from existing non-Rust callers).

There is ongoing work on landing Rust components in Firefox; most obstacles there are around XPCOM integration for Rust code, which currently requires writing lots of routine boilerplate by hand.

There is also some initial scoping of work to extract and use Gecko C++ components from Servo, which is a similar problem to using Gecko C++ components from Rust background services or mobile applications. Those estimates are large (but not yet published), on the order of 3-5 engineer years for non-trivial components.

Extracting such a module is not routine work: delicate balances need to be found between maintaining useful integration with Gecko (*e.g.*, Necko's `nsIChannel` permeates the codebase) versus designing for embedding, and between reimplementing Gecko dependencies in Rust for reuse versus reworking the component to avoid them.

Our position is that extracting Necko from Gecko for reuse would be a significant amount of work. Doing so in a way that allows its use from ordinary Rust consumers, without shipping XPCOM/NSPR/*etc*., would be even more so.

That amount of work is certainly much more than simply using the existing well-tested and idiomatic networking libraries available to Rust code. Given that the benefits of using Necko in its entirety (instead of *e.g.*, just using Hyper for HTTP) are relatively small, our conclusion is that we **should not attempt to reuse Necko outside of Gecko**.

Standalone Rust components should use Rust networking libraries of equivalent functionality, *e.g.*, Tokio/Hyper (already vendored for Servo) or libpnet. Useful Necko functionality should be ported to Rust as needed. Those Rust reimplementations can be imported back into Necko like any other piece of oxidized code.

## Alternatives

Alternatives are, variously:

* Do the work to extract Necko. Some significant benefit would need to be articulated in order for this to be worthwhile.
* Use Gecko for all network-related background services. We already know that this is not feasible, both for application size reasons (Gecko is large), and for technical lifecycle reasons as articulated above. Furthermore, this argument ends up as "use Gecko for everything everywhere", which is not useful.

## Links

This document was originally drafted in the ["Answered questions" Google Doc](https://docs.google.com/document/d/1RIMbe0yJGNywFTf1n5ka35JpLtrkE3lHz8iip6jH_VE/edit#).

## Colophon

This document was written by Richard Newman (rnewman), reviewed by Jason Duell (jduell), Rob Helmer (rhelmer), Joe Walker, and Myk Melez.
