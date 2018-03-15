---
title: IPC Security Models and Status
layout: text
---

# IPC Security Models and Status

Randell Jesup

Reviewers: pauljt, baku, bkelly, posidron, ehsan

## TL;DR:

* Considerable additional work could be done validating IPC parameters received by the Master process from Content (and other) processes

* More emphasis on Best Practices should be used in IPC work - especially documenting the security aspects in-tree

* There is some cruft in our IPC code left over from B2G, though some of it may need to be kept and updated instead of removed

* We should consider either moving to Mojo, or mirroring the partially-automatic validation of IPC arguments provided by Mojo

* A more in-depth look at how shmem is used might be valuable

* Improved automated fuzzing would be good

* An audit of 12 semi-randomly-chosen IPC protocols is attached (of ~144)

    * One possible security issue found

## Current State:

IPC security is basically ad-hoc, implemented by each IPC protocol implementation.

The basics are that many parts of our system are gatewayed by using Principals, which encode the origin and related information associated with the document. Largely we rely on good reviewers, and that’s more for avoiding sec issues like UAFs.

There are currently many components that send (child-to-parent) a Principal or a PrincipalInfo, but the parent is not really able to know if the child is allowed to use that principal.  We should be able to validate what Principals a Content Process is allowed to use with data stored in the ClientManagerService.  This is being worked on in [bug 1432825](https://bugzilla.mozilla.org/show_bug.cgi?id=1432825) and [bug 1432831](https://bugzilla.mozilla.org/show_bug.cgi?id=1432831), though that just covers a single case; resolving bug 1432831 would provide examples on how to do similar argument validation elsewhere.

As an example of Principal validation, when BroadcastChannel is sending a message, it sends the origin and its principal. The check is done on the parent side. 

Once we have Site Isolation, the danger when Principle ownership isn’t checked is that if a compromised Content Process (somehow) knows a principal that’s valid in another document, it could pass it up, masquerading as that other document.  Without checking if that document is loaded into that Content Process, the Master process will allow the action to proceed with an incorrect Principal.  This can lead to all sorts of information leakage or spoofing. (Note: since a current Content Process could be navigating to a new origin, we can’t really stop a compromised Content Process from getting a Principal for a random origin until we have support for process-per-origin (or set of origins) and necessary process switching code for navigations).

The issue isn’t just Principals, however; all arguments must be checked, and where possible validated against what a given sender should be able to use.

In addition to information leakage, crafting arguments for an IPC message can also lead to considerable risk of sandbox escape - sending an IPC request with invalid arguments to trick the Master Process into taking an action that leads to the master process hitting a UAF or other security issue.  Luckily, even in this case it’s hard to craft an escape, but this greatly increases the possibility.

Securing IPC code is important.  It’s more critical as we move towards site isolation - compromise of a Content Process with 1/4 your tabs (and everywhere you browse in the future in those) is pretty bad, so a sandbox escape is only incrementally worse, but with site isolation a compromised process may be worth much less, unless it can escape.

## Best Practices:

* Firefox [IPC guide](https://wiki.mozilla.org/Security/Sandbox/IPCguide)

* Google’s legacy IPC [security tips](https://www.chromium.org/Home/chromium-security/education/security-tips-for-ipc) (read for full explanations)

    * (paraphrased/subsetted):

    * **Trust only the ****browser**** "****Master" ****(aka “Chrome”) process.**

    * **Sanitize and validate untrustworthy input.**

    * **Whitelisting is better than blacklisting**

    * **Safely handle known-bad input**

        * Don’t RELEASE_ASSERT on bad data

    * **Use and validate specific, constrained types; let the compiler work for you**

        * Use types IPC knows about, try to avoid bare strings and custom serializers

    * **Keep it simple.**

    * **Be aware of the subtleties of integer types.**

    * **Be aware of the subtleties of integer types across C++ and Java, too.**

    * **Don't leak information, don't pass information that would be risky to use.**

        * Be careful with shared memory mappings (specifically tracking their sizes on either side of the IPC channel). Do not store and trust sizes on one side of the channel. Avoid specifying the size when calling Map.

        * (many more in the doc)

* Carefully document lifetimes of objects that can receive IPC messages and ensure (and comment in the code) how we know no more messages can be received for an object

    * Document the valid ranges for parameters, or validation criteria

* Provide guidance on when a Principal should be required for IPC protocols

    * This primarily applies for info going to/from the Master Process, though as we get a more-complex web of Processes this might be needed elsewhere. 

    * An example where it isn’t needed would be GMP webrtc codec processes, which are inherently per-site already and only communicate with a specific Content Process and webrtc connection within it.

* Validate parameters, lengths and existence of any data received from another Process.  

    * It’s less critical to check validity of data received from the Master Process, but even there it may help catch problems and bugs.  Also, in some cases, an attacker might be able to leverage a bug in code in the master to cause it to send a bad message (or race) to the Content Process, provoking a UAF/etc in content they can exploit.  It’s also possible to attack another non-Master process (Content Process, Extension Process, etc) by having the Master relay messages to it, as is often done in Extensions.

        * Compromising the extension process could lead to control or disclosure of all content data

    * Keep lists of things that are associated with a specific connection/Content Process to do validity checks (perhaps typically a Hashmap)

    * Be very careful about overflows with data from the Content Process

* Where possible, give tokens out instead of raw data, if the other process doesn’t actually need to manipulate or examine the data.  This is similar in concept to things like fd’s, but with added validation whenever doing a lookup of a token (again, typically a hashmap, but perhaps not always).  The lookup routine is a good place to put ownership checks and enforce their being checked.

* Fuzzing - we’ve done this before (fuzzing Pickle/etc).  We can probably do more.  

    * Faulty

        * Process bi-directional fuzzer in real-time.

        * Integrated in --enable-fuzzing builds available at TaskCluster.

        * Fuzzes pickled messages and/or pipe states.

        * Runs in multiple instances in our fuzzing cluster at EC2.

        * Upgrade of EC2 instance required which supports HW counters for rr. Important for general test-case reproducibility. There is currently no way to feed a mutated message into Firefox from the outside of the process.

        * Requires targets (i.e mochitests, test-suite) to trigger individual IPC messages.

            * We have no way to know if all IPC messages are hit in our testing

            * We could keep stats on numbers of different IPC messages in debug builds

    * [Meta bug for fuzzing](https://bugzilla.mozilla.org/show_bug.cgi?id=fuzzing-ipc-ipdl) (80+ bugs)

    * Adding fuzzing for every new protocol would be useful.

    * Christoph Diehl has a list of improvements for Faulty, and is also looking at how Mojo uses Google’s fuzzer.

* Discovering dead/unused IPDL APIs. We’ve had unused IPDL APIs left in the browser that could be used as attack vectors.  It would be nice to be able to use automated tools to get rid of such code.

    * As an example, the partial audit found that the DocumentRenderer IPC protocol was unused (and had a security vulnerability), and it has been removed.

## Possible Improvements to our IPC use:

* Provide more support for IPC parameter validation

    * [Bug 1430159](https://bugzilla.mozilla.org/show_bug.cgi?id=1430159) - specified validator for specific IPC protocols

    * [Bug 1325647](https://bugzilla.mozilla.org/show_bug.cgi?id=1325647) - Automatic validators for integers generated by the IPDL compiler 

    * Support classes for checking token/principal ownership

    * Leveraging improved argument checking support in Mojo if we switch from IPC/chromium to Mojo

        * Mojo heavily leverages C++ and types for validation, and also allows custom validators

    * Add the possibility to mark each IPC method with **[principal_needed]** or something similar. Maybe that would be nice to be the default for each method. Basically, each method should have a principal or a principalInfo passed as argument. This would be interesting, if we have the possibility to validate which principal is allowed to run on the content process. 

* We could move to a model closer to the dbus model, where Services are available under Well Known names, and clients can access them safely without worrying about if the target object exists (anymore).  This would help remove a common failure case, where due to timing races an IPC request for a deleted object is received.

* A form of cross-IPC/process "weakptr"

* Possibly rewrite MessageManager (not in JS)?

## Further Investigations

* Examine an example protocol in great detail and document any issues, and solutions (as in [bug 1432831](https://bugzilla.mozilla.org/show_bug.cgi?id=1432831))

    * Use as a guide for looking at others

    * HTTPChannel Init was suggested (non-trivial, complex)

* List all IDPL protocols (~144 ipdl files)

* Document all JS users of MessageManager

# Prior work from Security Assurance Team on sandboxing (pauljt)

[Metabug](https://bugzilla.mozilla.org/show_bug.cgi?id=sandbox-sa) 

The key activities that we have done are:

### IPDL auditing

We’ve been through IPDL in an ongoing manner, but most recently the approach we took was [here](https://docs.google.com/document/d/1YYWGQAqKzBpXPmBnhjAJuCtr8ADtxhz7mVlbAvWorcQ/edit#). Summary:

* Audit message serialisation (param traits).  DONE.See also:  [Bug](https://bugzilla.mozilla.org/show_bug.cgi?id=1315840)  and [Spreadsheet](https://docs.google.com/spreadsheets/d/1sTgA4bOuV0j1bP7_Q_QKsfiNNLfdIg_1CCo4-iB_cfo/edit#gid=0)

    * Some issues were found

    * There were a number which were "might be an issue". The follow up [bug](https://bugzilla.mozilla.org/show_bug.cgi?id=1325670) was filed by DH but then he left the org.

* Audit Recv Methods (look just at the recv function for bugs)

    * This was an ongoing effort for Jhector before he left

    * Bugs are filed under [https://bugzilla.mozilla.org/show_bug.cgi?id=1041862](https://bugzilla.mozilla.org/show_bug.cgi?id=1041862)

    * The challenge here is that auditing this code is very time consuming and requires an understanding of the code/api in question

* Deeper audit diving into where data from recv function goes

    * We’ve only looked at this level in piecemeal approaches or when hunting for dangerous patterns

    * We started trying to organise an [audit](https://docs.google.com/spreadsheets/d/17wvJPTfKto8abz7UD2NoPTebNYoV-dh60ejeCx84Vkw/edit#gid=1569995826) by contacting module owners but had to put this on hold due to the focus on Quantum

## External IPDL Audit

I also used some parental leave backfill to get a [timeboxed audit of IPDL](https://docs.google.com/document/d/1ad9JV6yeRcoqnS-vQR2h3A8avPDL7U4aydmzzHL7lhQ/edit) last year from a contractor (Stephen Fewer). It was only a month though, so very timeboxed.

## Message manager audit

* We’ve been through all the "MessageManager" messages a couple times now, and I’m pretty confident of our coverage here

* One exception is nested protocols, e.g. Web Extension message passing

* Detailed review notes are [here](https://docs.google.com/spreadsheets/d/1YnOFWatdnSBEvDKHLQV4DFngNuwC1Kkb2hZShV1cVx0/edit#gid=35305492)

* Some findings tracked under [here in bug 1040184](https://bugzilla.mozilla.org/show_bug.cgi?id=1040184)

**Other sandbox related auditing activities**

* Audit places where were run remote content in parent process: [bug 1305531](https://bugzilla.mozilla.org/show_bug.cgi?id=1305331)

* Attempt at automatic bounds checking (since this was the most common bug). [Bug 1325647](https://bugzilla.mozilla.org/show_bug.cgi?id=1325647)

## Specific Protocols and IPC messages Audit (jesup)

We have ([currently](https://searchfox.org/mozilla-central/search?q=&case=false&regexp=false&path=*.ipdl)) about 144 separate non-test .ipdl files in the tree.  Looking in them for "manager XXXXXX":  (This search may be comparable to mine below, but I don’t trust it as much - [instances of manager in searchfox](https://searchfox.org/mozilla-central/search?q=manager%20&case=true&regexp=false&path=*.ipdl))

* About 36 managed by PBackground*

* About 9 managed by PBrowser

* 1 PCache

* 6 PClient*

* 8 PCompositor*

* About 23 managed by PContent

* 5 PGMP*

* 2 PImageBridge

* 17 PNecko

* 6 PPlugin*

* 2 PPresentation

* 2 PQuota

* 1 PServiceWorkerManager

* 1 PSpeechSynthesis

* 1 PVideoDecoderManager

* 1 PVRManager

* 2 PWebBrowserPersistManager

Looking at all the methods inside these ipdl files, there are roughly:

* ~1284 async IPCs

    * 129 of these as async __delete__(...)

    * Perhaps ~15-18 are async DeleteMe/DeleteSelf/RequestDelete

* ~124 sync IPCs

* ~19 nested(inside_cpow) (sync and async)

* ~146 nested(inside_sync) sync IPCs

* 1 prio(input) async

* ~10 prio(high) async

…

### Examination of example protocols:

* PVsync

    * Child of PBackground, used between Parent (Master) and Child (Content) processes

    * Only arguments are the current Timestamp on a Notify, and the timestamp rate (which could change I believe), both sent Master->Child

    * Exists for the entire life of the Content process; if Content doesn’t need VSync, it tells the Master to hold off on sending it.

    * Security 

        * The only information exposed through this to the Content process is the VSync rate of the current monitor, and the exact timings of when the VSyncs happen (modulo latency in IPC).

        * The exact timings might vaguely reveal some aspect of varying load on the Master process, maybe, and then only if the child can somehow time them extremely accurately.  I seriously doubt this could be used to convey any significant data even with cooperation of code on the Master side and full control of the Content process (owned).

    * Recommendations:

        * None

* PUDPSocket

    * Child of PNecko of PBackground

        * PNecko for JS-visible PUDPSocket

            * Was planned to move to PBackground also ([Bug 1167039](https://bugzilla.mozilla.org/show_bug.cgi?id=1167039)), but that was deprioritized with the end of B2G

            * Likely PNecko support could be removed

        * PBackground for WebRTC ICE network usage

    * Originally designed to support B2G networked applications, such as Mail

        * Web content was not allowed to use UDPSocket

            * Principal used to check that udp-socket is allowed for that Principal

        * All such uses have been removed with the end of B2G

    * PBackground use was added to support WebRTC’s need to send/receive UDP data and negotiate port usage via ICE (IETF protocol for NAT traversal)

        * WebRTC doesn’t pass up a Principal - Bug [1126232](https://bugzilla.mozilla.org/show_bug.cgi?id=1126232)

            * Not critical since all web content is allowed to create RTCPeerConnections, which can be asked to initiate WebRTC with an arbitrary address, if (and only if) the target address responds to a STUN message which is used as a "Consent" indicator that the target is willing to exchange WebRTC data.

            * No other data in the Master process is accessed via this protocol, only the network, so we don’t need the principal to gate it

        * All traffic on the port is filtered to ensure we see a STUN response before allowing any other traffic

            * Until we see a STUN response, outgoing traffic is only allowed if the packet is a STUN packet, and incoming traffic is blocked

            * This is the security requirement of the ICE & WebRTC protocols

    * All methods are async

        * From Content to Master, there are a number of methods to set up sockets or modify them

            * Bind(), Connect(), JoinMulticast(), LeaveMulticast(), Close

            * These have a number of structures as parameters

            * WebRTC doesn’t use the Multicast methods, but an owned Content process could send them

        * The others are OutgoingData() (i.e. send()) and RequestDelete (asks the parent to send a __delete__ message to the child; avoids races)

            * OutgoingData() takes a structure and a buffer

        * From Master to Content, they’re all responses to Content IPC’s except CallbackReceivedData() - i.e. incoming data on the port

    * Security

        * The only extant use (WebRTC) doesn’t use a Principal

            * This isn’t important since the it can’t be used to access data keyed by the Principal in the master, just network sockets

            * Bug is filed (P4)

        * Can send and receive data to arbitrary addresses

            * STUN filter avoids content (even owned) being able to try to interact with websites or anything using a protocol not associated with realtime communication (e.g. STUN/ICE)

                * Avoids using it to bypass other web controls or to try to DoS sites

                * It can generate STUN packets, but STUN packets are rate-limited to avoid DoS-via-STUN

                    * Rate-limitation is in Content, so could be bypassed

                * An owned process could create a UDPSocket channel without passing a filter string ("stun"), but that will cause the parent to fail Init() - it requires all UDPSockets without a Principal to use a filter (and the only valid filter is ‘stun’)

        * Other than the default IPC read checks (read a string, read a uint32, read a bool, etc), generally there’s no sanity checking of parameters

            * However, when passed to system calls bad data *should* be rejected

            * If there’s an underlying library/OS bug, passing arbitrary data to bind()/connect()/etc could trigger it

            * Additional sanity-checking of parameters is warranted for belt-and-suspenders security against sandbox attacks

            * OutgoingData is an array or an IPCStream (not used for WebRTC)

                * It has an address field, but if a filter (stun) is in place, it isn’t allowed to be used.

    * Recommendations

        * Add validators for AddressInfo, etc per above

            * Highest priority

        * Remove PNecko and non-WebRTC support

            * Unused and non-spec

            * Lets us remove Multicast support

            * Cons: disturbs working code, man-hours

        * [don’t do] Add Principal to WebRTC

            * Removing non-WebRTC support means we can stop using Principal as a flag -- simplifies code in UDPSocketParent

            * Low priority due to no obvious advantage to having the principal

            * Con: man-hours, coding risk - we don’t have access to the Principal down in mtransport; API changes would ripple through the stack to get access to it for no known current benefit

* PTexture

    * Child of PImageBridge or PCompositorBridge or PVideoBridge

    * Virtual reference to a Compositor resource (a gfx texture), which allows dropping or recycling it

        * Used as part of higher-level protocols

    * No arguments

    * No security issues

    * No recommendations

* PWebSocket

    * Child of PNecko

    * Supports implementing the DOM WebSocket API

        * Since network resources must be used in the Master process, the Content process’s WebSocket impl is basically an interface to a WebSocket that lives within the Master

        * WebSocket creation involves many arguments.  Once created, there’s two-way communication over the channel, and the ability to close it.

        * All requests as async

    * Security

        * Methods after creation are simple - send data, or receive data, or close.

            * Master can request client delete itself

            * Only security concerns here are to ensure that any data sent does not extend outside the memory the script has access to, and to avoid UAFs (especially on close/deletion)

        * Open() and OnStart() are where most of the arguments are

            * Open() takes 12 arguments, some of which are structures, OnStart() has 3 strings and a bool

            * None of these are a Principal

                * There are both original and triggering principals on the LoadInfo structure, which is passed in LoadInfoArgs (optionally null)

                * Since it can be null, when converted into a LoadInfo it will succeed and put a nullptr in the nsCOMPtr<>

            * It includes an optional URI, an Origin (string), and an InnerWindowID

                * These get passed on the an HTTP(S)Channel AsyncOpen(), since WebSocket is a protocol run over an HTTP(S) channel

                * Origin is used as a header for the HTTP/HTTPS request, to inform the server what origin URI generated this WebSocket connection request; note that it’s suggested that servers not use this as a form of authentication.

            * Use on HTTP is as insecure as any HTTP connection, and an attacker could use this to feed bad data or commands (or collect information from) a website that uses HTTP WebSockets, via MITM attacks

            * Using WebSocket as a TCP tunnel (WebSocket to Server, Server connects it to a TCP connection to some internal or external resource) is dangerous (to the resource), since a compromised Content (or Master) process could craft attacks on that resource.

    * Recommendations

        * Document when aLoadInfoArgs can be null, what happens, and what it implies - what will succeed and what will fail.

        * Modify LoadInfoArgsToLoadInfo() (in BackgroundUtils.cpp) to verify if the principals are valid for the requesting content process/document

            * Probably also affects all the other AsyncOpen’s (HTTP, FTP, etc)

        * Verify that the InnerWindowID is associated with the principals in to LoadInfo

* PPresentation (and friends)

    * Child of PContent (or PPresentationBuilder or PPresentationRequest)

    * Supports the [Presentation API](https://developer.mozilla.org/en-US/docs/Web/API/Presentation_API), which allows remotely controlling a presentation using DataChannels or https/TCP

        * [http://w3c.github.io/presentation-api/](http://w3c.github.io/presentation-api/)

        * "enable Web content to access presentation displays and use them for presenting Web content"

            * Originally implemented as part of B2G and perhaps also the TV project

            * Apparently enabled in Chrome on Android since Chrome 48 (2015), and Chrome 51 (Desktop)

        * Disabled by default (pref("dom.presentation.enabled", false);)

    * Security

        * There seems to be little or no checking in the Parent of the parameters and that they’re internally consistent, especially the session ID, window ID, tab ID, etc.

            * It’s unclear if this actually creates a security risk here

        * Review [comment from smaug](https://bugzilla.mozilla.org/show_bug.cgi?id=1069230#c141) unaddressed by updates supposedly to address review -- requested the NS_CreatePresentationService() function check if the Presentation API is enabled/permitted

        * Security review of this code has not been done: [Bug 1207051](https://bugzilla.mozilla.org/show_bug.cgi?id=1207051)

    * Recommendations

        * Find out the priority of completing and enabling this API in Firefox, either for Android or Desktop.

            * If there’s no interest in enabling it anytime soon, consider if we want to remove it

                * Removing it will make it harder to bring it back (code rot); the code exists, apparently works and has tests

            * Alternatively ensure that the IPC methods are gated by the enable pref, so that a compromised Content process can’t try to leverage these IPC methods

            * Since the API isn’t even enabled, it’s unlikely that the Content Process can provoke any meaningful problems (other than DoS) by sending Presentation API IPC methods to the Master process, but adding a check of the preference or ensuring these methods can’t do anything is a good safety move

* PMedia

    * Child of PContent

    * Used to manage unique deviceid’s for media devices (via [enumerateDevices](https://developer.mozilla.org/en-US/docs/Web/API/MediaDevices/enumerateDevices), which is speced in W3’s [Media Capture and Streams](https://w3c.github.io/mediacapture-main/#mediadevices) API)

        * Backed by a file mapping id’s to devices on a per-origin basis

            * Each origin has a different ID for the same device, to avoid them being supercookies.

        * Mapping cleared when you clear cookies (per spec)

        * Mapping is temporary (not saved) for private browsing

        * Persistence gated by user permission to use a device

    * Security

        * Can cause Parent to access disk (flat file encoding deviceID/origin triples (id, time, and origin)

            * File lives in profile

        * A compromised Content process could probably look up the device ids for any site, instead of just the current site (if it knows or can guess what sites have called enumerateDevices())

            * Just a privacy leakage in this case (supercookie), which is mostly irrelevant given then control a content process

            * Could be blocked if we have Site Isolation

    * Recommendations

        * Once we have Site Isolation, verify the PrincipalInfo

        * Ensure that the Sync & Storage team knows about this file

* PDataChannel

    * Child of PNecko

    * Involved in Necko redirection to data: URIs

        * Necko code assumes that there’s an IPC channel associated with the redirect

        * The parent class wrapping the IPC channel largely does nothing

    * Security

        * There are no methods other than __delete__

    * Recommendations

        * None, other than possible future Necko rework for isolating parts of Necko might allow us to get rid of this required wart

* PColorPicker

    * Child of PBrowser

    * Requests a Color Picker be displayed by the Master (parent); implementing <input type=color>

    * Security

        * Uses the IPC Manager (PBrowser) to get the owner element, and thus the Outer Window

        * No parameters from the Content Process; one method that passes a picked color from the Master Process

    * Recommendations

        * None

* PRenderFrame

    * Child of PBrowser

    * Represents one web "page"; it's used to graft content processes' layer trees into chrome's rendering path.

        * Only one direct method to trigger a repaint by the Master; no parameters

        * Used to connect Child layout Frames to the Parent

    * Security

        * Nothing direct to this protocol; the connection is passed around by a number of other parts of TabParent/Child

        * Primary security risk would be lifetime management of the PRenderFrameParent/Child objects

        * Per the ipdl file, the PRenderFrame is conceptually owned by the Content Process, and its lifetime is tied to the PresShell in the Content Process.

    * Recommendations:

        * Audit/document the lifetime management (low-ish priority)

* PDNSRequest

    * Child of PNecko

    * Implements DNS requests

        * Most parameters are in the construction request

            * Host, attributes, network interface

        * Passed again to cancel

        * Results are returned asynchronously once complete

    * Security

        * Host and Network Interface are strings

        * None of the parameters are checked for validity on reception

        * Relies on existing validity checks in nsDNSService::AsyncResolveExtendedNative() and functions it calls, like PreprocessHostname()

        * Probably these are well-checked in general.  They are (if internal checks are passed) sent to the OS

        * The ability to specify a network interface might be a subtle risk, especially if we’re trying to route traffic through a VPN interface or proxy.

    * Recommendations

        * Consider validating parameters, or documenting where they’re validated

* PDocumentRenderer

    * Child of PBrowser

    * Implements rendering of a Canvas in the Content Process, and returns the rendered data

    * Security

        * Creation is Master->Content - no issues

        * When rendered, the IPC protocol object is destroyed with the data being returned in the __delete__() method - size and a raw buffer as an nsCString (though it isn’t really a null-terminated string

            * This is concerning, since it opens the door to mishandling the data

            * Depends on DataSourceSurface to avoid going past the real end of the data from the canvas, but nothing checks this

            * Likely information-leakage bug possible if a crafted attack message is received in __delete__() (csectype-bounds)

    * Recommendations

        * Validate that the IntSize(aSize.width, aSize.height) is not larger than the length of the aData nsCString

        * As a result of this review and evaluating a fix to implement the above, it was determined that PDocumentRenderer was unused, and it was instead removed.

* PGamepadEventChannel

    * Child of PBackground

    * Used to notify the Content process (and gamepad) of Update events

        * Controllers added/removed

        * Buttons pressed

        * Joysticks moved

    * Used by Content to tell the gamepad to vibrate (haptic feedback)

        * Parameters are largely numbers (length, intensity, etc) and which controller

        * Master Process appear to not actually implement haptic feedback ([bug 680289](https://bugzilla.mozilla.org/show_bug.cgi?id=680289))

* Security

    * Parameters for Haptic feedback aren’t checked

        * However, they’re not used currently

* Recommendations

    * Add validation of Haptic feedback parameters (even if we’re not currently implementing it), or add a comment stating it’s needed before implementing it.

        * Especially check that the index to the controller is valid

        * There may be an very unlikely information leakage channel if Haptic feedback can be sensed by another Content process (causes joystick "motion", audible to an active mic, etc); however, exfiltrating data is far simpler in other manners, so I see almost no real risk here

