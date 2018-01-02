
# Introduction

There are a large number of data stores being used to hold data across all Firefox instances and platforms. There is currently nowhere that documents all these data stores and how they share information between them. We have 3 platforms, desktop, Android and iOS and each of these platforms shares data with the others. But not all the information is synced completely across all platforms, and the data itself is not stored in the same way everywhere. When starting to think about how we might provide a holistic user data solution for Mozilla, it is important to understand the scope of the problem and the specifics of the data that is being handled. 

We therefore need to look at all of the data stores that we are currently using and document their purpose, structure, the kind of data that they hold and how they share that data.


# Summary

The extensive documentation can be found in the [Firefox Data Stores](https://github.com/mozilla/firefox-data-store-docs) repository.

* There are a total of 46 separate data stores in Firefox for desktop. Within these 46 stores, there are at least 10 different storage formats. Very few of these stores are well documented in the code. 
* Firefox for Android Java frontend uses 3 stores. It consolidates `places.sqlite` and `favicons.sqlite` into a single SQLite database, `browser.db`. It uses 4 different storage formats, `SQLite`, `JSON`, `XML` & `JS`. Android frontend documents its data stores well and uses a single file to define the schema for all its tables.
* Gecko on Android uses the vast majority of the same stores as Gecko on Desktop. 
* Firefox for iOS has 5 data stores. `metadata.db` was created to store metadata for use in Activity Stream and was created separately to make syncing of this data at a later data easier. Due to a change in strategy, this store is no longer in use. iOS uses 3 different storage formats, `SQLite`, `JSON` and `plist`. It also writes credential information to the iOS keychain. iOS stores are not especially well documented, but the schemas are defined in a database specific files which makes discovering the schemas trivial. The exception to this is the reading list store which is defined separately from the others. iOS has very strong encapsulation of storage. The front-end code uses specific, narrow, safe APIs to read and write data, and storage manages sync metadata.
* Desktop stores logins in an encrypted JSON store, `logins.json`. iOS keeps logins as a separate SQLite database, `logins.db`, that uses `sqlcipher` for encryption. Android uses a SQLite database, `signons.sqlite`, for storing logins. `signons.sqlite` used to be used on desktop too, but they were migrated into `logins.json`. Android frontend is currently in the middle of removing the dependency on the Gecko-side store, resulting in a set of login tables inside `browser.db`, but the work is as yet incomplete.
* `prefs.js` is a simple key value file written to disk, but it contains well over 1200 entries. Its design means that changes made are often not written out until shutdown. Therefore, if the shutdown is not clean, any changes made during the session are lost, resulting in potential inconsistency between prefs and state elsewhere in the system.
* Firefox desktop has an entire SQLite database, `storage.sqlite`, that contains no tables. It is used entirely to reference the schema version value and uses that value to create the version number for the Storage directory. The Storage directory contains all of the files and stores used by the app.
* Of the data fields stored by desktop, only 8% are available to Firefox Sync.
* A visual overview of the [fields made available to Firefox Sync](https://docs.google.com/spreadsheets/d/1k9_K7Dc3q2h3SDV0vwjTgJou-ndza6WuobyJ1bbemtc/edit?ts=5977ab9d#gid=1269587388) and how each sync record is implemented on each platform is also available.

