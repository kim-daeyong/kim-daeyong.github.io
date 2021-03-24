---
layout: post
title: web jar, 부트스트랩적용
subtitle: 
gh-repo: 
gh-badge: [star, fork, follow]
tags: [Spring ,CICD]
comments : true
categories : [Java]
---

jquery, bootstrap  

프론트 라이브러리도 maven, gradle을 추가할 수 있다.  

~~~
		<!-- webjar 프론트단을 불러오는 라이브러리-->
		<dependency>
			<groupId>org.webjars</groupId>
			<artifactId>jquery</artifactId>
			<version>3.3.1-2</version>
		</dependency>
		<dependency>
			<groupId>org.webjars</groupId>
			<artifactId>bootstrap</artifactId>
			<version>4.2.1</version>
		</dependency>
~~~

html문서 안에서 다음과 같이 사용  

정적파일들 : css, image ....  

ex> /resources/static/css/blog-home.css    