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
create table {table name for backup} as (select * from {table name to be backuped})
```
