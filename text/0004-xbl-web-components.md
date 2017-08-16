
# XBL and Web Components

XBL is a technology used to implement reusable widgets in XUL. Web Components are a set of specifications that provide modern and standards-based alternatives for doing the same thing with HTML. For the purposes of this document, "Web Components" refers to the [Custom Elements](https://w3c.github.io/webcomponents/spec/custom/) and [Shadow DOM](https://w3c.github.io/webcomponents/spec/shadow/) specs, which map relatively closely to how the Firefox frontend is using XBL.

What follows is an initial analysis of what a migration away from XBL and to Web Components may look like. We will need to spend more time prototyping this within the Firefox UI to get a clearer picture about the viability of a migration.

## What bindings are we talking about?

Most of the relevant bindings are declared in [toolkit/content/widgets](https://github.com/mozilla/gecko-dev/tree/master/toolkit/content/widgets).

This analysis is focused on widely used and relatively small XBL components that could potentially be swapped out more-or-less in place with corresponding Web Components. For instance: buttons, labels, input fields, etc.

Components that are only used once or are implemented mostly in C++ are ignored in this analysis. For instance: tabbrowser, trees, etc. Most of the same principles that apply to smaller components should also apply to them, but we may also consider different options for moving them away from XBL.

## Auditing the XBL feature set

From the [Introduction to XBL](https://developer.mozilla.org/en-US/docs/Mozilla/Tech/XUL/Tutorial/Introduction_to_XBL), a binding has five types of things that it declares: Content, properties, methods, events, and style.  But it's helpful to look at specific features used in Firefox and comparing how they map back to Web Components.

Which bindings use which features is visualized at https://bgrins.github.io/xbl-analysis/. The scripts to generate that tree and also to semi-automate translation of bindings can be found at https://github.com/bgrins/xbl-analysis.

### XBL features that map to Web Components features

1. `<property>`: These map to JS getters and setters on the Custom Element class (‘val’ name is magic in xbl, would need to keep same name to translate directly)
2. `<method>`: These map to functions on the Custom Element class
3. `<constructor>` This maps to connectedCallback on the Custom Element class (which is called when the element is inserted into a document).
In XBL, the constructor fires after the element is inserted into the document (since the binding is attached through a CSS property).  However, the constructor will wait some additional time to fire, unless if the element is accessed via JS in which case it will run immediately
4. `<destructor>`: These map to `disconnectedCallback` on the Custom Element class
5. `[extends]`: A way to extend the functionality of another binding, i.e. `extends="chrome://global/content/bindings/general.xml#basetext"`. On a surface level this appears to map directly to how the ‘extends’ keyword in ES6 classes behavior with Custom Elements, but more investigation is needed.  There’s a small set of tags where you can do `extends="xul:button"` but that’s aliasing the same thing.
6. `<handlers>`: Can be implemented with addEventListener or possibly a helper on the base element i.e:
```
helpers: {
  "click": () => { /* bubble by default*/ }
  "focus|capturing": () => { /* copy in xbl implementation */ }
}
```
7. `[inherits]`: Is a way to declaratively copy attributes from a binding parent to content and keep them in sync, similar to the `<observes>` element. For example, this copies the feedURL from the parent into the 'value' field of the label and also observes future changes: `<feed feedURL="http://foo.com"><xul:label xbl:inherits="value=feedURL,tooltiptext=feedURL"></feed>`. The same thing could be built on top of Custom Elements using a MutationObserver and some glue code to set it up in the `connectedCallback`.
8. Scoped CSS Styling: this can be done with Shadow DOM
9. Insertion Points: XBL and Shadow DOM appear to use similar models but different keywords. Further investigation is needed.

### XBL features that don't map Web Components features

1. `[implements]`: This is a way for elements with XBL bindings to implement an XPCOM interface.  XBL also auto generates the Qi bits for you. One example usage is that labels implement complex cropping behavior via `implements="nsIDOMXULLabelElement"`, but there are many others (panels, menus, scrollbars). There are around 39 instances of this feature, and this will need to be handled on a case-by-case basis. But we have some options:

  * Write chrome-only webidl bindings for each of the individual nsiDOMXULFoo classes
  * Replace things like `implements="nsIObserver"` with just a separate object that in fact implements it
  * For some input elements like checkboxes, we could get rid of the xul interface since the corresponding HTML element has the same behavior already

2. Extension Compat: For instance, the Tab Center test pilot addon causes tab browser to become vertical via inheritance with some changes.  This allows us to use the same tag name / markup so that it doesn’t need to re-write CSS. However, with the migration to Web Extensions this won’t be necessary in the general case.
3. Binding via CSS: `-moz-binding` allows a XBL binding to be attached or detached to any element through CSS.  Web Components would be tied to HTML (based on the particular tag being created). This feature is used in [at least one place to swap an element to a different binding when setting a property](http://searchfox.org/mozilla-central/rev/7cc377ce3f0a569c5c9b362d589705cd4fecc0ac/toolkit/content/widgets/text.xml#47), but we can work around it.

## XBL Usage in Firefox

How many times are individual bindings used?  We've written a module to track how often XBL (and XUL) are used, and a modified version of the test runner that keeps a count throughout the mochitest-browser suite. [A sorted list of used bindings is attached on the bug](https://bugzilla.mozilla.org/show_bug.cgi?id=1376546), but here are the top few bindings:

    url("chrome://global/content/bindings/text.xml#text-label")
    url("chrome://global/content/bindings/general.xml#image")
    url("chrome://global/content/bindings/toolbarbutton.xml#toolbarbutton")
    url("chrome://global/content/bindings/menu.xml#menuitem")
    url("chrome://global/content/bindings/scrollbox.xml#autorepeatbutton")
    url("chrome://global/content/bindings/button.xml#button")

## Performance Considerations

XUL and XBL both use fastload and they both have prototype cache, so loading them and opening new windows should be quite fast. Custom Elements (and Shadow DOM) don't have anything like that right now. If we want to use them in browser chrome, we may need to figure out how to support things like this.

The first step is getting some measurements to compare things like element creation time, memory usage, etc. This work is being tracked in [Bug 1387125](https://bugzilla.mozilla.org/show_bug.cgi?id=1387125).

## Notes

* Custom Elements don't currently work in XHTML documents, but are specified to.
* Custom Elements don't currently work in XUL documents and additional work is required to support them. In particular:
  * Elements can be created from the XUL prototype cache, bypassing the XML parser
  * XUL elements are subtly different from HTML elements in how they handle their parenting and prototype chains
* The [Custom Elements polyfill](https://github.com/webcomponents/custom-elements) doesn't work in XUL documents, but a [fork designed to use deepTreeWalker](https://github.com/webcomponents/custom-elements/compare/master...bgrins:firefox-browser-chrome?expand=1) does. Using this would allow us to prototype swapping out components without switching the top level document away from XUL. If this turns out to be a good way to migrate away from XBL, we'd have to do work to make the native Web Components implementation work in XUL documents.
* The relevant Web Components implementations are still off by default and missing features in Firefox:
  * Custom Elements is in progress, with plans to complete the work by end Q3 2017
  * Shadow DOM is backlogged, with plans to start the work in H2 2017.
