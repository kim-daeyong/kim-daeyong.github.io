---
layout: post 
title: MongoDB Replica Set With docker-compose(Mac)
subtitle: 
gh-repo: 
gh-badge: [star, fork, follow]
tags: [MongoDB, Shell, Infra, Docker, Other]
categories : [Infra]
toc: true
---


### TRY  
저번에 Spring Data Mongo에서 @Transactional을 사용하려면 MongoDB 가 replica set이 구성되어 있어야 한다고 했다.  

local에서 test하기 위해 이를 구성하여 하는데 docker-compose를 이용하기로 생각했다.  
docker-compose를 사용하는 이유는 관리하기도 쉽고 공유하기도 쉬워서 선택했다.  

### CATCH  
먼저 완성한 yml과 sh 파일이다.

```
version: '3.7'

services:
  mongo1:
    hostname: mongo1
    container_name: mongo1
    image: arm64v8/mongo:6.0
    ports:
      - "27017:27017"
    volumes:
      - ./data1:/data/db
      - /yourpath/security.keyFile:/etc/security.keyFile
    environment:
      - MONGO_INITDB_ROOT_USERNAME=root
      - MONGO_INITDB_ROOT_PASSWORD=1234
      - MONGO_INITDB_DATABASE=yourdb
      - MONGO_REPLICA_SET_NAME=rs0
      - MONGO_PRIMARY=mongo1
    privileged: true
    command: ["mongod", "--replSet", "rs0", "--bind_ip_all", "--keyFile", "/etc/security.keyFile"]

  mongo2:
    hostname: mongo2
    container_name: mongo2
    image: arm64v8/mongo:6.0
    ports:
      - "27018:27017"
    volumes:
      - ./data2:/data/db
      - /yourpath/security.keyFile:/etc/security.keyFile
    environment:
      - MONGO_INITDB_ROOT_USERNAME=root
      - MONGO_INITDB_ROOT_PASSWORD=1234
      - MONGO_INITDB_DATABASE=yourdb
      - MONGO_REPLICA_SET_NAME=rs0
      - MONGO_PRIMARY=mongo1
    privileged: true
    command: ["mongod", "--replSet", "rs0", "--bind_ip_all", "--keyFile", "/etc/security.keyFile"]

  mongo3:
    hostname: mongo3
    container_name: mongo3
    image: arm64v8/mongo:6.0
    ports:
      - "27019:27017"
    volumes:
      - ./data3:/data/db
      - /yourpath/security.keyFile:/etc/security.keyFile
    environment:
      - MONGO_INITDB_ROOT_USERNAME=root
      - MONGO_INITDB_ROOT_PASSWORD=1234
      - MONGO_INITDB_DATABASE=yourdb
      - MONGO_REPLICA_SET_NAME=rs0
      - MONGO_PRIMARY=mongo1
    privileged: true
    command: ["mongod", "--replSet", "rs0", "--bind_ip_all", "--keyFile", "/etc/security.keyFile"]
```

```
#!/bin/bash

rm -rf ./data1 ./data2 ./data3

openssl rand -base64 741 > /yourpath/security.keyFile
chmod 600  /yourpath/security.keyFile

docker-compose up -d

echo "Waiting for MongoDB instances to start..."
sleep 10

mongosh --username=root --password=1234 --eval <<EOF
  rs.initiate({
    _id: "rs0",
    members: [
      { _id: 0, host: "host.docker.internal:27017" },
      { _id: 1, host: "host.docker.internal:27018" },
      { _id: 2, host: "host.docker.internal:27019" }
    ]
  })
EOF

echo "Replica Set configured"
```

* 실행  
    ```
    $ sh script.sh
    ```

이제 만났던 에러들을 나열해 본다.  

