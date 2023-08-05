---
layout: post
title: test-concurrency-update-of-spring-jpa
date: 2023-08-05
category: test-for-spring-jpa
---

# prerequisite

* DB version

```shell
 gyujinan@GYUJINui-MacBookPro  ~  postgres --version

postgres (PostgreSQL) 11.20
```

* Entity

```java
@Entity
public class Account {

    @Id
    public UUID id;

    public UUID organizationId;

    public String name;

    public Account() {
        id = UUID.randomUUID();
    }
}
```

* refer to [test code](https://github.com/agj017/play-ground/blob/main/spring/jpa/src/test/java/com/example/jpa/lock/LockTestCaseTest.java)

# update by second transaction 

## purpose
* transaction 시작과 상관없이 second update가 first update를 처리를 waiting한 후, 발생함을 확인한다.  

## prerequisite

## test
1. 두 transaction이 concurrency 상황에서 update는 경합이 발생한다. (lock 발생)
2. isolation level이 READ_COMMITED인 경우 second transaction의 update는 frist transaction이 commit이 발생할 때까지 waiting한다.  
3. waiting이 끝난 second transaction은 발생한다.
4. 결과적으로 second transaction의 update가 적용된다. 

* concurrency case code

```java
@Transactional
    public void testSecondTxWaiteLockFromFirstTx(UUID accountId, String nameToUpdate, boolean isFirstTx, boolean isSecondTx) throws InterruptedException {
        Account account = accountRepository.findById(accountId).get();
        System.out.println("isFirstTx: " + isFirstTx + ", isSecondTx: " + isSecondTx + ", name: " + account.name);
        if (isFirstTx) {
            account.name = nameToUpdate;
            em.flush();
            Thread.sleep(3000);
        }

        if (isSecondTx) {
            Thread.sleep(1000);
            account.name = nameToUpdate;
        }
    }
```

* test code

```java
@Test
    public void testSecondTxWaiteLockOfFirstTx() throws InterruptedException {
        // given
        UUID accountId = UUID.randomUUID();
        accountRepository.save(new Account(accountId, null, "before updating"));

        // when
        new Thread(() -> {
            try {
                lockTestCase.testSecondTxWaiteLockFromFirstTx(accountId, "first tx", true, false);
            } catch (InterruptedException e) {
                throw new RuntimeException(e);
            }
        }).start();
        lockTestCase.testSecondTxWaiteLockFromFirstTx(accountId, "second tx", false, true);

        // then
        Account account = accountRepository.findById(accountId).get();
        assertEquals("second tx", account.name);
    }
```

## conclusion
* account name은 second transaction의 결과인 "second tx"로 저장됨


# update by first transaction 

## purpose
* transaction 시작과 상관없이 first update가 second update를 처리를 expired한 후, 발생함을 확인한다.  

## prerequisite

## test
1. 두 transaction이 concurrency 상황에서 update는 경합이 발생한다. (lock 발생)
2. isolation level이 REPEATABLE_READ 경우 second transaction의 update는 frist transaction이 commit이 발생할 때 CannotAcquireLockException를 throw하므로서 expired 된다. 
3. 결과적으로 first transaction의 update가 적용된다. 

* concurrency case code

```java
@Transactional(isolation = Isolation.REPEATABLE_READ)
    public void testCannotAcquireLockException(UUID accountId, String nameToUpdate, boolean isFirstTx, boolean isSecondTx) throws InterruptedException {
        Account account = accountRepository.findById(accountId).get();
        System.out.println("isFirstTx: " + isFirstTx + ", isSecondTx: " + isSecondTx + ", name: " + account.name);
        if (isFirstTx) {
            account.name = nameToUpdate;
            em.flush();
            Thread.sleep(3000);
        }

        if (isSecondTx) {
            Thread.sleep(1000);
            account.name = nameToUpdate;
        }
    }
```

* test code

```java
@Test
    public void testCannotAcquireLockException() throws InterruptedException {
        // given
        UUID accountId = UUID.randomUUID();
        accountRepository.save(new Account(accountId, null, "before updating"));

        // when
        new Thread(() -> {
            try {
                lockTestCase.testCannotAcquireLockException(accountId, "first tx", true, false);
            } catch (InterruptedException e) {
                throw new RuntimeException(e);
            }
        }).start();

        // then
        assertThrows(CannotAcquireLockException.class, () -> {
            lockTestCase.testCannotAcquireLockException(accountId, "second tx", false, true);
        });
        Account account = accountRepository.findById(accountId).get();
        assertEquals("first tx", account.name);
    }
```

## conclusion
* account name은 frist transaction의 결과인 "first tx"로 저장됨