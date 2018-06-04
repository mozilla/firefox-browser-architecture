# XULStore Using rkv – Proof of Concept

In his "Key-value storage proposal," Richard Newman proposed the "standardization of a simple key-value storage capability, based on LMDB, that is fast, compact, multi-process-capable, and equally usable from JS, Java, Rust, Swift, and C++." Dave Townsend reviewed the proposal and concluded that we should pursue it.

This document summarizes the proposed implementation of key-value storage, explains the purpose and data model of XULStore, and describes the proof of concept implementation of XULStore using key-value storage that is posted to [bug 1460811](https://bugzilla.mozilla.org/show_bug.cgi?id=1460811). It then identifies findings, notes risks, and suggests next steps, ending with a note on data migration strategies.

tl;dr: we should re-implement XULStore using key-value storage to improve its reliability, reduce its complexity, and potentially improve application startup performance while we continue to investigate other consumers for rkv, including session store, TLS session tickets ([bug 1428901](https://bugzilla.mozilla.org/show_bug.cgi?id=1428901)), other desktop consumers of JSONFile, and mobile apps like Fenix.

## rkv

The [rkv crate](https://crates.io/crates/rkv) is the proposed implementation of key-value storage for Firefox. It's a high-level API to the [LMDB](https://symas.com/lmdb/) library that wraps the low-level [lmdb crate](https://crates.io/crates/lmdb) and intends to improve developer ergonomics and safety via failure-based error propagation, management of database handles, and typed value encoding/decoding, among other abstractions.

rkv has the potential to replace a variety of bespoke data stores based on JSON and other formats that Firefox uses today for relatively simple data that isn't synced. The potential benefits of replacing such stores with rkv include improved reliability and durability, reduced complexity and maintenance cost, and in some cases increased performance and responsiveness.

## XULStore

XULStore is an example of one such store. It persists a set of element attribute values of chrome XUL windows (and, after [bug 1453788](https://bugzilla.mozilla.org/show_bug.cgi?id=1453788), chrome HTML windows), especially their width, height, and screen coordinates. It is currently implemented as a JS XPCOM component that implements the [nsIXULStore](https://searchfox.org/mozilla-central/source/toolkit/components/xulstore/nsIXULStore.idl) API and stores data in the _xulstore.json_ JSON file.

XULStore "records" comprise 4-tuples of (document URL, element ID, attribute name, attribute value), all components of which are strings.

## Proof of Concept

The PoC vendors rkv into mozilla-central and adds a "xulstore" crate that expresses two C ABI-compatible sets of functions to Firefox for storing and retrieving XULStore data: a C-string based set that accepts and returns raw pointers to char* arrays, and an nsString-based set that uses the bindings in the [nsstring](https://searchfox.org/mozilla-central/source/xpcom/rust/gtest/nsstring) and [nserror](https://searchfox.org/mozilla-central/source/xpcom/rust/nserror) crates to accept and return Gecko nsString and NS_* result values.

The PoC then modifies the XULStore XPCOM component to store data in rkv rather than the JSON file. It includes C++ and JS tests of both sets of functions, and it also passes all existing XULStore tests.

### Technology Stack

A rough outline of the component stack (and implementation languages) in the PoC:

![drawing](https://docs.google.com/drawings/d/e/2PACX-1vTeV2q50bm1uDoF-YnTxlawWvJFbPF-gjM0dgqgw5d90rP7Z9f5wIpj31zv9pxqVt9QiSixVAJZNcr9/pub?w=292&h=386)

Only the edges of this stack are essential—in principal, both C++ and JS consumers could call directly into LMDB, avoiding all Rust layers and the XPCOM intermediary—but each layer serves a useful function: the "lmdb" crate wraps LMDB's C interface in safe (albeit low-level) APIs; rkv improves on developer ergonomics and safety with higher-level abstractions; the "xulstore" crate translates the XULStore data model to arbitrary key-value pairs; and the XPCOM component provides idiomatic access for both C++ and JS consumers.

Note that the PoC doesn't rewrite the JS XPCOM component in C++, as that work is a "known known." A complete implementation would likely do so, however, given that there are limitations to XPCOM components implemented in JS, especially those that call C functions and manage non-GCed data.

(Specifically, the consumer of the C API needs to free Rust-allocated strings and iterators after finishing with them, but JS lacks destructors, so a JS implementation would either need to copy the data into JS objects and free the memory itself manually or force its own consumers to explicitly free the memory themselves, which would add complexity and consume resources.)

Here's what the stack would look like in a complete implementation (including other consumers of the rkv crate):

![drawing](https://docs.google.com/drawings/d/e/2PACX-1vTe9LCCP6XGn77-_Et1n-LbzVvZXoJiOL4M8fgwv2XgTJOtTxHeCFKSVolcEVl2gSk1ZcUSi3LckGT_/pub?w=572&h=388)

### Data Model

There are multiple ways to represent XULStore records in a key-value store. This PoC translates the XULStore data model into rkv concepts by mapping:

*   JSON file store -> rkv store
*   (XUL document, element ID, attribute name) -> rkv key
*   attribute value -> rkv value

In the rkv implementation a "store" is an abstraction over an LMDB "[database](http://www.lmdb.tech/doc/group__internal.html#structMDB__db)", and there can be multiple distinct stores within a single LMDB "environment", which is represented in rkv by an "RKV" object. Note that the maximum number of rkv stores that will be accessed via a given RKV instance needs to be known in advance, during initialization of the instance.  Also note that "a moderate number of slots are cheap but a huge number gets expensive," per [http://www.lmdb.tech/doc/group__mdb.html#gaa2fc2f1f37cb1115e733b62cab2fcdbc](http://www.lmdb.tech/doc/group__mdb.html#gaa2fc2f1f37cb1115e733b62cab2fcdbc). The PoC uses a single rkv store within a single environment.

As rkv keys are strings, the (XUL document, element ID, attribute name) tuple is serialized to a concatenation of its components (with separator char).

A complete implementation might choose the alternative of mapping documents to rkv keys in a single rkv store and storing (element ID, attribute name, attribute value) triples for a given document as a single rkv value, using JSON or the like to structure and serialize the data to a blob:

*   XUL document -> rkv key
*   blob of (element ID, attribute name, attribute value)* -> rkv value

## Findings

### rkv Completeness

Although rkv is incomplete, and it doesn't yet provide some all of the intended ergonomic improvements and safety guarantees, it is nevertheless complete enough to satisfy most of XULStore's requirements, especially given the maturity of the underlying API (and LMDB itself).

The PoC includes only a few small changes to the vendored copy of rkv, and it only bypasses rkv to access the low-level LMDB API directly in one case (to retrieve a read-only cursor for iterating keys).

### rkv Limitations

One particular limitation is that rkv doesn't store arbitrary blobs of data, so the PoC has to store attribute values as UTF-8 strings, even though the strings that its consumers persist (including both JS and C++ consumers) are UTF-16, which results in unnecessary string conversions. [rkv issue #20](https://github.com/mozilla-prototypes/rkv/issues/20) tracks adding blob support. (Update: that issue has now been fixed, and the PoC could be updated to store values as blobs, although it would still need to convert the 16-bit strings to 8-bit arrays of bytes.)

### Feasibility of Direct C API

It's feasible to implement a C API in Rust and call into it directly from both C++ and JS consumers, but it would be more idiomatic to retain an XPCOM interface to the Rust API, especially given that there are both C++ and JS consumers. That XPCOM  interface should be implemented in C++, given the limitations of JS.

## Risks

### Impact on Performance

While rkv will improve the reliability and reduce the complexity of the XULStore implementation, it isn't clear that it'll improve application startup perf. XULStore is on the startup path, so it could theoretically do so. But the current implementation doesn't show up much in profiles, so the XULStore workload may not be hot enough to matter either way.

Regardless, we need to ensure that rkv doesn't regress startup perf. And we can mitigate this risk via testing.

### Proliferation of Implementations

rkv needs to be useful for more than just XULStore to justify its footprint and the effort to develop, maintain, and integrate it (i.e. to avoid the "proliferation of implementations" problem, a.k.a. [https://xkcd.com/927/](https://xkcd.com/927/)). Although we've identified a variety of other potential consumers, there are no other concrete prototypes yet.

We can mitigate this risk by waiting to complete the implementation until other rkv consumers appear or by committing not to land the implementation until we see the whites of other rkv consumers' eyes. We can also define a program to replace all JSON stores with rkv and gain approval for that program from engineering leadership.

### Filesystem Compatibility

Although LMDB should be compatible with a broad array of filesystems, and the only known incompatibility is with multi-process access on networked filesystems (which isn't an issue for XULStore, whose only consumers live in the parent process), LMDB was originally developed to back an LDAP server, which presumably targets a narrower (and more controlled) range of filesystems than those employed by our users.

So it's possible that LMDB will be incompatible with some archaic or unusual filesystems that Firefox should continue to support.

We can mitigate this risk by the usual technique: testing on Nightly and Beta, with a feature flag that enables us to disable the feature on release until we've gained confidence in it. We could also mitigate it to some extent by storing data in both the old and new stores for a period of time, as described in [Notes on Migration](#Notes_on_Migration), in a way that allows to disable the new implementation selectively for incompatible filesystems without losing user data.

### Known Unknowns

There remain questions to be answered around LMDB usage more generally, such as how often we should check for and clear stale readers, how to check for and recover from data corruption, and how often (and how) to vacuum a datastore.

## Next Steps

1.  Upstream rkv changes and implement desired rkv features.
1.  Measure PoC perf and efficiency and investigate any regressions.
1.  Research known unknowns and convert them to knowns.

## <a name="Notes_on_Migration"></a> Notes on Migration

The PoC doesn't implement data migration from the existing JSON store, but there are several ways it might be implemented.

### Without Support for Downgrades

If support for downgrading Firefox is not a priority, then the new implementation can detect an upgrade and migrate data from the JSON store to the rkv one, removing records from legacy extensions and the JSON store file itself after migration is complete.

This is the conventional approach that Firefox has employed for many previous datastore migrations, including XULStore's own migration from localstore.rdf to xulstore.json in Firefox 34 via [bug 559505](https://bugzilla.mozilla.org/show_bug.cgi?id=559505), with the import code subsequently removed in Firefox 55 via [bug 1368567](https://bugzilla.mozilla.org/show_bug.cgi?id=1368567) (albeit that migration didn't remove records for legacy extensions, as those extensions weren't legacy at the time).

On initialization, the implementation checks for the existence of the new datastore, and if the file doesn't exist, then the implementation imports data from the old datastore, as shown in this [code snippet from XULStore.js](https://hg.mozilla.org/mozilla-central/file/7970ea085861/toolkit/components/xulstore/XULStore.js#l68) (from before the import code was removed):

```
   if (!this._storeFile.exists()) {
      this.import();
    } else {
      this.readFile();
    }
```

This strategy has the benefits of being conceptually simple, straightforward to implement, and satisfying the use cases of the vast majority of Firefox users who don't downgrade their installations. It is also the well-trodden cow path that Firefox has employed successfully for many previous data migrations.

However, it doesn't completely satisfy the use cases for the subset of users who do downgrade their browser installations, either because they switch between versions of multiple installations as a matter of course (f.e. between Beta and Release when testing websites) or because they downgrade Firefox after experiencing issues with a new version.

### With Support for Downgrades

If support for downgrading Firefox is a priority, then several approaches can be taken to ensure it, described here in rough order of effort and impact (from least to greatest).

#### Retain Old Store

The simplest form of support for downgrades is to retain the old store (JSON file) in the profile directory after migrating data to the new store, such that a user who upgrades and then downgrades Firefox will go back to their most recent state preceding the upgrade.

We can achieve this level of support for downgrades basically for "free," by merely not deleting a file that we would otherwise delete, so it is trivial to implement. However, its impact is also limited, as it loses all XUL document state changes that take place after the upgrade.

This approach is a reasonable solution for the use case of a user who upgrades Firefox, quickly discovers an issue with the new version, and immediately downgrades, as those users are unlikely to have made significant (or any) XUL document state changes during their brief time running the new version.

This approach does not satisfy the use case of a user who switches between versions of Firefox as a matter of course, since state changes will not propagate between stores (in either direction) after the initial migration.

#### Retain Records From Legacy Extensions

The implementation could retain records from legacy extensions, so a user who downgrades Firefox to a version that still supports them will go back to their most recent state, without any data loss (as the state cannot change when the extension is disabled).

As with "Support Only Most Recent State Before Upgrade", this is trivial to implement, as it only requires us to not do something that we would otherwise do. And just like that approach, its impact is also limited, as the current version of Firefox (60) is already four versions newer than the last one that supported legacy extensions (56), and it's the basis for the latest ESR, so the number of users still on versions that support legacy extensions who upgrade to a version of Firefox with the new XULStore and then choose to downgrade will presumably be very low.

#### Maintain Both Stores

The implementation could persist data in both the old and new stores for a period of time sufficient for the likelihood of a downgrade to approach zero, with the new store serving as the system of record, and writes to the old store being deprioritized, so they don't affect application responsiveness.

This approach requires more effort than the others but also satisfies all use cases. Instead of removing the old store—or retaining it but ignoring it after migrating data to the new store—the implementation would retain and update both stores internally, writing changes to both stores while reading only from the new one (which would become the source of truth, albeit not a single one). So downgrades, whether one-off (due to issues with a new version) or regular (a user who moves back and forth between two versions) would always retain the latest state.

This approach raises some implementation questions, such as whether to retain the existing JS implementation of nsIXULStore that uses the old store or merge it into the C++ implementation that uses the new store (and if so, how to abstract across the two implementations).

Nevertheless, there are straightforward answers to these questions, and it doesn't seem to present technical challenges. It's just more work, and a bigger footprint for the period of time during which the two implementations coexist.

In theory, this approach could also raise design questions, such as how to support downgrades from a version of Firefox that no longer implements its interface with XUL documents. However, there are no known breaking architectural changes on the horizon, as we have no plans to migrate Firefox interfaces to any other technology besides HTML, with which XULStore is already compatible, per [bug 1453788](https://bugzilla.mozilla.org/show_bug.cgi?id=1453788).
