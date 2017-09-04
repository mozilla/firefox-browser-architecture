---
title: Browser Architecture Newsletter 2
layout: text
---

# Browser Architecture Newsletter 2

[Link to mailing-list archive](https://groups.google.com/d/msg/firefox-dev/Rc2w2a9e8HQ/vuJZffViCwAJ)

Welcome to the second Browser Architecture Newsletter! Since our [last update](https://groups.google.com/forum/#!msg/firefox-dev/ueRILL2ppac/RxR9lLPkAwAJ;context-place=forum/firefox-dev), we have a [new website](https://mozilla.github.io/firefox-browser-architecture/) which includes posts about some of the [problems with XUL](https://mozilla.github.io/firefox-browser-architecture/text/0003-problems-with-xul.html), [extracting Necko](https://mozilla.github.io/firefox-browser-architecture/text/0002-extracting-necko.html) as a standalone component, and [comparing](https://mozilla.github.io/firefox-browser-architecture/text/0004-xbl-web-components.html) XBL with Web Components.

The website is built from [this GitHub repository](https://github.com/mozilla/firefox-browser-architecture) of Markdown files, and we’re using pull requests to review and revise our research proposals and analyses. Do you have ideas for improvements? File an issue on the repo or contact us in #browser-arch!

## XBL Conversion

We'd like to reduce or even remove XBL and XUL from the tree. It's becoming clear that replacing XBL is more important and easier than replacing XUL, so there are a few projects ongoing to learn more about it.

A conversion of the behavioral <preference> binding is in progress at [Bug 1379338](https://bugzilla.mozilla.org/show_bug.cgi?id=1379338). Although this work started as a prototype, it resulted in a more straightforward implementation than we currently have, so is on the path to being landed.

As mentioned above, there’s some new [documentation](https://mozilla.github.io/firefox-browser-architecture/text/0004-xbl-web-components.html) comparing XBL with Web Components. There’s also a [visual tree](https://bgrins.github.io/xbl-analysis) of the bindings we use in Firefox and what features each uses.

A prototype of replacing some XBL UI components in about:preferences with Custom Elements is attached to [Bug 1392367](https://bugzilla.mozilla.org/show_bug.cgi?id=1392367). This relies on a [forked version](https://github.com/webcomponents/custom-elements/compare/master...bgrins:firefox-browser-chrome?expand=1) of the [custom elements polyfill](https://github.com/webcomponents/custom-elements) made to work in XUL documents.

A prototype of converting <tabbrowser> to a JS class is in progress at [Bug 1392352](https://bugzilla.mozilla.org/show_bug.cgi?id=1392352). This would cause gBrowser to become a JS object instead of a DOM Node. This conversion is semi-automated [by a script converting the binding to a JS class](https://github.com/bgrins/xbl-analysis/blob/ad87d682b937620b1c129e49c4081483c7074540/scripts/build-custom-elements.js).

## Storage and Sync

We're in the process of cataloging and analyzing some of Firefox's sprawling storage technologies (45+ different data stores and dozens of formats on desktop alone), and how they relate to Sync and across platforms. We're hoping to find ways to make this situation simpler, more reliable, and easier to extend.

You can see the beginnings of our knowledge capture [here](https://github.com/mozilla/firefox-data-store-docs) (PRs welcome!), and a [spreadsheet](http://bit.ly/2vpZI9e) showing how data types interact with Sync and mobile platforms.

Huge thanks to the Sync team and data and product folks across the org for helping with our research.

## Workflow Improvements

Lately I've been paying extra attention to development papercuts when working on bugs, especially issues when using ./mach run without passing a profile. Here’s a list of recent changes on that front:

*   You can [open the Browser Toolbox](https://groups.google.com/d/msg/firefox-dev/678mrnS6120/KXcP18ZUCAAJ) in a local build without flipping prefs.
*   The default browser prompt and about:config warning screen [no longer show up](https://groups.google.com/d/msg/firefox-dev/kPwA1y-7BpI/gX6rvhEjBQAJ) for the scratch_user (the profile used when you do ./mach run without a profile).
*   You can flip any pref for the scratch_user with the --setpref option, or by adding them in your [machrc file](https://developer.mozilla.org/en-US/docs/Mozilla/Developer_guide/mach#Does_mach_have_its_own_configuration_file). For example, to make sure I always see dump output I added this to my ~/.mozbuild/machrc file:  
    `[runprefs]  
    browser.dom.window.dump.enabled=true`
*   There’s a [new keyboard shortcut](https://groups.google.com/d/msg/firefox-dev/Tme95bp3EHY/ow-l077FAAAJ) to do a restart + session restore in local builds.
*   The Browser Console is [now part of session store](https://bugzilla.mozilla.org/show_bug.cgi?id=1388552), so it will reopen after using the above keyboard shortcut.
