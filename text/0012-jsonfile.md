# A brief analysis of JSON file-backed storage

Several components in Firefox — including libpref, XULStore, autofill, and logins — follow a general pattern:

- Store data in memory, usually as a JS collection.
- Load that data from disk during initialization.
- Persist that data to disk in its entirety, usually serialized as JSON, at some point after each write, and/or at shutdown.

[JSONFile.jsm](https://dxr.mozilla.org/mozilla-central/source/toolkit/modules/JSONFile.jsm) is a centralized implementation of this pattern. It supports Promise-based asynchronous loading, synchronous loading, and flushes asynchronously, defaulting to 1500ms after each change. libpref flushes after 500msec.

This approach is well suited to data that:

- Changes infrequently.
- Is relatively small.
- Is only ever directly read or written by a single thread.
- Needs to be kept entirely in memory for reads.
- Has a straightforward JSON encoding (encoding binary blobs, such as images, in base64 is relatively inefficient and typically avoided).
- Has limited durability requirements.

Advantages of this approach:
- It's relatively simple, with serialization and deserialization being the only steps.
- Because the file is rewritten frequently, there is some hard-to-measure resilience to gradual file corruption.
- The file on disk is — unless compressed — human-readable and editable, which is convenient for development, test, and support.
- Automatic opt-in whole-file compression is offered by JSONFile.
- The in-memory data can be queried and manipulated synchronously and quickly, assuming the representation is a good match for retrieval patterns.
- Development is 'natural' for front-end developers — get things working with only in-memory data, then add persistence on top.

Disadvantages:
- Versioning is often an afterthought: engineers rarely think to version in-memory data formats.
- Change tracking is almost always an afterthought, thanks to the 'natural' development process: adding syncing later becomes difficult.
- Users feel relatively empowered to edit readable files on disk, which can result in persistent state that was never created by the component's own logic. The use of compressed formats makes this less likely.
- Frequent writes will cause the entire file to be written to disk repeatedly, [which causes complaints about SSD impact](https://bugzilla.mozilla.org/show_bug.cgi?id=1304389).
- The entire file typically needs to be read into memory to be used, which increases memory footprint and can impact startup time.
- Writes at shutdown harm the user experience and don't happen during a crash.
- In order to achieve atomic file writes, the entire file contents briefly exist twice on disk, which could be problematic on mobile platforms for large data sets.
- The only approach to cross-process use is duplication of all or part of the in-memory data, which can increase memory footprint. It's not possible for multiple processes to collaborate on the same data via the filesystem. This is why Firefox for Android still uses the SQLite implementation of `nsILoginManagerStorage`: it allows the Android-side `ContentProvider` code to read and write saved logins without worrying about Gecko, which nominally owns the backing storage, but has a shorter lifetime than the enclosing Android code. Using desktop's JSON-backed storage on Android would require launching Gecko on each sync, using messaging for safe access to the file contents.
- The in-memory object is essentially a non-transactional write-back cache. This has several issues:
  - It makes isolation (readers can't see in-progress writes) and atomicity (all writes complete or none do) difficult: we typically mutate the in-memory object directly, and so an exception can cause partial changes to 'commit', and readers will see each change as it is applied.
  - Similiarly, timed flushing makes consistency difficult: it's possible for only some of a set of asynchronous writes to be flushed to disk because the flush beat the last few writes, leaving the data inconsistent after a crash.
    
    JSONFile.jsm advises that callers do all of their work synchronously on the main thread to avoid the possibility of concurrent readers or partial writes:
    
    > The raw data should be manipulated synchronously, without waiting for the
    > event loop or for promise resolution, so that the saved file is always
    > consistent. 
     
    This can cause jank.
- Synchronization of data stored in this way takes some careful thought. Syncs typically take hundreds of milliseconds or more, and involve asynchronous network operations, which makes exclusive synchronous access to the in-memory data between reads and writes infeasible.
- There is a tension between durability (that is: writes that complete are permanent) and performance. We typically choose not to flush the file immediately and synchronously after every change, but not doing so leaves a window in which a crash would cause data loss. By default, that window is 1.5 seconds, plus the time needed to write and atomically switch the files. This forces careful consumers to manually flush if they want their writes to stick, which is a bad pattern.
- In-memory objects lack the sophistication of most databases. This leads to front-end features building their own [simple query engines](https://dxr.mozilla.org/mozilla-central/source/toolkit/components/passwordmgr/storage-json.js#296) for [finding records by linear search](https://dxr.mozilla.org/mozilla-central/source/browser/extensions/formautofill/FormAutofillStorage.jsm#1081).
- Similarly, these stores must reinvent (or contribute to JSONFile) their own:
  - Versioning and upgrade logic.
  - Validation and schema checking.
  - Concurrent access patterns.
  - Write coalescing/async update patterns.
  - Tooling, if existing general-purpose JSON tools (*e.g.*, `jq`) are not sufficient.
  - Backup, if any.
  - Indexing, if any.
  - Datatype serialization (*e.g.*, timestamps).
- Finally, the use of `JSONFile.jsm` is intimately linked to Gecko; it's a poor choice for code that will later need to be deployed on other platforms.
