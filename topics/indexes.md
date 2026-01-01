# Indexes

<!-- TOC -->
* [Indexes](#indexes)
  * [Tuning](#tuning)
  * [Concepts](#concepts)
    * [Index is](#index-is)
    * [Partial indexes (WHERE)](#partial-indexes-where)
    * [Covering indexes (INCLUDE)](#covering-indexes-include)
  * [Index types](#index-types)
    * [B-tree (default, most common).](#b-tree-default-most-common)
      * [Best for](#best-for)
      * [Advantages](#advantages)
      * [Disadvantages](#disadvantages)
      * [Typical fields](#typical-fields)
      * [Notes](#notes)
    * [HASH](#hash)
      * [Best for](#best-for-1)
      * [Advantages](#advantages-1)
      * [Disadvantages](#disadvantages-1)
      * [Typical fields](#typical-fields-1)
    * [GIN (Generalized Inverted Index)](#gin-generalized-inverted-index)
      * [Best for](#best-for-2)
      * [Advantages](#advantages-2)
      * [Disadvantages](#disadvantages-2)
      * [Typical fields](#typical-fields-2)
      * [Notes](#notes-1)
    * [BRIN (Block Range Index)](#brin-block-range-index)
      * [Best for](#best-for-3)
      * [Advantages](#advantages-3)
      * [Disadvantages](#disadvantages-3)
      * [Typical fields](#typical-fields-3)
    * [GiST (Generalized Search Tree)](#gist-generalized-search-tree)
    * [SP-GiST (Space-Partitioned GiST)](#sp-gist-space-partitioned-gist)
    * [Bloom index (bloom extension)](#bloom-index-bloom-extension)
      * [Best for](#best-for-4)
      * [Advantages](#advantages-4)
      * [Disadvantages](#disadvantages-4)
      * [Typical fields](#typical-fields-4)
      * [Notes](#notes-2)
<!-- TOC -->

## Tuning

## Concepts

### Index is

A PostgreSQL index is a separate **on-disk** data structure that provides a faster path to locate rows than scanning the
whole table.
Conceptually, an index stores:

* Search keys (derived from one or more **columns** or **expressions**), plus
* A pointer to the table row or `TID`.
  TID consists of block number + offset within the block. For example, `(42,9)` means that the item is the ninth
  element in the
  43rd 8KB block of the table or index (blocks are counted from 0).

[The planner](https://www.interdb.jp/pg/pgsql03/01.html#314-planner-and-executor) may choose an _Index Scan_,
_Index-only Scan_ or _Bitmap Index Scan_ to find candidate row locations, then fetch the table rows.

**Indexes accelerate reads but add write costs!**

* _INSERT_: add index entries.
* _UPDATE_: may add new entries (and sometimes remove old ones logically).
* _DELETE_: leaves “dead” entries until cleanup.
  This is why autovacuum/vacuum and reindexing considerations matter operationally.

### Partial indexes (WHERE)

* Index only some part of data based on filter.
* Reduces storage consumed by an index.

Documentation [is here](https://www.postgresql.org/docs/current/indexes-partial.html).

```sql
CREATE INDEX ON orders (created_at)
  WHERE status = 'open';
```

### Covering indexes (INCLUDE)

* This reduces heap access and forces _Index-only scan_.
* Increases storage consumed.

Documentation [is here](https://www.postgresql.org/docs/current/indexes-index-only-scans.html).

This index:

```sql
CREATE INDEX ON users (email)
  INCLUDE (phone);
```

can help with a query like `SELECT email, phone FROM users WHERE email = 'email@gmail.com'`.

## Index types

### B-tree (default, most common).

Name stands for: multi-way balanced tree.

#### Best for

General-purpose indexing: equality, range queries, and sorting (=, <, <=, >, >=, BETWEEN, ORDER BY, prefix LIKE 'abc%').

#### Advantages

* Default and most well-optimized index type in PostgreSQL
* Supports ordering and range operations (can satisfy ORDER BY and range filters).
* Supports multi-column indexes with _left-to-right matching rules_ (explained below).
* This is default index for PRIMARY KEY and UNIQUE.

#### Disadvantages

* Can become large for wide keys or many columns, increasing memory and I/O.
* Not ideal for very low-selectivity columns alone (e.g., boolean flags).
* Does not natively handle “contains” semantics for arrays/jsonb/text search (
  see [GIN index](#gin-generalized-inverted-index)).

#### Typical fields

Integers, timestamps, UUIDs, numeric, short text.

#### Notes

**_Left-to-right matching rules:_**

If we have the table:\
`CREATE TABLE my_table(col1 INT, col2 INT, col3 INT)`

and we have the next index:\
`CREATE INDEX my_table_idx on my_table(col1, col2, col3);`

Then next queries can use index:

```sql
SELECT FROM my_table WHERE col1 < 5;
SELECT FROM my_table WHERE col1 < 5 AND col2 > 10;
SELECT FROM my_table WHERE col1 < 5 AND col2 > 10 AND col3 <= 25;
```

But these queries can **not**:

```sql
SELECT FROM my_table WHERE col2 < 10;
SELECT FROM my_table WHERE col3 <= 25;
SELECT FROM my_table WHERE col2 < 10 AND col3 <= 25;
```

### HASH

Implementation of the [hash table](https://en.wikipedia.org/wiki/Hash_table) data structure.

#### Best for

Equality (`=`) only.

#### Advantages

* Optimized for fast `=` operator lookup.
* Implementation is [WAL-logged](https://www.postgresql.org/about/featurematrix/detail/wal-support-for-hash-indexes/) in
  PostgreSQL 11+, so it is crash-safe.
* Historically less preferred.
* Index size can be slightly more compact than a comparable **B-tree** in some workloads.
* Useful when **B-tree** has a lot of collisions (after profiling).

#### Disadvantages

* Only supports `=`.
* No `ORDER BY` support.
* **B-tree** usually performs **similarly** for equality while also supporting more operators.
* Historically less mature and less used than **B-tree**; fewer real-world tuning examples.
* No multi-column support.

#### Typical fields

Anything with direct comparison (IDs, lookups, etc.).

### GIN (Generalized Inverted Index)

#### Best for

“Contains” queries on multi-valued fields.

#### Advantages

* Good for `jsonb` (`@>`, `<@`, `?`, `?|`,
  `?&`). [Details here](https://www.postgresql.org/docs/9.4/functions-json.html).
  It indexes individual keys/values inside JSON, not just the whole document
* Good for `arrays` (`@>`, `<@`, `&&`). [Operators are here](https://www.postgresql.org/docs/8.3/functions-array.html).
* Supports **full-text search** (`tsvector @@ tsquery`).
  [Text search types](https://www.postgresql.org/docs/current/datatype-textsearch.html).
  [Text search functions and operators](https://www.postgresql.org/docs/current/functions-textsearch.html).

#### Disadvantages

* GIN often creates very large indexes, because it stores multiple entries per row (one per term/element).
  This can impact disk, memory, and backup size.
* Speaking of index size, it also affects write operations.
  It can be noticeably slower than **B-tree**.
* Not for ranges or sorting. This index is for containment, not for ordering.
*

#### Typical fields

`jsonb`, `text[]`, `int[]`, `tsvector`.

#### Notes

If it's possible to move data into normal columns, then **B-tree** index can be simpler and cheaper.

### BRIN (Block Range Index)

#### Best for

Very large tables where values are correlated with physical order.
Useful for logs, metrics, and archival data.

#### Advantages

* Great for **time-series** and **append-only** data.
* Small size. BRIN stores summary information per block range, not per row.
* Fast to _build_ and _reindex_.
* Useful for partitioning.
  On partitioned tables with nicely ordered data, BRIN can be an effective “first filter” before scanning partitions.

#### Disadvantages

* Not precise, gives only approximate estimation for number of blocks (pages).
* Useless on unordered data.
* Not helpful on small tables.
* Doesn't help with sorting.

#### Typical fields

* Append-only (like `created_at`).
* Sequentially incrementing IDs.
* Columns with strong correlation to insertion order.

### GiST (Generalized Search Tree)

GiST is a flexible framework used by extensions (like PostGIS) to index geometric objects and support operations like
“intersects” or “overlaps”.

### SP-GiST (Space-Partitioned GiST)

Data that naturally fits into **tree-like structures**, like network types with prefix-based queries.

### Bloom index (bloom extension)

#### Best for

**Large tables** with many columns where queries use many equality conditions (`=`) on different columns and it's not
possible to predict combination of fields used for selection.

#### Advantages

* Handles many columns in one index. It's okay for use 10+ columns in a single Bloom index.
* Compact per-row storage.
  Each row is represented by a fixed-size bitset (a signature).
  Index size grows mostly with row count, not with total column width.
  Larger signature works better (less false positives), but consumes more storage and increases time to scan.
* Flexible for many combinations of predicates.
  Index on `(a, b, c, d, e, f, g, h, i, j)` can be used with any combination of columns using `=`.

#### Disadvantages

* **False positives.**
  In worst-case scenario index returned positive, then postgres scanned heap and found nothing.
* Only supports `=`. No `ORDER BY` support. Similarly to [HASH](#hash) index.
* Complex to tune
  [parameters](https://www.postgresql.org/docs/current/bloom.html#BLOOM-PARAMETERS)
  like **bitset length** and **number of hash bits per column**.
  Bad choices can lead to either many false positives or wasted space.
* It's an extension. Not built-in by default.

#### Typical fields

A set of low to medium cardinality fields with unpredictable ordering.

#### Notes

Answers to question "Can this row **possibly** match **all** these filters?"

Documentation [is here](https://www.postgresql.org/docs/current/bloom.html).

