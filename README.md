# better-pg

Cookbook for PG optimization.

Sometimes a tuning is about a pure optimization, but sometimes about compromises.

This repository is planned more like _personal_ hints of hidden Postgres corners,
but it's open to critic and improvements.

## Useful resources

| Name                                                                                                                     | Description                                                                                                                                            |
|--------------------------------------------------------------------------------------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------|
| [Tuning guide](https://postgresqlco.nf/tuning-guide)                                                                     | An interactive tuning of postgresql.conf with interactions and simple explanations.                                                                    |
| [Postgres internals](https://www.interdb.jp/pg/index.html#the-internals-of-postgresql)                                   | Explanation of Postgres internals from an enthusiast [Hironobu Suzuki](https://github.com/s-hironobu).                                                 |
| [Udemy PostgreSQL High Performance Tuning Guide](https://www.udemy.com/course/postgresql-high-performance-tuning-guide/) | May be hard to learn, but very useful course related to Postgres turing using all the DB aspects. [Author](https://www.udemy.com/user/lucian-oprea-2/) |

## Topics

* [Indexes](/topics/indexes.md)
* SARGable queries
* Write ahead log (WAL).
* Vacuum
* Statistics (pg_stat)
* Analyzation (vacuum analyze, analyze, explain, explain analyze)
* Scalability
* Data distribution
* Modern observability

## Architecture overview

Postgres architecture can be split into 2 parts: processes and memory.

Process architecture:
![process architecture](assets/pg-process-architecture.png)

Details here - https://www.interdb.jp/pg/pgsql02/01.html

Memory architecture:
![memory architecture](assets/pg-memory-architecture.png)

Details here - https://www.interdb.jp/pg/pgsql02/02.html

Each component can be tuned accordingly to system requirements.

## Cloud

In AWS RDS `postgresql.conf` values can be set up in `RDS` > `Parameter groups`.
[Docs](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_WorkingWithDBInstanceParamGroups.html)

## Tags in this repo

Tags in this repository show resources that can be affected by some topic.
Doesn't matter whether a tuning makes that aspect better or worse.
For example index can increase speed of search,
but consume a storage, therefore it's marked with speed and storage hashtags.

A reader can use tags to determine what aspect of a DB he wants to optimize and continue with an actual topic(s).
