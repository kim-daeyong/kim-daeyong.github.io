---
layout: post
title:  Kafka-Docker image를 위한 Dockerfile 작성
subtitle: 
gh-repo: 
gh-badge: [star, fork, follow]
tags: [Docker, Kafka]
categories : [Other]
---


## Kafka-Docker image를 위한 Dockerfile 작성

1. 환경
2. Dockerfile 및 Shell script 작성
3. build
4. 생성된 image로 run

---

>환경

host : alpine-linux  
Kafka : 2.2.1  
jdk : openjdk 8  

---
>Dockerfile 및 shell script 작성

Dockerfile
~~~
# 베이스 이미지
FROM alpine

# 기본 베이스 패키지 설치
# no cache -> apk를 run할때 이미지화하여 저장해두기때문에 용량이 늘어나는걸 방지하기위해 적용
RUN apk --no-cache update && \
	apk add --no-cache bash && \
	apk add --no-cache openjdk8 && \
	wget http://www-us.apache.org/dist/kafka/2.2.1/kafka_2.12-2.2.1.tgz && \
	tar -zxf kafka_2.12-2.2.1.tgz 

ADD docker-entrypoint.sh / 
ENTRYPOINT ["./docker-entrypoint.sh"]

CMD ["/kafka/bin/kafka-server-start.sh", "/kafka/config/server.properties"]
~~~

docker-entrypoint.sh
~~~
#!/bin/bash

# TARGET - Volume으로 잡혀있는 Kafka폴더에서 
# fs_count - 파일이 있는지
# ds_count - 폴더가 있는지 확인한다.
TARGET="./kafka"
fs_count=$(ls -Rl $TARGET | grep ^- | wc -l)
ds_count=$(ls -Rl $TARGET | grep ^d | wc -l)

# 파일이나 폴더가 없다면 설치파일을 복사한다.
if [ "$fs_count" = "0" -o "$ds_count" = "0" ]; then

mv ./kafka_2.12-2.2.1/* ./kafka
exec "$@"

# 있다면 그냥 진행한다.
else

exec "$@"

fi
~~~

docker-entrypoint를 작성해준 이유는 kafka를 옮긴 후 Volume을 잡을때 사용중이던 kafka 폴더를 날려버리는 걸 방지하기위해..


---

>bulid

~~~
// sh에 대한 권한 설정을 해줘야 한다.
chmod +x docker-entrypoint.sh

// docker 빌드
docker build -t kafka_sample [Dockerfil경로]

//확인
docker images
~~~


---
>run

~~~
//zookeeper가 먼저 실행된 후 run해야 합니다.

// 도커 run
docker run -dit --name kafka_test -v /data/test/:/kafka/ -p 9092:9092 kafka_sample

~~~
-d : 데몬으로 실행

-v : 볼륨설정 host의 폴더와 container 안의 폴더를 선언해준다.  
	하지만 -v를 사용하면 docker에서 volume ls 등으로 추적할 수 없다고 한다.  
	이를 추적하고자 한다면 dockerfile에서 작성하는 것이 좋을 것 같다.

-p : port 설정


사용할때 리스너나 사용할 포트를 server.properties에 넣는 스크립트를 짜서 -e를 이용해 환경변수로 넣어주는 것도 좋은 것 같다.  
-e로 환경변수를 넣어줘서 shell script에서 port나 listener등의 서버설정을 변경해 줄 수 있었다.  

---
>코멘트

그냥 간단하게 kafka image를 dockerfile을 이용해 작성하는 방법을 적어봤다.  
도커를 공부할수록 리눅스나 다른 것들에 대해 좀 많이 알아야 잘 쓸 수 있겠다는 생각이 든다  