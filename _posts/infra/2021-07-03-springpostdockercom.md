---
layout: post
title: Springboot + postgreSQL docker-compose!
subtitle: 
gh-repo: 
gh-badge: [star, fork, follow]
tags: [Docker,Infra]
comments : [true]
categories : [Infra]
---

### Spring project를 PostgreSQL과 함께 docker-compose로 배포하는 방법

1. 먼저 spring poject 안에 Dockerfile을 만들어준다.

    ``` Dockerfile
    FROM openjdk:12-alpine 
    VOLUME /tmp
    ADD /build/libs/*.jar app.jar 
    ENTRYPOINT ["java","-Djava.security.egd=file:/dev/./urandom","-Dspring.profiles.active=container", "-jar", "/app.jar"]
    ```
    jar를 이용해 프로젝트를 run 하는 방법인데, 프로젝트 자체를 ADD로 복사해  
    'ENTRYPOINT ["./gradlew", "bootRun"]'로 실행하는 방법도 있을 것이다. 


2. 해당 프로젝트를 빌드해 jar를 생성한다.

    Gradle을 사용하였는데 테스트를 제외하고 build하는 command는 다음과 같다.  
    ```
    ./gradlew build --exclude-task test
    ```


3. Docker build를 실행해 image 파일로 만들어 준다.

    ```
    docker build -t testapp .
    ```

4. 다음 docker-compose.yml 파일을 작성해 준다.

    ```docker-compose.yml
    version: "3"
    services:
    postgresdb:
        image: postgres:latest
        container_name: postgresdb
        ports:
        - 5432:5432
        environment:
            - POSTGRES_PASSWORD=postgres
            - POSTGRES_USER=postgres
            - POSTGRES_DB=scope
        restart: unless-stopped

    # APP*****************************************
    springbootapp:
        image: testapp:latest
        container_name: springbootapp
        ports:
        - 8080:8080
        restart: unless-stopped
        depends_on:
        - postgresdb
        links:
        - postgresdb
    volumes:
    postgres-data:
    ```
    * 각 부분 설명
        - version : docker compose의 파일 형식 버전 정보입니다.
        - service : 컴포즈로 구성될 서비스의 정의 입니다.  
                    이 파일의 서비스는 postgresdb, springbootapp 이 있습니다.
        - image : 해당 서비스에 사용될 이미지 입니다.
        - container_name : docker container로 생성됐을때 container의 name 입니다.
        - ports : 사용될 port를 정의합니다. docker run의 -p와 같습니다. host:container 매핑입니다.
        - environment: 해당 서비스의 환경변수를 정의 합니다.
            - 각 서비스의 환경변수를 정의하는데 postgreSQL의 접속정보부터 spirngboot에서 접속정보까지 지정가능합니다.
            - spring에서 접속할 ip(url)과 container가 다르기 때문에 기존에 설정한 ip를 contianer의 name이나 지정한 형태로 변경해야 합니다.
            - spring poroperties 예시 : spring.datasource.url=jdbc:postgresql://localhost:5432/test -> 접속안됨
            - spring poroperties에서 작성 시 : spring.datasource.url=jdbc:postgresql://postgresdb:5432/test
            - 환경변수로 지정 시 : SPRING_DATASOURCE_URL: jdbc:postgresql://postgresdb:5432/test
        - depends_on : 의존성을 추가하는 부분으로 database connection을 요청할때 db가 완전히 로드된 이후 접속을 시도합니다.
        - links : container 간의 연결을 정의합니다.  
        - restart : container에 문제가 발생했을 때 재시작하는 방법을 정의합니다. 

        이외에
        - volumes : 볼륨을 지정합니다 host:container 형태입니다.  
        - networks : network를 지정합니다.
        - build: 빌드에 사용될 사항을 정의합니다. dockerfile: Dockerfile로 도커파일도 지정 가능합니다. context: 로 해당 소스의 path를 지정할 수 있습니다.  
        등이 있습니다.
5. docker-compose를 실행합니다.
    ```
    docker-compose up
    ```