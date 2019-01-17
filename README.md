
# Firefox Browser Architecture

## Mission

Change Mozilla. Investigate big technical challenges and produce engineering programs to address them.


## Our Conclusions

This is a list of our findings that we're reasonably happy with so far.

* [Documenting our output](text/0001-documenting-output.md) looks at how we’re going to communicate with the rest of Mozilla.
* [Extracting Necko](text/0002-extracting-necko.md) considers whether it's feasible or worthwhile to extract Necko — Gecko's C++ networking library — for use as a standalone component.
* [Problems with XUL](text/0003-problems-with-xul.md) aims to list the different kinds of problems that exist with XUL.
* [XBL and Web Components](text/0004-xbl-web-components.md) compares some old Mozilla technology (XBL) with modern Web Components.
* [Problems with XBL](text/0005-problems-with-xbl.md) aims to list the different kinds of problems that exist with XBL.
* [Architecture Reviews](text/0006-architecture-review-process.md) are healthy and we proposed a process for healthy reviews.
* [XBL Design Review packet](text/0007-xbl-design-review-packet.md) is the packet that we prepared for the architectural design review for XBL removal.
* [Roadmap Review: Sync and Storage](text/0008-sync-and-storage-review-packet.md) establishes that storage and syncing of user data is a pillar of the Firefox ecosystem, warranting holistic and long-term attention, and outlines where we’d like to end up and some ideas for how to get there.
* [JavaScript Type Safety Systems](text/0009-type-safety-systems.md) are some conclusions of an investigation into the use of JavaScript type safety systems.
* [Firefox Data Stores Documentation](text/0010-firefox-data-stores.md) documents the existing data stores across all current Firefox platforms.
* [Fluent in Prefs Design Review](text/0011-fluent-in-prefs-design-review.md) describes the lightweight design review for Fluent in Prefs.
* [A brief analysis of JSON file-backed storage](text/0012-jsonfile.md) outlines some of the pros and cons of using a flat file (particularly via `JSONFile.jsm`) to store data in Firefox.
* [Process Isolation in Firefox](text/0012-process-isolation-in-firefox.md) is a WIP evaluation of how far we can push process isolation to improve security and stability.
* [IPC Security Models and Status](text/0013-ipc-security-models-and-status.md) is an audit of our current IPC status.
* [XUL Overlay Removal Review Packet](text/0014-xul-overlay-removal-review-packet.md) is the packet that we prepared for the architectural design review for XUL Overlay removal.
* [Design Review: Key-Value Storage](text/0015-rkv.md) proposes the introduction of a fast, cross-platform key-value store for Mozilla products.
* [XULStore Using rkv – Proof of Concept](text/0016-xulstore-rkv-poc.md) describes a proof-of-concept implementation of XULStore that uses [rkv](https://github.com/mozilla/rkv).
* [LMDB vs. LevelDB](text/0017-lmdb-vs-leveldb.md) compares the [Lightning Memory-mapped Database](https://symas.com/lmdb/) (LMDB) key-value storage engine to the [LevelDB](https://github.com/google/leveldb) key-value storage engine.

## Posts

We typically send our newsletters to [firefox-dev](https://www.mozilla.org/en-US/about/forums/#firefox-dev).

* [Browser Architecture Update](newsletter/_posts/2017-07-27-browser-architecture-update.md). See also [mailing-list-post](https://groups.google.com/forum/#!topic/firefox-dev/ueRILL2ppac).
* [Browser Architecture Newsletter #2](newsletter/_posts/2017-08-24-browser-architecture-newsletter-2.md). See also [mailing-list-post](https://groups.google.com/forum/#!topic/firefox-dev/Rc2w2a9e8HQ).
* [Browser Architecture Newsletter #3](newsletter/_posts/2017-09-22-browser-architecture-newsletter-3.md). See also [mailing-list-post](https://groups.google.com/forum/#!topic/firefox-dev/p9rTlfFUXlQ).
* [Browser Architecture Newsletter #4](newsletter/_posts/2017-10-19-browser-architecture-newsletter-4.md). See also [mailing-list-post](https://groups.google.com/forum/#!topic/firefox-dev/CLFtj8qUSv8).
* [Browser Architecture Newsletter #5](newsletter/_posts/2017-11-29-browser-architecture-newsletter-5.md). See also [mailing-list-post](https://groups.google.com/forum/#!topic/firefox-dev/XKp3EthdJ60).

## Explorations and Experiments

To support our conclusions we occasionally perform explorations and experiments. The first exploration is designed to support the notion that we can create a sync and storage layer in Rust that we can deploy to Desktop, Android and iOS.

* [Deploying a Rust library on iOS](experiments/2017-09-06-rust-on-ios.md). A short tutorial describing how to build and deploy a Rust library for use inside an iOS app.
* [Deploying a Rust library on Android](experiments/2017-09-21-rust-on-android.md). A short tutorial describing how to build and deploy a Rust library for use inside an Android app.
