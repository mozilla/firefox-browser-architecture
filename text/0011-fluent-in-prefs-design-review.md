# Fluent in Prefs Design Review

## Introduction

[Fluent](http://projectfluent.org/) is a "family of localization libraries designed to improve how software is translated." Mozilla's localization engineering team (the "engineering team") is migrating Firefox's localization architecture from DTD/properties files to FTL files processed by Fluent, which will enable richer and more accurate translations, runtime locale loading/switching, runtime LTR/RTL and pseudo-locale switching, and a fallback locale. The first target for this migration is the preferences interface (the about:preferences page and its various subdialogs).

The engineering team met with a set of architects and domain experts (the "review team") to discuss risks and mitigations, which constituted a lightweight "design review" of the project's architecture. This document describes the concerns that the review team raised and the risk mitigations that the teams discussed.

## Why Preferences

The team chose preferences because they're text-heavy, include users' names (which exercises Fluent's support for grammatical gender), and because the precision of text matters in preferences to ensure users clearly understand the implications of changes. So preferences are a good way to prove the change to Fluent before adopting it more widely.

Fluent is a big change to the localization process, and the team plans to validate the change by translating just four strings initially (the "name" strings in brand.dtd/properties?) and then verifying those changes across the localization toolchain.

## Chrome Privileges

The review team asked whether there are parts of preferences that don't have chrome privileges, and if that influences the use of Fluent. It seems like the only such part is the Firefox Accounts image (???), which loads over a network and is localized independently of Firefox, so it won't be an issue.

## Search Incompatibility

The search feature of about:preferences is an issue, as the way that it searches preference labels and descriptions is not compatible with the way that Fluent injects text into the page.  The teams discussed options and concluded that there is a straightforward way to update the search implementation to be compatible with Fluent.

Specifically, FTL messages support arbitrary attributes, so an FTL message for a searchable preference control could include a "searchterms" attribute containing the text to search, i.e.

```
key1
  .label = "click"
  .searchterms = " … "
```

Attribute values can include references to other strings within the FTL file, so localizers could avoid string duplication if desired.  This model would also enable localizers to customize search terms to include extra keywords that aren't included in other strings related to the control.  Finally, this model supports English fallback for searches via the creation of a secondary *document.l10n* object with the `en` locale.

The review team raised some concerns about the performance of runtime searches and suggested that the engineering team consider generating an index from search terms to the controls they match, either at build time or on first search.  Some participants suggested being careful to avoid prematurely optimizing, however. In the end, the teams didn't reach consensus on the optimal solution, and further research/experimentation is warranted here.

## Schedule Risk

After validating the process with the first four strings, the engineering team plans to migrate the rest of the strings, but they're aware of the large set of pending changes for XBL replacement in [bug 1379338](https://bugzilla.mozilla.org/show_bug.cgi?id=1379338), and they'd prefer to await those before proceeding to minimize merge hell.

The XBL replacement patch author ([myk](https://mozillians.org/en-US/u/myk/)) and reviewer ([jaws](https://mozillians.org/en-US/u/jaws/)) were both members of the review team, and they noted that those changes should land soon and thus shouldn't create schedule risk.

Another member of the review team noted that another team is rewriting the "delete cookies" dialogs, and the engineering team should coordinate with that team.

## Migration

The teams discussed tools for migrating from DTD/properties files to FTL files. They also discussed the possibility of migrating strings from one FTL file to another, as the difficulty of migrating strings between DTD/properties files is a common frustration with the current system.

The engineering team has developed tools to migrate strings from DTD/properties to FTL files, but they don't yet have a way to migrate them from one FTL file to another. That doesn't regress functionality, but the review team still recommended that the engineering team consider tackling that problem in the future.

The teams also discussed merging the various preferences DTD/properties files into a single FTL file, since their current ontology is out-of-date (because pref categories have changed over time, and the files haven't changed with them), and since there isn't a reason to load only a subset of strings (especially given the new search feature, which needs to search across all of them).

There was also some discussion about the conflict between engineers, who bias toward coalescing "identical" English strings used in different contexts (such as "open") because DRY; and localizers, who separate such strings because they aren't actually always identical in all locales.

Coalescing is considered harmful, and a concern was raised that merging preferences strings into a single FTL file might encourage engineers to do it even more. The teams concluded that this concern can be addressed at the policy/review level.

## String Changes

The review team probed about Fluent support for string changes, and the engineering team explained that support is as good as the current system, which has two ways to modify a string, and work is planned to make it even better by adding a third way.

Specifically, under both the current system and the current version of Fluent, a developer can make a minor change to a string (f.e. a typo fix) that shouldn't affect its translations by preserving the string's key, and they can make a major change to a string that should affect its translations by changing the string's key.

In the future, Fluent will support versioning of a string key via a tag, such that developers can also make changes that affect localizations without having to change the string's key. (There was some discussion about which kinds of changes would fit into this bucket.)
