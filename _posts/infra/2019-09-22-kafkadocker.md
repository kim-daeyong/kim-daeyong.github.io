---
layout: post
title:  Docker에 Kafka 설치하기
subtitle: 
gh-repo: 
gh-badge: [star, fork, follow]
tags: [Docker, Kafka]
categories : [Infra]
---

간단하게 도커에 카프카를 설치하는 방법을 적어둔다. 이후에 DockerFile 작성이나 EntryPoint등에 대해 적을 것이다.  
도커 우분투를 이용할 것이다..(용량이 가장 적고 빠른건 Alpine-linux이다)

#### 우분투 이미지 

* Docker run -it --name name ubuntu:16.04

#### 필요한 패키지들 설치

Apt-get은 우분투의 패키지를 관리하는 명령어

* Apt-get update - 패키지목록을 갱신하기 위해 씀

* apt-get install net-tools
net-tools를 설치해주어야 ifconfig 명령어를 사용 가능하다.

* apt-get install vim
vi 편집기 사용을 위해 미리 vim도 설치해주자.

* apt-get install iputils-ping
ping 테스트가 필요한 경우 iputils-ping 패키지를 설치한다.

* apt-get install wget
wget 명령어 이용하여 다운로드 받아오기 위해 wget 패키지를 설치한다.

* apt-get install openjdk-11-jdk - openjdk 자바 설치

* ln -s jdk1.8.0_201/ java
해당 폴더에 대해 심볼릭 링크를 설정한다.(이건그냥 참고)

#### 카프카 다운

Wget 파일주소 - 파일 다운로드

* wget http://www-us.apache.org/dist/kafka/2.2.1/kafka_2.12-2.2.1.tgz

압축해제
* tar -zxf kafka_2.12-2.2.1.tgz
* mv kafka_2.12-2.2.1.tgz kafka

#### Kafka 서버 구동

* bin/zookeeper-server-start.sh config/zookeeper.properties

* bin/kafka-server-start.sh config/server.properties 

그냥 시작하면 cmd에서 확인을 할 수가 없다.  
그래서 데몬으로 돌리자    

* bin/zookeeper-server-start.sh -daemon config/zookeeper.properties

* bin/kafka-server-start.sh -daemon config/server.properties

Topic을 생성한다.

* bin/kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic testTopic 

생성된 topic 을 확인한다.

* bin/kafka-topics.sh --list --zookeeper localhost:2181 을 사용하면 확인 할 수 있다.

producer 구동 방법은 다음과 같다.(producer는 topic에서 받아온 메세지를 broker 보낼 수 있다.)

* bin/kafka-console-producer.sh --broker-list localhost:9092 --topic TestTopic
>

이런 식으로 나오는데 저기다 원하는 글을 적으면 Kafka에 저장된다.(> hello world)

kafka console consumer(consumer는 broker에 저장된 메세지를 땡겨온다)를 구동해 보겠다.  

* bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic TestTopic --from-beginning  

 hello world

이런식으로 보여진다.

---

간단하게 Kafka의 기본 구동까지의 과정을 봤다.
이를 사용하기위한 방법중 dockerfile을 작성하여 구동까지의 방법을 다음에 적어본다.