* zsh: command not found: mongo  

    oh my zsh

    역시 날 만나러 와줬다.  

    처음 script는 mongo 를 이용해 replica set을 구성하려고 했었다.  

    하지만 내가 사용하는 버전은 mongodb-community@6.0 였으므로 deprecated 되었고 mongosh를 사용하라고 되어있었다.  

    [참고](https://github.com/dotnet/AspNetCore.Docs/issues/25379)

    그래서 mongosh로 변경하였다.  

    아

    난 brew를 통해 mongodb-community@6.0를 설치하였다.  

    ```
    $ brew tap mongodb/brew

    $ brew update

    $ brew install mongodb-community@6.0

    ```

* NoReplicationEnabled  

    ```
    {
            "ok" : 0,
            "errmsg" : "This node was not started with replication enabled.",
            "code" : 76,
            "codeName" : "NoReplicationEnabled"
    }

    ```

    이 에러는 기존에 구성해두었던 volume으로 생성된 데이터들이 mongoDB의 파일들을 삭제하지않아서 발생하였다.  

    이를 삭제 하였다.  

    이것과 함께 

    ```
    {"error":"UPGRADE PROBLEM: Found an invalid featureCompatibilityVersion document (ERROR: Location4926900: Invalid featureCompatibilityVersion document in admin.system.version: { _id: \"featureCompatibilityVersion\", version: \"7.0\" }. See https://docs.mongodb.com/master/release-notes/5.0-compatibility/#feature-compatibility. :: caused by :: Invalid feature compatibility version value, expected '5.0' or '5.3' or '6.0. See https://docs.mongodb.com/master/release-notes/5.0-compatibility/#feature-compatibility.). If the current featureCompatibilityVersion is below 5.0, see the documentation on upgrading at https://docs.mongodb.com/master/release-notes/5.0/#upgrade-procedures."}
    ```

    이 에러도 같이 발생하였는데 기존 구성 파일이 mongoDB 다른 버전을 사용하고있었다.  

* Unauthorized

    ```
    {
            "ok" : 0,
            "errmsg" : "Command replSetInitiate requires authentication",
            "code" : 13,
            "codeName" : "Unauthorized"
    }
    ```  

    이 에러는 replSetInitiate을 하는데 authentication이 필요하다는 에러다.  

    user와 password를 추가해 주었다.  

    ```
    mongosh --username=root --password=1234 --eval <<EOF
    rs.initiate({
        ...
    })
    EOF

    ```

* Connection refused

    ```
    MongoServerError: replSetInitiate quorum check failed because not all proposed set members responded affirmatively: localhost:27018 failed with Error connecting to localhost:27018 (127.0.0.1:27018) :: caused by :: Connection refused, localhost:27019 failed with Error connecting to localhost:27019 (127.0.0.1:27019) :: caused by :: Connection refused
    ```

    이 에러는 localhost를 사용해서 그렇다.  

    docker에 올라간 호스트의 다른 컨테이너들을 연결하려면 `host.docker.internal` 를 통해 접근할 수 있다.  

* Authentication failed
    이는 keyFile이 필요해서 그렇다.  
    위에 security.keyFile을 생성하는 부분으로 해결하였다.  

    그리고..

* mongodb compass ENOTFOUND {host}

    난 간단하게 데이터 확인하고 할때는 mongodb의 compass를 사용하고 있었는데 localhost를 사용해도, 컨테이너 이름을 사용해도 `ENOTFOUND` 에러가 발생하였다.  
    찾아보니 나만 그런게 아니었고.[참고](https://www.mongodb.com/community/forums/t/cannot-connect-to-replica-set-via-mongo-compass/165856/27)  

    다음과 같이 hosts 파일에 추가해서 해결하였다.  

    ```
    s sudo vim /etc/hosts

    add

    127.0.0.1       mongo1
    127.0.0.1       mongo2
    127.0.0.1       mongo3

    ---


    mongosh --username=root --password=1234 --eval <<EOF
    rs.initiate({
        _id: "rs0",
        members: [
        { _id: 0, host: "mongo1:27017" },
        { _id: 1, host: "mongo2:27018" },
        { _id: 2, host: "mongo3:27019" }
        ]
    })
    EOF

    ---

    or

    add

    127.0.0.1       host.docker.internal

    ---

    mongosh --username=root --password=1234 --eval <<EOF
    rs.initiate({
        _id: "rs0",
        members: [
        { _id: 0, host: "host.docker.internal:27017" },
        { _id: 1, host: "host.docker.internal:27018" },
        { _id: 2, host: "host.docker.internal:27019" }
        ]
    })
    EOF
    ---

    ```  

    입맛대로 골라 넣으면 된다.  

### FINALLY  

이것 외에도 정말 다양한 걸 만났고 정말 긴 시간이었다.

kubernetes를 사용했다면 helm을 이용해 간단하게 했을텐데..  

그냥 k8s 사용할까 하고 계속 고민 했다.  

하지만 나만 잘쓰자고 하는게 아니고 러닝커브도 있고 공유용이니까 열심히 했다.  

k8s & helm 쓰세요!  

끝

---
