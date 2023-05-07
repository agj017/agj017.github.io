---
layout: post
title: memo for jpa
date: 2023-03-26
category: memo
---

# How to log sql statement with parameter on spring

## [com.github.gavlyukovskiy:p6spy-spring-boot-starter:1.5.8](https://github.com/gavlyukovskiy/spring-boot-data-source-decorator) dependency 추가

* gradle

```gradle
implementation("com.github.gavlyukovskiy:p6spy-spring-boot-starter:${version}")
```

* maven

```maven
<dependency>
    <groupId>com.github.gavlyukovskiy</groupId>
    <artifactId>p6spy-spring-boot-starter</artifactId>
    <version>${version}</version>
</dependency>
```

