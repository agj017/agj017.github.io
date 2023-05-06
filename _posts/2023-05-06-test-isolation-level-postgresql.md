---
layout: post
title: test-isolation-level-postgresql
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
    name    varchar(255),
    age     integer
);
```

* DML
```sql
insert into account values (1, 'john', 10);
insert into account values (2, 'jacob', 11);
insert into account values (3, 'logan', 12);
insert into account values (4, 'justin', 13);
insert into account values (5, 'tom', 14);
```

# lost update test

## purpose

## prerequisite

## test

## conclusion

# dirty read test

## purpose

isolation level이 read uncommitted인 trasaction이 isolation level이 read committed가 update한 row를 read했지만 updated row가 rollback 되면서 dirty read 현상이 발생하는 것을 확인한다.

## prerequisite

```sql
play-ground=# select id, name from account where id = 1;
 id | name
----+------
  1 | john
(1 row)
```

## test

* session A

```sql
play-ground=# start transaction isolation level read uncommitted;
START TRANSACTION
play-ground=# show transaction isolation level;
 transaction_isolation
-----------------------
 read uncommitted
(1 row)
play-ground=# select id, name from account where id = 1;
 id | name
----+------
  1 | john
(1 row)
```

* session B

```sql
play-ground=# start transaction isolation level read committed;
START TRANSACTION
play-ground=# show transaction isolation level;
 transaction_isolation
-----------------------
 read committed
(1 row)
play-ground=# update account set name = 'brad' where id = 1;
UPDATE 1
play-ground=# select id, name from account where id = 1;
 id | name
----+------
  1 | brad
(1 row)
```

* session A
```sql
play-ground=# select id, name from account where id = 1;
 id | name
----+------
  1 | john
(1 row)
play-ground=# commit;
COMMIT
```

* session B

```sql
play-ground=# commit;
COMMIT
```

## conclusion

session A의 transaction isolation level은 read uncommitted 이므로 updated row에 대해서 dirty read가 발생할 것으로 예상했으나 발생하지 않았다. chat gpt에게 물어본 결과 postgreSQL은 dirty read에 대해서 protection 기능이 있다고 한다. 아래는 gpt의 답변이다.

> This is because PostgreSQL's READ COMMITTED isolation level already provides a certain level of protection against dirty reads. When a transaction reads a row that has been updated by another transaction, but has not yet been committed, PostgreSQL will acquire a "snapshot" of the database at the beginning of the transaction and use this snapshot to provide a consistent view of the data, even if the underlying data changes during the transaction. This means that even if a frist transaction is started with the READ UNCOMMITTED isolation level, it will still see the committed value of the data.

# non repeatable read test

## purpose

first transaction이 실행 도중에 second transaction이 row을 update하여 first transaction이 read한 data가 변경 된 현상을 확인한다. 

## prerequisite

```sql
play-ground=# select id, name from account where id = 1;
 id | name
----+------
  1 | john
(1 row)
```

## test

* session A

```sql
play-ground=# start transaction isolation level read committed;
START TRANSACTION
play-ground=# select id, name from account where id = 1;
play-ground=# show transaction isolation level;
 transaction_isolation
-----------------------
 read committed
(1 row)
play-ground-# ;
 id | name
----+------
  1 | john
(1 row)
```

* session B

```sql
play-ground=# start transaction isolation level read committed;
START TRANSACTION
play-ground=# select id, name from account where id = 1;
play-ground=# show transaction isolation level;
 transaction_isolation
-----------------------
 read committed
(1 row)
play-ground=# update account set name = 'brad' where id = 1;
UPDATE 1
play-ground=# commit;
```

* session A

```sql
play-ground=# select id, name from account where id = 1;
 id | name
----+------
  1 | brad
(1 row)
play-ground=# commit;
COMMIT
```

## conclusion

first transaction이 실행 도중에 second transaction이 해당 row를 update했으므로 repeatable read를 실행했을 때 해당 row가 변경된 것을 확인 할 수 있다.

# phantom read test

## purpose

first transaction이 실행 도중에 second transaction이 row을 insert하여 first transaction이 없던 row 즉 phantom data가 생겼는지 확인한다.

## prerequisite

```sql
play-ground=# select id, age from account where age between 10 and 20;
 id | age
----+-----
  1 |  10
  2 |  11
  3 |  12
  4 |  13
  5 |  14
(5 rows)
```

## test

* session A

```sql
play-ground=# start transaction isolation level read committed;
START TRANSACTION
play-ground=# show transaction isolation level;
 transaction_isolation
-----------------------
 read committed
(1 row)
play-ground=# select id, age from account where age between 10 and 20;
 id | age
----+-----
  1 |  10
  2 |  11
  3 |  12
  4 |  13
  5 |  14
(5 rows)
```

* session B

```sql
play-ground=# start transaction isolation level read committed;
START TRANSACTION
play-ground=# show transaction isolation level;
 transaction_isolation
-----------------------
 read committed
(1 row)
play-ground=# INSERT INTO account (id, name, age) VALUES (6, 'brad', 15);
INSERT 0 1
play-ground=# commit;
COMMIT
```

* session A

```sql
play-ground=# select id, age from account where age between 10 and 20;
 id | age
----+-----
  1 |  10
  2 |  11
  3 |  12
  4 |  13
  5 |  14
  6 |  15
(6 rows)
play-ground=# commit;
COMMIT
```

## conclusion

first transaction이 실행하는 도중 second transaction이 data를 insert 하여서 repeatable하게 read 했을 때, 새로운 row가 생성되었다. 

## notice

first transaction의 isolation level이 repeatable read라도 phantom read가 발생할 것이라 예상했지만 isolation level이 repeatable read일 경우에는 phantom read 현상을 발견할 수 없었다.
