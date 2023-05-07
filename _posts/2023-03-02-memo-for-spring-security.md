---
layout: post
title: memo for spring security
date: 2023-03-02
category: memo
---

# How to check active security filter

* setting

```kotlin
@Configuration
@EnableWebSecurity(debug = true)
class SecurityConfiguration {
}
```

* log

```
************************************************************

Request received for POST '/api':

org.apache.catalina.connector.RequestFacade@34499f5b

pathInfo:null
content-type: application/json
user-agent: PostmanRuntime/7.29.2
accept: */*
postman-token: 72ff2b36-91a7-4b62-bdf8-342333c93d04
host: 
accept-encoding: gzip, deflate, br
connection: keep-alive
content-length: 707


Security filter chain: [
  WebAsyncManagerIntegrationFilter
  SecurityContextPersistenceFilter
  HeaderWriterFilter
  LogoutFilter
  BearerTokenAuthenticationFilter
  RequestCacheAwareFilter
  SecurityContextHolderAwareRequestFilter
  AnonymousAuthenticationFilter
  ExceptionTranslationFilter
]


************************************************************
```

* reference

[https://www.baeldung.com/spring-security-registered-filters](https://www.baeldung.com/spring-security-registered-filters)