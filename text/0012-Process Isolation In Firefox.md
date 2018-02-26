Process Isolation in Firefox

Randell Jesup, Mozilla Browser Architecture team

## NOTE: this document is a Work in Progress!


# OVERVIEW

We’ve recently moved a number of parts of Firefox into separate processes (e10s, multi-e10s, GPU process, compositor, WebExtensions process, GMP).  This has produced some serious improvements in both security (sandboxing content, codecs) and stability (restarting GPU process when a GPU driver crashes, etc).  This project is to evaluate how much further we can push process isolation to further improve these things.

### Problems:

* We have large processes, running many unrelated items of highly varying security and stability properties.   A single bug (including in OS drivers) in many cases will take down either a major part of your tabs, or the master process and by extension the entire browser.

* In a related concern, a single exploitable bug gives access to a large part of the browser.  Even if it’s in the Content process, it can give access to ¼ of your tabs, and because Content processes have very wide needs to access data and communicate with the Master process, the possibilities for either sandbox escape or information leakage are quite high.

* Features and capabilities often have code strewn across various parts of the tree, increasing the maintenance cost and risk of unrelated changes breaking them.

There are some secondary benefits we hope to achieve by doing this, such as decoupling parts of the system and providing more-stable interfaces to them, as well as easing some aspects of unit testing.  There may be some advantages in build times or cost of maintenance; we’ll see.

There are costs: development time, memory use, and performance all can or will be negatively impacted.  Part of this project is to quantify these impacts (and hopefully reduce them) in order to guide the decisions on how far to push this process.

