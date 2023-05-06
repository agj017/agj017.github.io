---
layout: post
title: test-postgresql-transaction
date: 2023-05-06
category: test for postgresql
---

## prerequisite

* login DB

login for session A
```shell
psql -d play-ground
```

login for session B
```shell
psql -d play-ground
```

* DDL
```sql
create table account
(
    id      bigint not null
        primary key,
    history jsonb,
    name    varchar(255)
);
```

* DML
```sql
insert into account values (1, null, 'john');
```

## buffer test

* purpose

각 transaction은 buffer memory(disk로부터 가져온 data 저장소)가 아닌 각 자의 memory에서 작업하는 것을 확인한다.

* prerequisite

```sql
play-ground=# select * from account;
 id | history | name
----+---------+------
  1 |         | john
(1 row)
```


* session A
```sql
play-ground=# start transaction;
START TRANSACTION
play-ground=# select * from account where id = 1;
 id | history | name
----+---------+------
  1 |         | john
(1 row)

play-ground=# update account set name = 'tom' where id = 1;
UPDATE 1
play-ground=# select * from account where id = 1;
 id | history | name
----+---------+------
  1 |         | tom
(1 row)
```

* session B
```sql
play-ground=# start transaction;
START TRANSACTION
play-ground=# select * from account where id = 1;
 id | history | name
----+---------+------
  1 |         | john
(1 row)
```

* session A
```sql 
play-ground=# commit;
COMMIT
```

* session B
```sql
play-ground=# select * from account where id = 1;
 id | history | name
----+---------+------
  1 |         | tom
(1 row)
```

* conclusion

session A는 자신의 memory에서 작업을 하였으므로 commit하기 전까지 session B는 id 1에 대한 name이 john으로 보이는 걸 확인 할 수 있다.

## locking test

* purpose

두 transaction이 하나의 row에 접근하여 write할 경우에 먼저 접근한 transaction이 해당 row에 lock을 거는 것을 확인한다.

* prerequisite

```sql
play-ground=# select * from account;
 id | history | name
----+---------+------
  1 |         | john
(1 row)
```

* session A

```sql
play-ground=# start transaction;
START TRANSACTION
play-ground=# update account set name = 'tom' where id = 1;
UPDATE 1
play-ground=# select * from account where id = 1;
 id | history | name
----+---------+------
  1 |         | tom
(1 row)
```

* session B

```sql
play-ground=# start transaction;
START TRANSACTION
play-ground=# select * from account where id = 1;
 id | history | name
----+---------+------
  1 |         | john
(1 row)

play-ground=# update account set name = 'lucas' where id = 1;
...(locking)
```

* session A

```sql
play-ground=# commit;
COMMIT
```

* session B
```sql
...(unlock)
UPDATE 1
```

* conclusion

session A가 id 1에 대해서 transaction 안에서 write를 할 때 session B가 해당 row에 대해서 write를 시도하면 session B는 locking 되는 것을 확인 할 수 있고 session A가 commit 되는 순간 session B의 write 작업이 자동으로 실행되는 것을 확인 할 수 있다.