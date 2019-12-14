---
layout: post
title: lazy,OSiV
subtitle: 
gh-repo: 
gh-badge: [star, fork, follow]
tags: [SpringBoot, Jpa]
comments : true
categories : [Spring]
---

Spring Web MVC

자동으로 Bean으로 등록된다.
DispatcherSerlvet
ViewResolver(spring-boot-starter-thymeleaf를 추가하여 타임리프를 사용할 수 있도록한다.)
....

~~~
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>
        <dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-thymeleaf</artifactId>
		</dependency>
~~~


Spring DATA Jpa

DataSource (Hikari CP)
PlatformTransactionManager
EntityManager

~~~
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-data-jpa</artifactId>
		</dependency>

		<dependency>
			<groupId>mysql</groupId>
			<artifactId>mysql-connector-java</artifactId>
			<version>5.1.47</version>
		</dependency>

		<dependency>
			<groupId>com.h2database</groupId>
			<artifactId>h2</artifactId>
			<version>1.4.197</version>
			<scope>test</scope>
		</dependency>
~~~

Web & JPA 를 함께 사용하면

OSiV 패턴 적용된다.
OSiV를 적용하고 싶지 않다면?
spring.jpa.open-in-view=false
JPA 관계와 관련 Annontation

lazy로딩을 하려면 트랜잭션안에서만 가능하다.
ex> Employee e = employeeRepository.getEmployee(1L);
Job job = e.getJob(); // lazy 로딩 : select * from job where id = ?
@OneToOne : FetchType.EAGER
@OneToMany : FetchType.LAZY
@ManyToOne : FetchType.EAGER
@ManyToMany : FetchType.LAZY

JPQL : SELECT e FROM Employee e



참고 : 1 + n 문제 : http://wonwoo.ml/index.php/post/975

fetch join
@BatchSize(size = 5)
@Fetch(FetchMode.SUBSELECT)