Chrome/Chromium has been doing similar work recently on "[Servicification](https://www.chromium.org/servicification)".  This is related to their slowly replacing classic ipc/chromium-based IPC with “[Mojo](https://chromium.googlesource.com/chromium/src/+/master/mojo/README.md)”; a new IPC implementation (and ipdl-like layer) that has improved performance compared to classic Chromium IPC.  Note that some ways Mozilla uses IPC might avoid some of the performance costs in Chrome (multiple channels, PBackground and the like, for example) - but we haven’t yet assessed how much overlap there is between Mojo and our additions to Chromium IPC.  It may be that with some smaller modifications our use of Chromium IPC will be as efficient (or nearly so) as Mojo.

Chrome is also working on [Site Isolation](https://www.chromium.org/developers/design-documents/site-isolation), to avoid a compromised Renderer from breaking cross-origin isolation (more details below).

As part of this work, since it affects the performance impact of Process Isolation, we plan to explore the potential of adopting Mojo for Mozilla IPC (either wholesale, or progressively as Chrome has been doing).

Alternatively, and maybe more interestingly, we could look at using a Rust IPC crate and some added interface logic. <Need some more concrete suggestions/pointers here>

Note: we don’t want to just "follow" Chrome’s lead, but to do what’s smart for the users and Firefox, whether it’s similar to Chrome or not.  Leapfrogging them in some manner would be great, but is not a requirement.  If we can leverage work or research they’ve done to reduce our cost or risks, great.

# GOAL

* Develop a browser-wide direction for Process Isolation:

    * How much Servicification should we do?

    * How many Content Processes should we use?

    * Should we consider a Chrome-like Process-Per-Origin or Process-Per-Iframe model?  What are the implications of this?

# Tentative Work Plan

1. Measure the overhead of using Process Isolation:

    1. Memory cost for various scenarios (which largely depend on what part of firefox code need to be initialized/used in the process - XPCOM, etc)

        1. Memory overhead for a Process on Win10 x64 appears to be circa 7-10 MB (Private) for a process with minimal XPCOM inited, IPC, etc.  The GPU process (which also loads GPU drivers, etc) is on the order to 20MB, and anecdotally the GMP process uses on the order of 10MB, which is in line.

    2. Performance cost - latency and throughput of calls through IPC (with classic IPC and Mojo, and perhaps a Rust IPC crate)

        2. Still to be measured

    3. Evaluate if we can make Chromium IPC as efficient as Mojo

        3. I suspect we can; the main perf advantage Mojo has on classic IPC is one less thread-hop per message (2 less per round-trip - 4 vs 6).  The overall code is probably a lot "cleaner", but it would be a fair bit of work to convert over, though probably much of it could be hidden by ipdl.

        4. We believe that Mojo’s shutdown code may be more robust/better engineered than IPC’s; shutdown has been a common source of crash/security bugs.

        5. We think it might be possible in some special(?) cases to avoid a thread hop on the receiving side as well as the sending.  Mojo does not do this.

    4. Startup cost - cost to browser startup time

        6. Unknown.  Need to measure the cost in time to start a minimal process, and see how many we need at startup.  We may be able to partly overlap creation and startup of service processes.  Expectation is that this will not be a blocker.

    5. Service startup time cost - cost on first use of a service which requires spawning a Process

        7. Evaluate using "embryo" processes to allow the forked processes to be smaller but not pay new relocation/etc costs (i.e. don’t fork the Master process and all that entails when we spin up a process/service).  (B2G did something like this with [Nuwa](https://bugzil.la/nuwa).)  Note that this doesn’t help Windows apparently, and there was some sort of issue with this on Mac - these need to be examined/verified.

2. Analysis of Process Isolation

    6. Analysis of the code maintenance impact

    7. Analysis of the stability impact

    8. Analysis of the security impact

    9. Analysis of embryo processes

        8. Per-OS

    10. Analysis of IPC options

        9. Update/improve current Chromium IPC

        10. Mojo

        11. Rust IPC

    11. Android analysis - android will likely require different tradeoffs

3. Develop a preliminary list of potential subsystems/features to consider for Isolation

    12. Necko is already planning to Isolate network socket access and protocol code (and some crypto code) after 57 or 58 -- [Bug 1322426](https://bugzilla.mozilla.org/show_bug.cgi?id=1322426)

        12. They expect to land code in 61 behind a pref, and enable it in release in 63.

            1. This will not be happening in the near term, due to staffing changes.  It’s not yet clear when or if this will occur.

    13. Video camera capture code (and screensharing) is another prime target, as it’s already remoted over IPC even when running non-e10s.  The way this works is very similar in principle to Chrome’s remote-services-over-Mojo approach.

    14. Places in particular, eventually profile data access in general. This pushes storage asynchrony to the process boundary and decouples lifetimes (background storage and syncing, faster 'startup' and relaunch, with Places possibly living longer than a crashed browser). Future technology refactors could make this standalone process reusable outside of desktop Firefox. No bugs filed yet. 

    15. Font and/or Image code to avoid or reduce duplication of data between Content processes and the Compositor.

    16. Printing

    17. PDF display/PDFium

4. Look at the Content Process state and model (mostly this has been done in the e10s team)

    18. How far do we want to push the model towards Chrome’s 1-per-origin/iframe model?

        13. Probably not as far… note however that Chrome has closed the gap we created with them on memory use. [reference?]

        14. Even Chrome can’t get away with their stated goal (yet?)

            2. While site isolation with a small number of sites is in the 15-20% range, this is largely because of the base overhead of Chrome (Master process, GPU processes, [zygotes, ](https://chromium.googlesource.com/chromium/src/+/lkcr/docs/linux_zygote.md)etc).  WIth  17 tabs (a mixture of simple and complex sites), the measured overhead was about 50%.

    19. How much does servicification help reduce Chrome process overhead (avoiding N instances of things)

        15. It may not help a lot

    20. How much can this work help [sandbox hardening](https://www.google.com/url?q=https://wiki.mozilla.org/Security/Sandbox/Hardening&sa=D&ust=1507349372899000&usg=AFQjCNHD1aRxpBa1fuJNcVKxytjULX6krA)?

        16. Main Process will still need file access probably

5. Very speculative: examine if the current Master Process could be moved to be a child process of a thin Master Process, allowing restarts on Master Process crash without reloading all the running Content Processes.

    21. **Very** likely this is too hard and/or too much work.  There’s lots of state stored in the Master on behalf of the Content processes; the reconnect operation between Content and restarted Master would be considerably complex.

# Previous work

* GMP

    * Very tight sandbox, one sandbox per-origin

* E10S

* GPU process

* Compositor

* WebExtensions

* Examination of the options for sandboxing Audio capture and playback, as well as other parts of the Media code: [Media, WebRTC and Audio Sandboxing Plans](https://docs.google.com/document/d/1cwc153l1Vo6CDuzCf7M7WbfFyHLqOcPq3JMwwYuJMRQ/edit)

* Background docs (needs to be updated but some useful info maybe)

    * [https://wiki.mozilla.org/Security/Sandbox/Hardening](https://wiki.mozilla.org/Security/Sandbox/Hardening)

    * https://wiki.mozilla.org/Security/Sandbox/Process_model

# Chrome’s Servicification and Mojo

Chrome has been discussing Servicification since roughly early 2016, and major work on it has begun this year (2017).  This is the primary document: [Chrome Service Model](https://docs.google.com/document/d/15I7sQyQo6zsqXVNAlVd520tdGaS8FCicZHrN0yRu-oU/), and this is the primary root of the Servicification work: [Servicification](https://www.chromium.org/servicification).

An example of one item currently being moved to a Service is Video Capture: [Design Doc](https://docs.google.com/document/d/1RLlgEdvqRA_NQfSPMJLn5KR-ygVzZ2MRgIy9yd6CdFA/edit#heading=h.qjpzx8d6k7t5) and [detailed plans and measurements](https://docs.google.com/document/d/1Qw7rw1AJy0QHXjha36jZNiEuxsxWslJ_X-zpOhijvI8/edit#).  Another which I think has been completed is the [Prefs Service](https://docs.google.com/document/d/1JU8QUWxMEXWMqgkvFUumKSxr7Z-nfq0YvreSJTkMVmU/edit).

[Mojo](https://chromium.googlesource.com/chromium/src/+/master/mojo/README.md) consists of a set of common IPC primitives, a [message IDL format](https://chromium.googlesource.com/chromium/src/+/master/mojo/public/tools/bindings), and a bindings library (with generation for a number of languages; undoubtedly we’d need to add Rust binding generation -- [C++ bindings are here](https://chromium.googlesource.com/chromium/src/+/master/mojo/public/cpp/bindings)).  Mojo has been measured in Chrome as being about ⅓ faster than classic IPC, and produces ⅓ less context switches.  (Probably these two facts are related, and their [performance analysis](https://chromium.googlesource.com/chromium/src/+/master/mojo/public/cpp/bindings) indicates that not having to "hop to the IO thread" is part of why it’s faster, which makes sense.)

One thing we plan to experiment with is seeing if (leveraging our IPDL compiler) we can fairly transparently replace (some?) existing Chromium IPC channels/messages with Mojo.

Chrome has docs on [how they move legacy IPC code to Mojo](https://chromium.googlesource.com/chromium/src/+/master/ipc#Using-Services).  This is a (somewhat dated) cheat sheet on [moving code from IPC and Mojo](https://www.chromium.org/developers/design-documents/mojo/chrome-ipc-to-mojo-ipc-cheat-sheet).

One interesting tidbit from that cheatsheet: 

**IPC**

IPCs can be sent and received from any threads. If the sending/receiving thread is not the IO thread, there is always a hop and memory copy to/from the IO thread. 

**Mojo**

A binding or interface pointer can only be used on one thread at a time since they're not thread-safe. However the message pipe can be unbound using either Binding::Unbind or InterfacePtr::PassInterface and then it can be bound again on a different thread. Sending an IPC message in Mojo doesn't involve a thread hop, but receiving it on a thread other than the IO thread does involve a thread hop.

Mojo has extensive support for message validation: 

Regardless of target language, all interface messages are validated during deserialization before they are dispatched to a receiving implementation of the interface. This helps to ensure consistent validation across interfaces without leaving the burden to developers and security reviewers every time a new message is added.

# Overhead Details

    1. Memory use.  Content Process overhead is tracked in [Bug 1436250](https://bugzilla.mozilla.org/show_bug.cgi?id=1436250).  It’s measured both with a small patch to dump the ASAN heap state on command, and using a DMD build with specific environment options.

        1. Minimal process (with IPC)

        2. With XPCOM

            1. 7-10MB

        3. Full Content process

            2. 25-30MB (varies by OS, memory model (32 vs 64), and system details (fonts, etc))

            3. Content Process overhead is critical for Site Isolation

    2. Performance -- measure each with small messages, large messages, and shared-memory payloads (anything else?)

        4. Latency

        5. Messages/second

# Security implications

Moving services and other code into sandboxed processes should generally increase the resilience of the system to security bugs.  In particular, compromising the system in one sandboxed process will require a sandbox escape of some sort to leverage that into full or increased control, and generally will only expose data already flowing through the Service to the attacker - and for many sandboxed processes, exfiltrating compromised data would be much harder.  

How hard exfiltration would be depends on what sort of data flows through the Service, how locked-down we can make the process, and if the output of the process normally is in some way visible to content - for example if the process did image decoding, then data could be exfiltrated by using it to break cross-domain restrictions (such as taking the content of image A in domain X, and outputting it in place of image B from domain Y (the attacker’s domain), allowing the attacker to render it in a canvas).

Another way that isolation will help security is by separating memory pools - memory errors such as UAFs can become much harder to exploit if you can’t put the memory system under load (forcing reallocation of the memory with content you control, for example).  In a Content process with JS running (and tons of other stuff), this is often not too hard; in an isolated Service it might be very hard indeed.

Once a Process is compromised, leveraging that into an exploit requires escaping into other Processes (using further bugs), or leveraging an OS bug.  How hard that is depends on the OS and how locked-down we can make these Processes.  "Small" processes may have much smaller OS attack surfaces, though this might be tougher to do on (say) Windows due to granularity of permissions.

# What Chrome does

Chrome doesn’t actually use a Process-per-tab (or origin), though many people believe it does: see [Peter Kasting’s post from June](https://plus.google.com/+PeterKasting/posts/TC4ACtKevJY).  (That was partially in response to some of our announcements around E10S.)  The number of Render processes they use depends on the available memory - though it sounds like they have bugs there, and that may cause them to overrun and slow down the user’s system.

Chrome is working on [Sit](https://www.chromium.org/developers/design-documents/site-isolation)[e Isolation](https://www.chromium.org/developers/design-documents/site-isolation).  Part of this is putting iframes OutOfProcess from the main page renderer, but more generally it’s about not using a renderer (Content process) for more than one origin.  This has some serious downsides if taken to an extreme, and currently they’re planning to do this only for "high value" sites.  (It’s unclear what “high value” means here; one presumes banks, paypal, and other especially juicy targets.)

As mentioned above, Chrome is increasing the number of non-Render processes they use as part of Servicification.

The [Chrome Process Model](https://www.chromium.org/developers/design-documents/process-models) document is useful, but very out of date with current details - for example, the [Site Isolation](https://www.chromium.org/developers/design-documents/site-isolation) work they’ve done.   Some of the code for all these decisions is [here](https://cs.chromium.org/chromium/src/content/browser/renderer_host/render_process_host_impl.cc?rcl=98c9b1d4fc903ab9dee06a98e7497b11de760449&l=341). 

# What Edge does

In "[Browser security beyond sandboxing](https://blogs.technet.microsoft.com/mmpc/2017/10/18/browser-security-beyond-sandboxing/)" Microsoft goes into detail on a Chrome vulnerability, but also highlights some of what they’ve done - in particular [Arbitrary Code Guard](https://blogs.windows.com/msedgedev/2017/02/23/mitigating-arbitrary-native-code-execution/#HGTKcYGQeuOxvYIB.97): “ACG, which was introduced in Windows 10 Creators Update, enforces strict Data Execution Prevention (DEP) and moves the JIT compiler to an external process. This creates a strong guarantee that attackers cannot overwrite executable code without first somehow compromising the JIT process, which would require the discovery and exploitation of additional vulnerabilities.”  Their post introducing ACG is [here](https://blogs.windows.com/msedgedev/2016/09/27/application-guard-microsoft-edge/).

In more detail: "To support this, we moved the JIT functionality of Chakra into a separate process that runs in its own isolated sandbox. The JIT process is responsible for compiling JavaScript to native code and mapping it into the requesting content process. In this way, the content process itself is never allowed to directly map or modify its own JIT code pages."

This suggests that we should consider moving the JIT itself out of the main process space, and just share the result of the JIT back with the Content process requesting it.  Since we already do JIT on a non-MainThread, the main issue here is probably shared-memory management.   There will be some perf tests to do on this to validate if this is feasible within our current overall architecture, but it seems possible.  This is being tracked in [Bug 1348341](https://bugzilla.mozilla.org/show_bug.cgi?id=1348341), and Tom Ritter has been investigating in [this doc](https://docs.google.com/document/d/13HwuPNxabIONDVTKInMkkIitXxcB_kmBjMTjA7dHtSE/edit).

Note that if the JIT process is compromised, anything running through it is as well, and any content processes it provides code for.  This would imply that JIT processes may need to be tied to a single requesting Content Process to avoid stepping backwards in security here (and this also increases the potential memory cost).

