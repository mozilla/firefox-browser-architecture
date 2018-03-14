---
title: XUL Overlay Removal Review Packet
layout: text
---

# XUL Overlay Removal Review Packet

## Summary

### Problem Space

XUL overlays provide a way to customize the UI and share code across different XUL documents. In addition to sharing many of the common issues already outlined about XUL, overlays were primarily used by legacy add-ons and are not supported by new web extensions. Within mozilla-central, there are still some uses of overlays, but the majority of them could be simplified and removed or use a more standard “web” like approach.

### End State

All of the C++ code that supports the complex logic of loading and merging overlays would be removed. Firefox developers would no longer need to learn an obscure Mozilla only technology. Startup performance may see a slight improvement since we would be doing the work at compile time instead of runtime.

### Stakeholders

XUL Maintainers (Neil Deakin, Gijs Kruitbosch), Firefox Frontend (Dave Townsend)

## Brief
A detailed review of XUL overlay usage shows some common patterns and gives insight into how they can be removed. In general, the uses fall into four categories: unused, used once, used multiple times for simple templating, and used multiple times in a more complicated manor.

### Unused

These can just be removed.

### Used Once

These can simply be inlined into their master documents.

### Simple Templating

These can be included via the preprocessor.

### Advanced Uses

These are more tricky and will have to be replaced case by case. A few possible approaches:

* Preprocessor include - break up the overlay into a few include files. This has the downside that the master document would need to do a number of includes to get all the various     files (dtd, css, js, xml).
* Custom element - use the new custom elements. The master document would need to includes a JS file that defines the element.
* JS - move element creation to JS
