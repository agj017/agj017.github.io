---
layout: post
title: test-lost-update-by-spring-jpa
date: 2023-08-08
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
    public String address;

    public Account() {
        id = UUID.randomUUID();
    }

    public Account(UUID id, UUID organizationId, String name, String address) {
        this.id = id;
        this.organizationId = organizationId;
        this.name = name;
        this.address = address;
    }
}
```

* refer to [test code](https://github.com/agj017/play-ground/blob/main/spring/jpa/src/test/java/com/example/jpa/lock/LockTestCaseTest.java)

# lost update by second transaction 

## purpose
* transaction 시작과 상관없이 second update가 first update를 처리를 waiting함을 확인하고 first update는 select의 시차에 의해 lost 된다.  

## prerequisite

## test
1. 두 transaction이 concurrency 상황에서 update는 경합이 발생한다. (lock 발생)
2. isolation level이 READ_COMMITED인 경우 second transaction의 update는 frist transaction이 commit이 발생할 때까지 waiting한다.
3. waiting이 끝난 후 second transaction의 update가 발생한다.
4. second transaction의 update가 적용된다.
5. first transaction의 update lost은 second transaction의 update에 반영되지 않았으므로 lost된다.

* concurrency case code

```java
    @Transactional
    public void testLostUpdateOfREAD_COMMITTED(UUID accountId, boolean isFirstTx, boolean isSecondTx) throws InterruptedException {
        Account account = accountRepository.findById(accountId).get();
        System.out.println("isFirstTx: " + isFirstTx + ", isSecondTx: " + isSecondTx + ", name: " + account.name + ", address: " + account.address );
        if (isFirstTx) {
            account.name = "nameToUpdate";
            em.flush();
            Thread.sleep(3000);
        }

        if (isSecondTx) {
            Thread.sleep(1000);
            account.address = "addressToUpdate";
        }
    }
```

* test code

```java
    @Test
    public void testLostUpdate_when_isolation_level_READ_COMMITTED() throws InterruptedException {
        // given
        UUID accountId = UUID.randomUUID();
        accountRepository.save(new Account(accountId, null, "before updating", "before updating"));

        // when
        new Thread(() -> {
            try {
                lockTestCase.testLostUpdateOfREAD_COMMITTED(accountId, true, false);
            } catch (InterruptedException e) {
                throw new RuntimeException(e);
            }
        }).start();
        lockTestCase.testLostUpdateOfREAD_COMMITTED(accountId, false, true);

        // then
        Account account = accountRepository.findById(accountId).get();
        assertEquals("before updating", account.name);
        assertEquals("addressToUpdate", account.address);
    }
```

## conclusion
* account name는 first transaction에 의해 update 되었으므로 update되지 않았다.


# lost update by first transaction 

## purpose
* transaction 시작과 상관없이 first update가 second update를 처리를 expired함을 확인하고 second update는 select의 시차에 의해 lost 된다.

## prerequisite

## test
1. 두 transaction이 concurrency 상황에서 update는 경합이 발생한다. (lock 발생)
2. isolation level이 REPEATABLE_READ 경우 second transaction의 update는 frist transaction이 commit이 발생할 때 CannotAcquireLockException를 throw하므로서 expired 된다.
3. first transaction의 update가 적용된다.
4. second transaction의 update는 lost 된다.

* concurrency case code

```java
    @Transactional(isolation = Isolation.REPEATABLE_READ)
    public void testLostUpdateOfREPEATABLE_READ(UUID accountId, boolean isFirstTx, boolean isSecondTx) throws InterruptedException {
        Account account = accountRepository.findById(accountId).get();
        System.out.println("isFirstTx: " + isFirstTx + ", isSecondTx: " + isSecondTx + ", name: " + account.name + ", address: " + account.address );
        if (isFirstTx) {
            account.name = "nameToUpdate";
            em.flush();
            Thread.sleep(3000);
        }

        if (isSecondTx) {
            Thread.sleep(1000);
            account.address = "addressToUpdate";
        }
    }
```

* test code

```java
    @Test
    public void testLostUpdate_when_isolation_level_REPEATABLE_READ(){
        // given
        UUID accountId = UUID.randomUUID();
        accountRepository.save(new Account(accountId, null, "before updating", "before updating"));

        // when
        new Thread(() -> {
            try {
                lockTestCase.testLostUpdateOfREPEATABLE_READ(accountId, true, false);
            } catch (InterruptedException e) {
                throw new RuntimeException(e);
            }
        }).start();

        // then
        assertThrows(CannotAcquireLockException.class, () -> {
            lockTestCase.testLostUpdateOfREPEATABLE_READ(accountId, false, true);
        });
        Account account = accountRepository.findById(accountId).get();
        assertEquals("nameToUpdate", account.name);
        assertEquals("before updating", account.address);
    }
```

## conclusion
* account address는 second transaction에 의해 update 되었으므로 update되지 않았다.