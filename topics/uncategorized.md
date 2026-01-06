# Uncategorized

This document contains different optimization techniques that are too small to dedicate the whole file for each of them,
but still they're very useful.

<!-- @formatter:off -->
<!-- TOC -->
* [Uncategorized](#uncategorized)
  * [Table fillfactor](#table-fillfactor)
<!-- TOC -->

## Table fillfactor

There's an optimization related to [index fillfactor](indexes.md#index-fillfactor).

Official documentation has a great description of this mechanism 
[here](https://www.postgresql.org/docs/current/sql-createtable.html?utm_source=chatgpt.com#RELOPTION-FILLFACTOR).

In short: When set below **100**, `INSERT` will pack each page only up to that percentage, 
leaving the remaining space reserved primarily for future row versions created by `UPDATE`.

Possible values of `fillfactor` are in a range `[10; 100]` with default 100.
But at the same time there's mentioned "in heavily updated tables smaller `fillfactors` are appropriate";

Usually almost all project tables are **often updated**, therefore it's suggested to decrease `fillfactor`.

Pros of low `fillfactor`:

* Increased chance of a [HOT update](https://www.postgresql.org/docs/current/storage-hot.html?utm_source=chatgpt.com).
* Less update overhead. New updated version of a tuple can fit into current page, even if it's not a HOT update.

Cons of low `fillfactor`:

* More storage consumed.
* Potentially worse read performance in rare cases. More heap pages may need to be visited for the same logical result set.
* Benefits are workload-specific. If updates change indexed columns (preventing HOT) or if rows rarely grow, 
  the gain may be limited, while the space cost remains.
* Costly change of `fillfactor`. Even after `ALTER TABLE` still need to use
  [full rewrite operation](https://www.bytebase.com/blog/postgres-table-rewrite/) (like `VACUUM FULL`).

Use cases:

* Update-heavy OLTP tables with stable indexed keys where update operation don't change a value of indexed column (except BRIN).
  Increases HOT updates chance.
* Frequently updated fields like timestamp, audit fields, user activity markers.
* Queue / workflow table records that move through different statuses (opened, validated, closed) and get a lot of updates.

The system view [pg_stat_all_tables](https://www.postgresql.org/docs/current/monitoring-stats.html#MONITORING-PG-STAT-ALL-TABLES-VIEW) allows monitoring of the occurrence of HOT and non-HOT updates.

TODO: sql snippet
