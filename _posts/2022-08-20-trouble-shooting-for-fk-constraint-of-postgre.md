---
layout: post
title: trouble-shooting-for-fk-constraint-of-postgre
date: 2023-08-20
category: trouble-shooting
---

# what happened

auto ddl 문제는 자동으로 테이블을 생성해주다보니 외래키 관련 에러가 많이 발생한다. 

```
errno: 150 "Foreign key constraint is incorrectly formed"
```

# reason

JPA의 auto ddl는 DB의 Foreign key constraint를 고려하지 않고 DB에 sql statement를 보낸다.

# solution

DB의 Foreign key constraint 파악하여 constraint를 해결한다. 아래는 DB의 key 정보를 파악하는 sql이다.

```
// 스키마 전제에 대한 키 정보를 확인하는 경우 
SELECT * FROM information_schema.key_column_usage WHERE constraint_schema = '스키마명';

// 특정테이블에 대한 키 정보를 확인하는 경우
SELECT * FROM information_schema.key_column_usage WHERE  table_name = '테이블명';
```