---
layout: post
title: Spring Data Mongodb transaction
subtitle: 
gh-repo: 
gh-badge: [star, fork, follow]
tags: [Java, Spring, MongoDB]
categories : [Programming]
toc: true
---


### 내용
Spring Data는 Transaction을 지원합니다.

Mongodb도 transaction을 지원합니다.

다만 RDB와 다르게 Spring에서 자동으로 지원해주진 않고 TransactionManager를 만들어야 합니다.
>Unless you specify a MongoTransactionManager within your application context, transaction support is DISABLED. You can use setSessionSynchronization(ALWAYS) to participate in ongoing non-native MongoDB transactions.

* 참고
[link](https://docs.spring.io/spring-data/mongodb/docs/current/reference/html/#mongo.transactions)
 

* Config
```java
@Configuration
static class Config extends AbstractMongoClientConfiguration {

    @Bean
    MongoTransactionManager transactionManager(MongoDatabaseFactory dbFactory) {  
        return new MongoTransactionManager(dbFactory);
    }
}
```


* Service
```java
@Component
public class StateService {

    @Transactional
    void someBusinessFunction(Step step) {                                        

        template.insert(step);

        process(step);

        template.update(Step.class).apply(Update.set("state", // ...
    };
});
```


다만 Transaction을 사용하려면 특별한 조건이 필요합니다.

만약 코드를 사용하려고 시도할때 다음과 같은 에러를 확인할 수 있습니다.

```sh
com.mongodb.MongoCommandException: Command failed with error 263 (...): 'Transaction numbers are only allowed on a replica set member or mongos' on server localhost:27017. 
The full response is { "ok" : 0.0, "errmsg" : "Transaction numbers are only allowed on a replica set member or mongos", "code" : 263, "codeName" : "..." }
```


>Transaction numbers are only allowed on a replica set member or mongos

Transaction을 사용하려면 mongodb가 replica set으로 구성되어있어야 합니다.


---
