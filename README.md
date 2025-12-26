# better-pg

Cookbook for PG tuning.

Sometimes a tuning is about a pure optimization, but sometimes about compromises.

## List of useful resources

| Name                                                                                                                     | Description                                                                                            |
|--------------------------------------------------------------------------------------------------------------------------|--------------------------------------------------------------------------------------------------------|
| [Tuning guide](https://postgresqlco.nf/tuning-guide)                                                                     | An interactive tuning of postgresql.conf with interactions and simple explanations.                    |
| [Postgres internals](https://www.interdb.jp/pg/index.html#the-internals-of-postgresql)                                   | Explanation of Postgres internals from an enthusiast [Hironobu Suzuki](https://github.com/s-hironobu). |
| [Udemy PostgreSQL High Performance Tuning Guide](https://www.udemy.com/course/postgresql-high-performance-tuning-guide/) | May be hard to learn, but very useful course related to Postgres turing using all the DB aspects.      |

## Main topics for increasing performance of Postgres instance

1. Indexes (vs Sequential scan vs Bitmaps).
2. SARGable queries.
3. WAL writer and it's Checkpoint and Background writer.
4. [Auto]vacuum.
5. Table data clustering
6. Statistics (pg_stat). Observability
7. Analyzation (vacuum analyze, analyze, explain, explain analyze)

## Indexes

Impacts: #speed #storage

## Cloud

In AWS RDS `postgresql.conf` values can be set up in `RDS` > `Parameter groups`.
[Docs](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_WorkingWithDBInstanceParamGroups.html)

## Tags in this repo

Tags in this repository show resources that can be affected by some topic.
Doesn't matter whether a tuning makes that aspect better or worse.
For example index can increase speed of search,
but consume a storage, therefore it's marked with speed and storage hashtags.

A reader can use tags to determine what aspect of a DB he wants to optimize and continue with an actual topic(s).
