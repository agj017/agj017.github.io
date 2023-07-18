---
layout: post
title: test-buffer-locking-postgresql-transaction
date: 2023-05-06
category: test-for-postgresql
---

# prerequisite

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
insert into account values (2, null, 'jacob');
```

# buffer test

## purpose

각 transaction은 buffer memory(disk로부터 가져온 data 저장소)가 아닌 각 자의 memory에서 작업하는 것을 확인한다.

## prerequisite

```sql
play-ground=# select * from account where id = 1;
 id | history | name
----+---------+------
  1 |         | john
(1 row)
```

## test

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

## conclusion

session A는 자신의 memory에서 작업을 하였으므로 commit하기 전까지 session B는 id 1에 대한 name이 john으로 보이는 걸 확인 할 수 있다.

# locking test (isolation level: READ COMMITTED)

## purpose

두 transaction이 하나의 row에 접근하여 write할 경우에 먼저 접근한 transaction이 해당 row에 lock을 거는 것을 확인한다.

## prerequisite

```sql
play-ground=# select * from account where id = 1;
 id | history | name
----+---------+------
  1 |         | john
(1 row)
```

## test

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

## conclusion

session A가 id 1에 대해서 transaction 안에서 write를 할 때 session B가 해당 row에 대해서 write를 시도하면 session B는 locking 되는 것을 확인 할 수 있고 session A가 commit 되는 순간 session B의 write 작업이 자동으로 실행되는 것을 확인 할 수 있다.

# locking test (isolation level: REAPEATABLE READ)

## purpose

isolation level이 REAPEATABLE READ인 두 transaction이 하나의 row에 접근하여 write 할 경우에 먼저 접근한 transaction이 commit하면 이후 접근한 transaction은 err를 발생시키는 것을 확인한다. 

## prerequisite

```sql
play-ground=# select * from account where id = 1;
 id | history | name
----+---------+------
  1 |         | john
(1 row)
```

## test

* session A
```sql
play-ground=# start transaction isolation level repeatable read;
START TRANSACTION
play-ground=# update account set name = 'John' where id = 1;
UPDATE 1
play-ground=#
```

* session B
```sql
play-ground=# start transaction isolation level repeatable read;
START TRANSACTION
play-ground=# update account set name = 'Top' where id = 1;
(waiting)
```

* session A
```sql
play-ground=# commit;
COMMIT
```

* session B
```sql
(waiting)
ERROR:  could not serialize access due to concurrent update
```

## conclusion

isolation level이 REAPEATABLE READ인 두 transaction이 하나의 row에 접근하여 write 할 경우에 먼저 접근한 transaction이 commit하면 이후 접근한 transaction의 statement는 err를 발생시키고 실행되지 않는다.

# deadlock test

## purpose

두 transaction 각각 하나의 row에 lock을 걸고 각각의 transaction이 lock이 걸린 row에 write를 하는 경우 deadlock err가 발생하는 것을 확인한다.

## prerequisite

```sql
play-ground=# select id, name from account where id = 1 or id = 2;
 id | name
----+-------
  1 | john
  2 | jacob
(2 rows)
```

## test

* session A

```sql
play-ground=# start transaction;
START TRANSACTION
play-ground=# update account set name = 'tom' where id = 1;
UPDATE 1
```

* session B

```sql
START TRANSACTION
play-ground=# update account set name = 'lucas' where id = 2;
UPDATE 1
```

* session A

```sql
play-ground=# update account set name = 'jackson' where id = 2;
...(locking)
```

* session B

```sql
play-ground=# update account set name = 'peter' where id = 1;
ERROR:  deadlock detected
DETAIL:  Process 31106 waits for ShareLock on transaction 2383786; blocked by process 30713.
Process 30713 waits for ShareLock on transaction 2383787; blocked by process 31106.
HINT:  See server log for query details.
CONTEXT:  while updating tuple (0,21) in relation "account"
```

* session A

```sql
...(unlock)
UPDATE 1
play-ground=# commit;
COMMIT
play-ground=# select id, name from account where id = 1 or id = 2;
 id |  name
----+---------
  1 | tom
  2 | jackson
```

* session B

```sql
play-ground=# commit;
ROLLBACK
```

## conclusion

session A는 id 1에 lock을 걸고 session B는 id 2에 lock을 걸고 있는 상황에서 session A가 id 2에 lock을 시도하면 waiting이 발생한다. 그 상황에서 session B가 id 1에 lock을 시도하면 두 session B는 dead lock err가 발생하게 된다. session A의 transaction은 commit되지만 session B의 transaction은 rollback이 된다.
