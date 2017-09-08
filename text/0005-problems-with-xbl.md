---
title: Problems with XBL
layout: text
---

# Problems with XBL

## Unmaintained

XBL is a proprietary technology developed by Mozilla. We are solely responsible for the maintenance and improvement of XBL. Any time spent on doing this doesn’t contribute to the evolution of the Web so effectively we don’t spend anytime improving XBL except for plugging serious holes.

## Poorly Documented

The XBL wiki page says:
> "XBL 1.0 is specified in XBL 1.0 Reference. Unfortunately, the actual implementation in Mozilla is different from the specification, and there's no known document available describing the differences. Hopefully, the Reference will be updated to describe those differences."

That was written in 2005. It’s still true. Except for the "hope" part.

Unlike other technologies like web components where there is plenty of documentation available on the internet, documentation for XBL is almost entirely written by Mozilla and so there is very little of it.

The only reference for what is "correct" is what the frontend expects, and what the frontend expects is largely based around a hodgepodge of workarounds for bizarre behavior in the XBL implementation

## Verbose and Brittle (Because XML)

Writing code in XML requires extensive use of CDATA sections making the syntax more verbose than it needs to be.

## Dynamic Binding Changes via CSS

An XBL binding can set an attribute on itself that causes it to change to a different binding, e.g. [the add-on binding](https://dxr.mozilla.org/mozilla-central/rev/3ecda4678c49ca255c38b1697142b9118cdd27e7/toolkit/mozapps/extensions/content/extensions.xml#1245). These cases are difficult to find and debug and it is very unclear what is happening unless the code is well documented.

## Complicates Stylo

We have a ton of code in stylo to handle XBL `<stylesheets>`, and XBL scoped stylesheets are sufficiently different from shadow DOM scoped stylesheets that the code isn't really reusable

## Complicates Gecko

The fact that XBL is driven off of frame construction is awful. It causes so much weird stuff to happen at very inopportune moments and creates an enormous amount of complexity in what is arguably the most complex and touchy part of the platform (the frame constructor).

Anonymous subtrees cause lots of complication in general, and we currently have to deal with three kinds (shadow DOM, NAC, and XBLAC). Dropping this down to 2 would be a huge improvement.

There is lots of complexity and security weirdness that required us to build and maintain the content XBL scope stuff in XPConnect. Eliminating the in-content XBL bindings (video, marquee, pluginProblem, and scrollbar) would be a lower-hanging fruit to eliminate this.
nsBindingManager is a mutation observer, and its existence means some additional overhead for our DOM mutations, especially [ContentAppended, ContentInserted and ContentRemoved](https://searchfox.org/mozilla-central/rev/51b3d67a5ec1758bd2fe7d7b6e75ad6b6b5da223/dom/xbl/nsBindingManager.h#43).

The way we do XBL binding (via CSS) means that often when you touch a node from script we have to go resolve styles for it to see whether there's XBL involved.  We optimize this out on the web in various ways, but in XUL it's a pain.

## Not Used Elsewhere

This presents a recruiting challenge, there aren’t developers with knowledge of XBL available for hire so everyone we hire has to be trained in this unique technology and there is a steep learning curve.
