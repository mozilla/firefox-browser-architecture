# LMDB vs. LevelDB

This document compares the [Lightning Memory-mapped Database](https://symas.com/lmdb/) (LMDB) key-value storage engine to the [LevelDB](https://github.com/google/leveldb) key-value storage engine.

## Meta

### Project Structure

#### LMDB

LMDB is developed by <a href="https://symas.com/">Symas Corporation</a>, a small "technical support firm" whose business model is to provide technical support for the open-source software it develops. Its primary product is OpenLDAP, and it originally developed LMDB for use in that server software, which continues to use it. It provides paid technical support for LMDB.

The LMDB code is stored in OpenLDAP's repository and read-only mirrored to GitHub. Issues and patches are accepted upstream, not in the GitHub repository. LMDB doesn't appear to have an active community beyond the core developers.

#### LevelDB

LevelDB was "written at Google" (according to its <a href="https://github.com/google/leveldb">GitHub project page</a>) and "inspired by <a href="https://en.wikipedia.org/wiki/Bigtable">Bigtable</a>" (according to its <a href="https://en.wikipedia.org/wiki/LevelDB">Wikipedia entry</a><sup>[1](#footnote-1)</sup>), although it doesn't share code with that proprietary storage system. It was originally developed, among other reasons, to back IndexedDB in Google Chrome, which continues to use it for that purpose.

The LevelDB code is stored on GitHub in the <a href="https://github.com/google/leveldb">google/leveldb repository</a>. It doesn't appear to have an active community beyond the core developers, although there seems to be an active community of developers of <a href="http://leveldb.org/">NodeJS bindings</a> and documentation.

### License

#### LMDB

LMDB uses the <a href="https://github.com/LMDB/lmdb/blob/mdb.master/libraries/liblmdb/LICENSE">OpenLDAP Public License</a>, which appears to be a permissive BSD-style 3-clause license.

#### LevelDB
LevelDB uses the <a href="https://github.com/google/leveldb/blob/master/LICENSE">BSD 3-Clause "New" or "Revised" License</a>.

### Language

#### LMDB

LMDB is implemented in C and has bindings to multiple languages, including Rust via the <a href="https://crates.io/crates/lmdb/">lmdb crate</a>, among others. The <a href="https://github.com/dw/py-lmdb/">Python binding</a> appears to be mature (although it is also <a href="https://github.com/dw/py-lmdb/commit/78afd828139ad8a7b3555bdfc0df8a556ef0e505">looking for a new maintainer</a>).

#### LevelDB

LevelDB is implemented in C++ and has bindings to multiple languages, including Rust via the <a href="https://crates.io/crates/leveldb">leveldb crate</a>, among others. The <a href="https://github.com/Level/levelup">Node.JS binding</a> appears to be particularly popular.

### Rust Bindings

Overall, the lmdb crate appears to be more mature and higher quality than the leveldb crate, but both crates suffer from issues, incomplete documentation, and project inactivity.

#### LMDB

The <a href="https://crates.io/crates/lmdb/">lmdb crate</a> is a Rust binding for LMDB that describes itself as "Idiomatic and safe APIs for interacting with the Symas Lightning Memory-Mapped Database (LMDB)."

The author has released <a href="https://crates.io/crates/lmdb/versions">11 versions</a>, the first in November 2014 and the most recent in March 2018. The crate uses a conservative versioning scheme, with the most recent version being 0.8.0.

The crate's <a href="https://docs.rs/lmdb">documentation</a> contains some comprehensive reference docs, but it doesn't include an introduction, usage examples, nor tutorials.

The project is inactive. Although the latest version was released in March 2018, it contains mostly refactorings and docs fixes (per this <a href="https://github.com/danburkert/lmdb-rs/compare/0.7.2...0.8.0">0.7.2...0.8.0 comparison</a>). And open <a href="https://github.com/danburkert/lmdb-rs/issues">issues</a> and <a href="https://github.com/danburkert/lmdb-rs/pulls">pull requests</a> have languished, including several for panics during common operations and a crash caused by a use-after-free bug.

The author did note in a comment in issue #4 <a href="https://github.com/danburkert/lmdb-rs/issues/4#issuecomment-310928298">on June 25, 2017</a>, "lmdb is pretty much in maintenance-mode; all major features have been implemented, and it works well. There are of course outstanding issues, but overall the crate isn't seeing too much churn. I'd consider releasing a 1.0 version, except that I don't think that's wise when lmdb itself is < 1.0… One thing to remember is that, when it comes to open source projects, inactivity != abandoned and activity != quality."

#### LevelDB

The <a href="https://crates.io/crates/leveldb">leveldb crate</a> is a Rust binding for LevelDB that describes itself as "Almost-complete bindings for leveldb for Rust."

The author has released <a href="https://crates.io/crates/leveldb">22 versions</a>, the first in November 2014 and the most recent in September 2017. The crate uses a conservative version scheme, with the most recent version being 0.8.4.

The crate's <a href="http://skade.github.io/leveldb/leveldb/">documentation</a> includes somewhat comprehensive (but terse and incomplete) reference docs and one usage example, but it doesn't include an introduction nor tutorials.

The project is inactive. There are no changes since the most recent release, and open issues are stagnant, especially <a href="https://github.com/skade/leveldb/issues/6">issue #4</a>, which is a significant usability flaw that makes the crate difficult to use for even common operations.

The author did note in issue #6 <a href="https://github.com/skade/leveldb/issues/6#issuecomment-265120961">on December 6, 2016</a>, "I must admit that a lot of the design around keys was me learning Rust." And in issue #27 <a href="https://github.com/skade/leveldb/issues/27#issuecomment-342073910">on November 16, 2017</a>, he described the project as "the first larger library I wrote on. So you can flag a lot of parts as "learning to build interfaces over C libs". It is (mostly) an exercise."

Finally, he also noted, "But mostly, I stopped developing this library and just maintain whatever patches come into it. This is mostly due to leveldb itself being superseded by rocksdb anyways and me rarely using it :)."

The project appears to have been forked to the <a href="https://crates.io/crates/exonum_leveldb">exonum_leveldb crate</a> for the <a href="https://exonum.com/">Exonum</a> project, but that project switched to RocksDB shortly thereafter per <a href="https://github.com/exonum/exonum/issues/178">issue #178</a>.

### Docs

#### LMDB

Symas publishes <a href="http://www.lmdb.tech/doc/index.html">comprehensive documentation</a> for LMDB that includes an introduction, a "getting started" guide, and a reference.

For example, the <a href="http://www.lmdb.tech/doc/group__mdb.html#ga1206b2af8b95e7f6b0ef6b28708c9127">MDB_cursor_op</a> documentation describes the "set of all operations for retrieving data using a cursor."

However, some of the reference documentation is terse and seems to assume familiarity with the software.

#### LevelDB

Google doesn't appear to publish comprehensive documentation for LevelDB, although the repository contains a <a href="https://github.com/google/leveldb/blob/master/doc/index.md">review of the software's major features</a>.

For example, that document shows code snippets that call `leveldb::Iterator::Seek()`, `SeekToFirst()`, and `SeekToLast()` methods. But there are no links to reference documentation for those methods.

## Features

### Key Types

#### LMDB

Keys are arbitrary byte arrays and are stored in lexicographical order.

#### LevelDB

Keys are arbitrary byte arrays and are stored in lexicographical order by default. Users can also specify a <a href="https://github.com/google/leveldb/blob/master/doc/index.md#comparators">custom comparator</a> that changes the order.

### Value Types

#### LMDB

Values are arbitrary byte arrays.

#### LevelDB

Values are arbitrary byte arrays.

### Iteration

#### LMDB

LMDB supports iterating key/value pairs in lexicographical order. It also supports seeking to a key (or the first key >= a given key). And it supports iteration in reverse order.

#### LevelDB

LevelDB supports iterating key/value pairs in lexicographical order. It also supports seeking to a key (or presumably the first key >= a given key). And it supports iteration in reverse order (although "reverse iteration may be somewhat slower" per <a href="https://github.com/google/leveldb/blob/master/doc/index.md#iteration">LevelDB > Iteration</a>).

## ACID Properties

### Atomicity

#### LMDB

LMDB is transactional and guarantees the atomicity of transactions.

#### LevelDB

LevelDB isn't transactional but does provide a <a href="https://github.com/google/leveldb/blob/master/doc/index.md#atomic-updates">WriteBatch API</a> that writes a set of changes to the store atomically.

### Consistency

#### LMDB

The meaning of consistency for a key-value store is ambiguous, but <a href="http://www.lmdb.tech/doc/starting.html">LMDB: Getting Started</a> notes: "as long as a transaction is open, a consistent view of the database is kept alive." In this case, the word is used to describe a view that is unaffected by subsequent write transactions.

#### LevelDB

The meaning of consistency for a key-value store is ambiguous, but LevelDB provides a <a href="https://github.com/google/leveldb/blob/master/doc/index.md#snapshots">Snapshot API</a> that it describes as providing "consistent read-only views over the entire state of the key-value store." In this case, the word is used to describe a view that is unaffected by subsequent write transactions.

### Isolation

#### LMDB

Per its <a href="http://www.lmdb.tech/doc/">Introduction</a>, LMDB is "fully thread-aware and supports concurrent read/write access from multiple processes and threads." LMDB implements concurrency via a copy-on-write strategy (<a href="https://en.wikipedia.org/wiki/Multiversion_concurrency_control">MVCC</a>) and permits concurrent access by one writer and unlimited readers.

#### LevelDB

Per its <a href="https://github.com/google/leveldb/blob/master/doc/index.md#concurrency">Concurrency model</a>, LevelDB supports synchronization across threads for some operations, although others require external synchronization. It does not support concurrent access by multiple processes.

### Durability

#### LMDB

LMDB transactions are both durable and performant. LMDB uses <a href="https://en.wikipedia.org/wiki/Shadow_paging">shadow paging</a> to improve the performance of write transactions, and Howard Chu notes in this <a href="https://news.ycombinator.com/item?id=7782400">Hacker News comment</a>: "LMDB write performance remains uniform under load."

#### LevelDB

Per <a href="https://github.com/google/leveldb/blob/master/doc/index.md#synchronous-writes">Synchronous Writes</a>, LevelDB sacrifices durability for performance, persisting changes asynchronously by default, although it's possible to configure a write to be performed synchronously.

Durability may not be the highest priority for LevelDB, as its primary Google consumer, IndexedDB, is <a href="https://developer.mozilla.org/en-US/docs/Web/API/IndexedDB_API/Basic_Concepts_Behind_IndexedDB#durable">not guaranteed to be durable in browsers</a>.

## Quality Attributes

### Performance

In a run of [this benchmark](https://github.com/mykmelez/kvbench/), LMDB was approximately an order of magnitude faster than LevelDB to open a database and read entries, while being roughly equivalent to 3x faster to write entries (depending on the type of write), in this [benchmark run on a macOS](https://mykmelez.github.io/kvbench/criterion/report/).

#### LMDB

LMDB read performance is excellent, and its write performance is reasonable and consistent.

LMDB reuses the space freed by deleted entries instead of compacting the store, which avoids the complexity and performance impact of compaction (trading off disk space in some situations).

#### LevelDB

LevelDB read performance is good, with <a href="http://www.lmdb.tech/bench/microbench/benchmark.html">this 2011 benchmark</a> showing improvements over engines like SQLite.

LevelDB "compacts on open by design," which can slow down opening a database and seeking to a key significantly, per <a href="https://github.com/google/leveldb/issues/210">#210</a>.

### Software Footprint

#### LMDB

The <a href="https://symas.com/lmdb/">LMDB product page</a> claims that LMDB comprises "32KB of object code," and its <a href="https://en.wikipedia.org/wiki/Lightning_Memory-Mapped_Database">Wikipedia entry</a> claims that it's 64KB in size. The sample Rust program that uses the <a href="https://crates.io/crates/lmdb">lmdb crate</a> in the <a href="https://github.com/mykmelez/kvbench">mykmelez/kvbench</a> repo is about 73kB larger than a control program.<sup>[2](#footnote-2)</sup>

#### LevelDB

The <a href="https://en.wikipedia.org/wiki/LevelDB">LevelDB Wikipedia entry</a> claims its "binary size" is 350kB. The sample Rust program that uses the <a href="https://crates.io/crates/leveldb">leveldb crate</a> in the <a href="https://github.com/mykmelez/kvbench">mykmelez/kvbench</a> repo is about 207kB larger than a control program.<sup>[3](#footnote-3)</sup>

### Reliability

#### LMDB

According to its <a href="https://en.wikipedia.org/wiki/Lightning_Memory-Mapped_Database#Reliability">Wikipedia entry</a>, "LMDB was designed from the start to resist data loss in the face of system and application crashes. Its copy-on-write approach never overwrites currently-in-use data. [Which] means the structure on disk/storage is always valid, so application or system crashes can never leave the database in a corrupted state. In its default mode, at worst a crash can lose data from the last not-yet-committed write transaction. Even with all asynchronous modes enabled, it is only an OS catastrophic failure or hardware power-loss event rather than merely an application crash that could potentially result in any data corruption."

#### LevelDB

According to its <a href="https://en.wikipedia.org/wiki/LevelDB#Bugs_and_reliability">Wikipedia entry</a>, "LevelDB is widely noted for being unreliable and databases it manages are prone to corruption. Academic studies of past versions of LevelDB have found that, under some file systems, the data stored in those versions of LevelDB might become inconsistent after a system crash or power failure. LevelDB corruption is so commonplace that corruption detection has to be built into applications that use it."

However, a review of the <a href="https://groups.google.com/forum/#!topic/leveldb/GXhx8YvFiig">LevelDB Issue 197 Workaround</a> discussion and some of the <a href="https://github.com/google/leveldb/issues?utf8=%E2%9C%93&q=corrupt+is%3Aissue">issues containing the string "corrupt"</a> in LevelDB's issue tracker suggests that corruption issues are taken seriously and actively investigated.

### Storage Footprint

In a run of the disk space "benchmark" in the [mykmelez/kvbench](https://github.com/mykmelez/kv-bench/) repo, LMDB used roughly 1.5–4x more disk space than LevelDB to store equivalent amounts of data. However, that benchmark employs a hack to measure the space taken by the LMDB and LevelDB datastores on disk, and it isn't clear that the hack is an accurate way to measure disk usage.

#### LMDB

LMDB doesn't compress data on disk, and it doesn't compact datastores, but it does reuse the space freed by deleted entries.

#### LevelDB

LevelDB compresses data on disk using <a href="https://github.com/google/snappy">Google's Snappy compression library</a>, and it compacts datastores.

## References

[LMDB: The Leveldb Killer?](http://banksco.de/p/lmdb-the-leveldb-killer.html)

[Is LMDB a LevelDB Killer?](https://symas.com/is-lmdb-a-leveldb-killer/)

[Understanding LMDB Database File Sizes and Memory Utilization](https://symas.com/understanding-lmdb-database-file-sizes-and-memory-utilization/)

[LevelDB "impl" document](https://github.com/google/leveldb/blob/master/doc/impl.md)

## Notes

### Footnote 1

<sup>1</sup>See Wikipedia's <a href="https://en.wikipedia.org/wiki/Reliability_of_Wikipedia">Reliability of Wikipedia</a> article for much discussion about the reliability of Wikipedia articles generally. This document uses the Wikipedia articles on LMDB and LevelDB as a source—but not the sole source—of information about the two storage engines.

### Footnote 2

<sup>2</sup>490,880 bytes for the LMDB program versus 417,804 bytes for the control program on a macOS system is an 73,076 byte difference. Both programs built using stable Rust v1.30.1 with the "release" profile and then stripped on December 18, 2018.

### Footnote 3

<sup>3</sup>624,900 bytes for the LevelDB program versus 417,804 bytes for the control program on a macOS system is a 207,096 byte difference. Both programs built using stable Rust v1.30.1 with the "release" profile and then stripped on December 18, 2018.
