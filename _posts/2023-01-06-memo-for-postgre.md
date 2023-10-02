---
layout: post
title: memo for postgre
date: 2023-01-06
category: memo
---

## path (from homebrew)

* log file: /opt/homebrew/var/log/postgresql@14.log

* lock file: /opt/homebrew/var/postgresql@14/postmaster.pid

* config file: /opt/homebrew/var/postgresql@14/postgresql.conf

## sql statement

* back up table 

```sql
CREATE TABLE {table name for backup} AS (SELECT * FROM {table name to be backuped})
```
## postgresql meta data

* path where data is stored
```sql
SHOW data_directory;
```

* meta data for each database in postgresql

the oid of the table is mapped with directory name.

```sql
SELECT * FROM pg_database;
```

* meta data of index, sequence, primary key

```sql
SELECT * FROM pg_class;
```
