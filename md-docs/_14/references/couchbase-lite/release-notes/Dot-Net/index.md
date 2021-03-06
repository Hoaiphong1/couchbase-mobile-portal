---
id: dot-net-release-notes
title: .NET release notes
---

__Deprecation notices__

The following features are being deprecated in Couchbase Mobile 1.4 and will be unsupported in 2.0.

- ForestDB
- Bonjour P2P Discovery Service

The following platforms are being deprecated in Couchbase Mobile 1.4 and will be unsupported in 2.0.

- .NET 3.5

## 1.4.1 release

__Performance Improvements__

- [__#827__](https://github.com/couchbase/couchbase-lite-net/issues/827) Purge does not remove row from 'docs' table
- [__#856__](https://github.com/couchbase/couchbase-lite-net/issues/856) View indexing is very slow, due to mis-optimized SQLite query

__Enhancements__

- [__#867__](https://github.com/couchbase/couchbase-lite-net/issues/867) Preserve revision history for each "branch" on pruning
- [__#915__](https://github.com/couchbase/couchbase-lite-net/issues/915) Support DocIDs setting for pull replication with Sync Gateway

__Bugs__

- [__#778__](https://github.com/couchbase/couchbase-lite-net/issues/778) Push sync becomes slower as local DB grows, fails to advance lastSequence
- [__#821__](https://github.com/couchbase/couchbase-lite-net/issues/821) .NET not encoding atts_since
- [__#824__](https://github.com/couchbase/couchbase-lite-net/issues/824) Pull replication stopping when coming online
- [__#830__](https://github.com/couchbase/couchbase-lite-net/issues/830) Remove RunInOuterTransaction
- [__#832__](https://github.com/couchbase/couchbase-lite-net/issues/832) New_edits on _bulk_docs should default to true
- [__#834__](https://github.com/couchbase/couchbase-lite-net/issues/834) 1.4 breaks custom headers on PUT HTTP requests
- [__#835__](https://github.com/couchbase/couchbase-lite-net/issues/835) Unable to select multiple values with LINQ 1.4
- [__#840__](https://github.com/couchbase/couchbase-lite-net/issues/840) Couchbase.Lite.dll missing Product Version Info and Others
- [__#847__](https://github.com/couchbase/couchbase-lite-net/issues/847) Trying to remove encryption from a SQLite DB throws an exception
- [__#848__](https://github.com/couchbase/couchbase-lite-net/issues/848) Retry loop causes web socket changes feed to stop
- [__#850__](https://github.com/couchbase/couchbase-lite-net/issues/850) Wrong return code for listener causes problems with puller and deleted items.
- [__#853__](https://github.com/couchbase/couchbase-lite-net/issues/853) Updating a design doc view map function does not take effect
- [__#863__](https://github.com/couchbase/couchbase-lite-net/issues/863) PollInterval is not obeyed
- [__#910__](https://github.com/couchbase/couchbase-lite-net/issues/910) Channel removal potentially disrupts attachments
- [__#914__](https://github.com/couchbase/couchbase-lite-net/issues/914) Bulk download error not passed to client

## 1.4 release

__API change__

- An `isDeletion` property is now available on the `DatabaseChange` object to identify if a change is tombstone document.

__Enhancements__

- [__#245__](https://github.com/couchbase/couchbase-lite-net/issues/245) Implement LINQ IQueryable
- [__#575__](https://github.com/couchbase/couchbase-lite-net/issues/575) Package Custom SQLite
- [__#709__](https://github.com/couchbase/couchbase-lite-net/issues/709) Implement LiteServ on mobile platforms
- [__#741__](https://github.com/couchbase/couchbase-lite-net/issues/741) Remove Mono.Security build
- [__#745__](https://github.com/couchbase/couchbase-lite-net/issues/745) Support 'stale=false' queries in Listener
- [__#766__](https://github.com/couchbase/couchbase-lite-net/issues/766) Push replication does not work when attachment content type is missing
- [__#797__](https://github.com/couchbase/couchbase-lite-net/issues/797) Add DatabaseChangedEventArgs.IsDeletion to public API
- [__#802__](https://github.com/couchbase/couchbase-lite-net/issues/802) Attempt to silence websockets when they shutdown

__Bugs__

- [__#672__](https://github.com/couchbase/couchbase-lite-net/issues/672) .NET LiteServ is loading windows specific binaries on mac
- [__#710__](https://github.com/couchbase/couchbase-lite-net/issues/710) SQL Exception caused by trying to query non-existent `maps_#` table
- [__#713__](https://github.com/couchbase/couchbase-lite-net/issues/713) QueryOptions skip and limit used incorrectly during reduce
- [__#718__](https://github.com/couchbase/couchbase-lite-net/issues/718) V1.3 Continuous replication stops on change tracker error
- [__#719__](https://github.com/couchbase/couchbase-lite-net/issues/719) Replication - Inconsistent Number of Documents After Replicating
- [__#724__](https://github.com/couchbase/couchbase-lite-net/issues/724) V1.3 GetPendingDocumentIDs value is cached while offline for Push
- [__#725__](https://github.com/couchbase/couchbase-lite-net/issues/725) Replication fails to authenticate with POST _replicate cookie. 
- [__#727__](https://github.com/couchbase/couchbase-lite-net/issues/727) BlobStoreWriter fails to delete encryption marker when removing encryption
- [__#731__](https://github.com/couchbase/couchbase-lite-net/issues/731) Missing docs when pushing to multiple sync gateways
- [__#732__](https://github.com/couchbase/couchbase-lite-net/issues/732) Transaction rollback does not invalidate cached doc ID map
- [__#735__](https://github.com/couchbase/couchbase-lite-net/issues/735) SIGSEGV when closing a database while replication is still active
- [__#742__](https://github.com/couchbase/couchbase-lite-net/issues/742) CB Lite on Android Xamarin fails to replicate when local DB is large
- [__#749__](https://github.com/couchbase/couchbase-lite-net/issues/749) Attachments failed to replicate when deleted and recreated
- [__#753__](https://github.com/couchbase/couchbase-lite-net/issues/753) SQLiteStorageEngine can return stale data in rare cases
- [__#755__](https://github.com/couchbase/couchbase-lite-net/issues/755) Pull replication gets stuck in Active with more completed changes than pending changes
- [__#756__](https://github.com/couchbase/couchbase-lite-net/issues/756) Pusher does not report "Forbidden" errors
- [__#757__](https://github.com/couchbase/couchbase-lite-net/issues/757) Replicator should not assume LastSequence is an integer
- [__#765__](https://github.com/couchbase/couchbase-lite-net/issues/765) Not all docs replicating with p2p .NET pull replication
- [__#767__](https://github.com/couchbase/couchbase-lite-net/issues/767) Changes feed skipping changes on rapid long poll
- [__#772__](https://github.com/couchbase/couchbase-lite-net/issues/772) ManagementException in GetWindowsName() of Platform.cs
- [__#773__](https://github.com/couchbase/couchbase-lite-net/issues/773) BrowseService's Dispose crash fix not merged to Master?
- [__#776__](https://github.com/couchbase/couchbase-lite-net/issues/776) Replication.Stop() doesn't release sockets
- [__#779__](https://github.com/couchbase/couchbase-lite-net/issues/779) Attachments disappearing from docs when continuous pull is enabled
- [__#782__](https://github.com/couchbase/couchbase-lite-net/issues/782) Encrypted attachments fail to push
- [__#785__](https://github.com/couchbase/couchbase-lite-net/issues/785) NullReferenceException in Replication.GetStatusFromError
- [__#789__](https://github.com/couchbase/couchbase-lite-net/issues/789) Investigate root cause of deletion not propagating to .NET client
- [__#795__](https://github.com/couchbase/couchbase-lite-net/issues/795) Set ServicePointManager.SecurityProtocol
- [__#796__](https://github.com/couchbase/couchbase-lite-net/issues/796) Headers not sent when pullReplication.Continuous = true
- [__#803__](https://github.com/couchbase/couchbase-lite-net/issues/803) NullReferenceException in WebSocketChangeTracker
- [__#804__](https://github.com/couchbase/couchbase-lite-net/issues/804) Attachment doesn't handle non-seekable streams gracefully
- [__#806__](https://github.com/couchbase/couchbase-lite-net/issues/806) 1.4 SQLite returns incorrect reason on REST API Get to purged doc
- [__#807__](https://github.com/couchbase/couchbase-lite-net/issues/807) Handle closed web sockets by reconnecting
- [__#825__](https://github.com/couchbase/couchbase-lite-net/issues/825) Pull replication doesn't restart with web sockets
- [__#826__](https://github.com/couchbase/couchbase-lite-net/issues/826) .NET not sending all docs to a client pull replication (p2p, SSL)

## Where to get it

You can download this release from the [downloads page](http://www.couchbase.com/nosql-databases/downloads#couchbase-mobile).