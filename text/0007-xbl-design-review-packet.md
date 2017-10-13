---
title: Design Review Packet - XBL Removal
layout: text
---

# Design Review Packet - XBL Removal

This is the packet prepared for the [browser architecture ‘design review’ process](https://mozilla.github.io/firefox-browser-architecture/text/0006-architecture-review-process.html). Some of the sources used to prepare this are the [XBL and Web Components](https://mozilla.github.io/firefox-browser-architecture/text/0004-xbl-web-components.html) analysis and these [scripts and visualizations](https://github.com/bgrins/xbl-analysis), including [graphs of the amount of XBL in tree](https://bgrins.github.io/xbl-analysis). Prototyping and investigations that have been done to create this packet are tracked in the [de-XBL meta bug](https://bugzilla.mozilla.org/show_bug.cgi?id=1397874).

**Review Chair**: Mossop

**Question to be resolved:** What is the best strategy for removing XBL usage in the Firefox frontend, and the platform implementation of XBL?

## Summary

**Problem Space**: XBL suffers from most of the same [problems as XUL](https://mozilla.github.io/firefox-browser-architecture/text/0003-problems-with-xul.html) and [more](https://mozilla.github.io/firefox-browser-architecture/text/0005-problems-with-xbl). Primarily, XBL is unmaintained, poorly documented, verbose and brittle, adds [extra development cost](https://groups.google.com/d/msg/mozilla.dev.platform/iBoROFkR9V8/1ndvU9QIAgAJ) to new code like Stylo, and creates a learning curve and training cost for new engineers. In interviews with senior platform and frontend engineers, XBL is consistently listed as a problem.

XBL is deeply ingrained into the Firefox codebase. Based on an instrumented run of the mochitest-browser integration test, Firefox creates approximately 11000 elements with XBL bindings and 6000 elements without a binding [[source]](https://bugzilla.mozilla.org/show_bug.cgi?id=1376546). Inside of mozilla-central there are more than 200 unique binding implementations and 40K LOC in those files [[source]](https://bgrins.github.io/xbl-analysis/).

Further, before legacy addon support was removed XBL removal was considered a non-starter, since many popular addons relied on extending or modifying existing bindings. This has also led to extra cost when deprecating features or doing refactorings on individual bindings.

**End State:**

This would allow us to remove around 14-20K lines of C++ code from dom/xbl (not including tests). It also would move the frontend codebase onto a more standardized tech stack, which will benefit from ongoing platform work and allow for integration with existing tooling. More importantly, the resulting codebase would be simpler and easier to modify, debug and maintain. Performance impacts are expected to be small.

**Stakeholders**: Firefox management, Firefox frontend development team, Firefox platform development team, other apps that use XUL such as Thunderbird and SeaMonkey.

## Brief

We propose a multi-prong strategy for incremental XBL replacement in the Firefox frontend:

*   Convert some bindings to JS modules
*   Convert many others to Web Components
*   Convert a few into unrefactored XUL/JS/CSS
*   Flatten the inheritance hierarchy to remove unused bindings
*   Handle special cases

### JS modules

There are some bindings that are not actually widgets - bindings that are only used once, or don’t render any DOM. These can be converted to plain JS objects (either inside a JSM or in a script scoped to the browser window) through a semi-automated process. This will also require manual changes to calling code, so they don’t access the objects as DOM Nodes, and adjust to any API changes that were done in the conversion. Here are a few examples of bindings that fit into this category:

*   [`<preferences> / <preference> / <prefpane>`](https://hg.mozilla.org/mozilla-central/file/tip/toolkit/content/widgets/preferences.xml): Bindings used in about:preferences to control changing preferences from form controls. An outline for how this can be achieved (and a patch that is on its way to being landable) can be found in [Bug 1379338](https://bugzilla.mozilla.org/show_bug.cgi?id=1379338#c6)
*   [`<stringbundle>`](https://hg.mozilla.org/mozilla-central/file/tip/toolkit/content/widgets/stringbundle.xml): A wrapper for string bundle objects with a trivially different API (i.e. `getFormattedString` instead of `formatStringFromName`). We could either build a new object that wraps Services.strings.createBundle and exposes the same API, or change all callers to use `Services.strings.createBundle` directly. Note that the l20n project already has Web Component support.
*   [`<panelmultiview>`](https://hg.mozilla.org/mozilla-central/file/tip/browser/components/customizableui/content/panelUI.xml#l44): Prior art - Mike de Boer’s work on the PanelMultiView included moving the majority of the implementation out of XBL and into a JSM. More details can be found in [this doc](https://docs.google.com/document/d/1D8WSxgyLA13pidT0umTNEmyrBN5njSPGNLsGkyr0oZM/edit).
*   [`<tabbrowser>`](https://hg.mozilla.org/mozilla-central/file/tip/browser/base/content/tabbrowser.xml): Much of the Firefox frontend is driven by this binding. It’s also referenced as gBrowser throughout the frontend codebase. It does render some UI below it, but the majority of its ~6000 lines are behavioral, making more than double the size of the next largest binding in Firefox.
	* This is currently being investigated as a prototype in [bug 1392352](https://bugzilla.mozilla.org/show_bug.cgi?id=1392352)
	* This could be done as a pure JS module, or following the `PanelMultiView` strategy of keeping XBL for construction and destruction but moving the implementation into a JS module

### Web Components

Most of the bindings in Firefox are in [toolkit/content/widgets](https://github.com/mozilla/gecko-dev/tree/master/toolkit/content/widgets), and many of these are self-contained UI widgets that exist at the XUL tag level. For instance, `<image>`, `<label>`, `<toolbarbutton>`, `<checkbox>`, etc. Each has its own methods, properties, internal state, and DOM tree that is currently implemented in XBL. The Web Components spec, specifically Custom Elements, is designed to solve a very similar set of problems, and platform support is actively being worked on. For more detail, see the browser architecture [analysis comparing XBL and Web Components](https://mozilla.github.io/firefox-browser-architecture/text/0004-xbl-web-components.html).

There are some early prototypes demoing this approach using a polyfill in the [preferences UI](https://bugzilla.mozilla.org/show_bug.cgi?id=1392367) and for [`<panel>`](https://bugzilla.mozilla.org/show_bug.cgi?id=1397876). Some  [initial performance numbers](https://bugzilla.mozilla.org/show_bug.cgi?id=1387125) from a benchmark comparing the Custom Elements polyfill with the native XBL implementation show that the polyfill is on par with the native XBL implementation for creating 1000 empty elements from JS.

Shadow DOM provides encapsulation, insertion point features similar to XBL anonymous content, and scoped styles. Waiting on Shadow DOM to land and support XUL documents should not be a requirement for most of the widgets, as long as we are willing to live without insertion points or extra encapsulation (which can be leaky with XBL via CSS direct child selectors and `document.getAnonymousElementByAttribute`).

*   The basic mechanism is a semi-automated conversion from XBL to JS Classes following the Custom Elements spec format, as [prototyped here](https://github.com/bgrins/xbl-analysis/blob/gh-pages/scripts/build-custom-elements.js). Then an element name is registered on the document (i.e. `checkbox`), and when a new `<checkbox>` is created the converted code runs.
*   We should make Web Components support XUL documents instead of converting XUL documents to (X)HTML to gain Web Components support. Supporting Web Components in XUL is much less work than converting XUL to (X)HTML, and it enables us to separate the XBL and XUL concerns, reducing risk. Removing XBL will make it easier to convert XUL to HTML in the future, should we choose to do so; but adding a dependency on XUL to HTML conversion would make it much harder to remove XBL in the present. We should not do that.
*   We should do tree-wide binding-by-binding replacements, as opposed to feature-by-feature. This is to avoid 2 copies of each widget, and to ensure that each new Custom Element supports all the necessary features. This may include making tree-wide changes to consumers if for some reason the Custom Element version can’t remain API compatible
*   We should complete platform support for Custom Elements
*   Add support for XUL documents on platform Custom Elements implementation
*   Create a way to load a set of Custom Elements registrations automatically in all chrome documents
*   Support non-dashed Custom Element names to avoid a project-wide find/replace of `<checkbox>` with `<firefox-checkbox>`, along with not affecting platform code that looks for certain tag names like `panel`
*   We should complete platform support for Shadow DOM to provide support for [bindings that currently use insertion points](https://bgrins.github.io/xbl-analysis/tree/#children) and that continue to need them after a case-by-case analysis

### Unrefactored XUL/JS/CSS

Bindings that are rarely used and insert relatively little content should be unrefactored into their constituent parts (XUL, JS, CSS) and inserted into each bound element.

For example, the `<prefwindow>` binding, which extends the `<dialog>` binding, is used only seven times and inserts relatively little content:

* 4 LOC: a `<windowdragbox>` toolbar containing a `<radiogroup>` “paneSelector” that enables selection of `<prefpane>`s, even though no `<prefwindow>` contains more than one such pane anymore, so this functionality is wholly unnecessary
* 5 LOC: an `<hbox>` “paneDeckContainer” that wraps multiple `<prefpane>`s, which is also unnecessary in our single-pane world
*   19 LOC: an `<hbox>` containing dialog `<button>`s and `<spacers>` that are almost identical to those provided provided by the `<dialog>` base binding

`<prefwindow>` should be unrefactored into its constituent parts as follows:

*   `<prefwindow>`s JS should be converted to a JS module that is imported into each bound element’s document (+1 LOC for each document).
*   `<prefwindow>`s CSS should be loaded into each bound element’s document (+1 LOC for each document).
*   `<prefwindow>`’s vestigial XUL content (`<windowdragbox>`, `<hbox>` paneDeckContainer) should be removed; and its remaining content (`<hbox>` containing `<button>`s/`<spacer>`s) should be merged with its base binding’s content or copied to each bound element;
*   The small number (7) of bound elements means that the additional code size (up to 19 LOC for each element, depending on how much can be merged with the base binding’s content) and maintenance cost is worth it relative to reimplementing the binding as a Web Component.

### Flatten Inheritance Hierarchy

We have a lot of bindings, and use inheritance pretty widely. There are likely cases where we don’t need to - the extra inheritance may have been required for legacy addons, or may have had a purpose in tree at one point but do not any longer:

*   Some could be detected by looking at the [XBL inheritance tree](https://bgrins.github.io/xbl-analysis), where a binding is the only child of its parent, and the parent is never directly used. In this case the child could be ‘folded into’ the parent.

*   The [`<prefwindow>`](https://hg.mozilla.org/mozilla-central/file/tip/toolkit/content/widgets/preferences.xml) binding is a potential example here as well - the ability to render out buttons at the bottom of a dialog could be an option passed into the dialog, instead of using the Unrefactor prong above.
*   Sometimes a binding is created to override a single property - in this case a new attribute / condition could be added to the parent to encode the behavior change.
*   For instance, [`<tabbrowser-tabbox>`](https://hg.mozilla.org/mozilla-central/file/tip/browser/base/content/tabbrowser.xml) is a simple binding can be folded into the existing tabbox module with a new attribute
*   Some bindings are attached via attributes in ways that aren’t able to be emulated in Custom Elements (`<panel type=”arrow”>`) binds #arrowpanel instead of #panel. We can generally migrate to a new tag in that case, but in some cases (like panels) there is platform code looking for the tag name, and it may be easier to fold the arrow functionality into the parent binding.

Making this change does not determine a solution for the parent - it would still need to follow another prong to get rid of XBL itself. Rather, this is a way to reduce the number of bindings that need to be converted, and a chance to refactor away some often unnecessary inheritance for those converted bindings.

### Special Cases

Some of the bindings used in preferences (and throughout the browser) aren’t easy to directly translate to Custom Elements or JS Modules. Generally, [bindings that use the [implements] attribute](https://bgrins.github.io/xbl-analysis/tree/#implements) but don't map pretty closely back to an HTML tag are components that may fall into this category. Each requires further investigation and prototyping before recommending a specific strategy, and may require a custom solution. Here are a few examples:

*   In-content XBL may provide special challenges due to security considerations (Scrollbars, Marquee, Video controls, Click-to-play plugin UI)
*   `<tree>` which implements `nsIDOMXULTreeElement` and has a pretty complicated API
*   For cases like this, we should consult with product / UX and make sure they still want the widget. We may have opportunities to simplify or remove the code instead of just copying the current implementation.
*   `<panel>` which implements `nsIDOMXULPopupElement`. Initial investigations in [Bug 1397876](https://bugzilla.mozilla.org/show_bug.cgi?id=1397876).
*   `<menu>` which opens a native context menu, and may have interaction with XUL overlays
*   Bindings which take advantage of CSS changes to switch to another binding at runtime

## Forces on the system

*   **Performance** is critical, given our commitment to shipping a fast, responsive browser. Initial benchmarking between XBL and Custom Elements is promising, but XBL has been optimized for certain cases like opening new windows and extra platform support or frontend optimizations could add time to the estimate.
*   **Security** is essential for the XBL-in-content replacements, which can’t enable untrusted content to escalate privileges or access private information.
*   **Maintainability** is key to improving our ability to evolve the codebase and avoid falling back into a technical debt trap.
*   **Flexibility** is important to ensure that XBL replacements support the needs of the current (and anticipated future) codebase. Note, however, that replacements don’t have to exactly replicate all XBL functionality.
*   **Reusability** (and the reuse of existing solutions, like Web Components, where applicable) is valuable to reduce the maintenance burden and the risk of obsolescence.

## Dependencies

Custom Elements and Shadow DOM have not yet landed, and Custom Elements doesn’t yet support XUL elements (althought this is being worked on in [Bug 1404420](https://bugzilla.mozilla.org/show_bug.cgi?id=1404420).

As a backup plan, Custom Elements does have a polyfill that may enable us to proceed without native support, but the Shadow DOM polyfill is heavier weight and we wouldn’t propose shipping it. We could mitigate this by not using Shadow DOM (see the Web Components section above).

## Competitive analysis

_Alternative 1: Do nothing_

We believe that XBL is a problem worth solving, as outlined in the summary. But, Firefox works as-is and this is a lot of work. So, what if we didn’t do anything about it?

Firefox would continue to function, and we would continue to develop it. Firefox development would be slower, however, because XBL is harder to use and evolve and it doesn’t always [integrate well with the devtools](https://bugzilla.mozilla.org/show_bug.cgi?id=1360072), so we would spend more time implementing features in XBL or working around its limitations.

Gecko development would also be slower, because XBL adds complexity in the frame constructor, among other places. In particular, implementation of Shadow DOM would take more time, since it would have to account for situations in which both a shadow DOM and XBL are active at the same time.

Removal of the old style system will be delayed until Stylo adds support for XBL, which means that Mozilla will continue to ship Firefox with two separate style systems until that time (adding complexity and footprint).

_Alternative 2: Get rid of simple cases, continue to use XBL for the harder cases_

If we identify the simpler bindings and migrate them with one of the proposed prongs, we could leave the more complex bindings in tree. This would narrow the scope of this proposal. However, it would introduce a complexity cost of having multiple approaches to UI components in tree, and wouldn’t allow us to achieve the end goal of removing the XBL implementation.

_Alternative 3: Do more of the work a window at a time vs a binding at a time_

The proposal we suggest is ‘horizontal’ replacement - that is, replace each individual binding entirely, across the tree. Another approach would be a ‘vertical replacement’ - that is, replace all bindings in a window, one window/feature at a time. For instance, convert the entirety of about:preferences, but don’t worry about using the converted bindings in other features. The drawback of this approach is having two implementations of a binding until every window has been complete, and keeping these in sync would require ongoing work.

_Alternative 4: Rewrite the frontend using a different stack_

Instead of converting XBL bindings to JS and Web Components, we could rewrite the frontend (probably feature-by-feature) using a modern library like React. There are some advantages to this approach, including better developer ergonomics (which improves developer efficiency), better application architecture (which improves application quality), and ecosystem effects. But a rewrite would take much more work, entail much greater risk, and entrain a separable set of concerns (in particular, XUL replacement).

We do think it's reasonable for teams to consider rewriting individual features using a library like React when other factors justify a rewrite, for example in the case of [debugger.html](https://github.com/devtools-html/debugger.html). And it may eventually become reasonable to undertake a complete rewrite of the application. XBL removal may make that easier. But we think XBL replacement is worthwhile enough by itself to justify pursuing a lower-risk strategy for its removal.

_Alternative 5: Convert the frontend to HTML (removing XBL in the process) without rewriting it_

We could remove XBL while converting the frontend to HTML—like alternative #4—without rewriting the frontend in the process. In this case, we’d convert XUL to its equivalent HTML, conserve the existing JS/CSS implementation, and convert XBL bindings to Custom Elements, JS modules, etc. as we’re proposing to do in this document.

Doing this would achieve both XBL and XUL replacement, and its cost of XUL replacement would be less than the cost of completely rewriting the XUL implementation in alternative #4.

But the cost of replacing XBL would be the same as the cost in our proposal. And the risk would increase with the increased scope. Also, this approach is quite similar to replacing XBL and then replacing XUL, modulo any cost of supporting Web Components (and any other necessary web technologies) in XUL.

Finally, while there is relative consensus that XBL should be replaced, XUL replacement is more controversial. We think it makes more sense from strategic and risk standpoints to separate these concerns, tackling each of them in turn, starting with XBL.

