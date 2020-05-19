---
layout: post
title: MongoDB cluster
subtitle: 
gh-repo: 
gh-badge: [star, fork, follow]
tags: MongoDB
categories : [DB]
---

### MongoDB HA

MongoDB는 failover등 장애사항이나 데이터 유실을 방지하기위해 ReplicaSet을 지원합니다.  
이는 두가지 방법을 제공하는데  
1. PSS 방식  
![image](https://user-images.githubusercontent.com/45562285/82333284-7bcf4180-9a21-11ea-80b9-5ad378349f10.png)  
여기서 말하는 p는 Primary, s는 secondary 입니다.  
primary는 주로 write/read를 하며 secondary는 read를 주로 합니다.  
구성된 pss에서 p는 지속적으로 s에 replication을 실행하여 동일한 data를 갖도록 합니다.  
p와 s들은 서로 Heartbeat를 날려 서로의 상태를 확인하다 p에 문제 생긴걸 파악하면 s가 p를 대신하여 Primary로 변경됩니다.    
이때 만약 장애가 한번 더 발생한다면 s가 한개밖에 남지 않기때문에 서버장애가 발생하게됩니다.  
이를 방지하려면 밑에 서술하려는 Arbiter가 존재한다면 하나 남은 secondary가 primary로 변경됩니다.  

재밌는 점은 Secondary에서 데이터를 읽거나 어떤 행동으로 하려면 오류가 뜨며 행동을 할 수 없고 이런 행동들은 P에서 가능합니다.

2. PSA 방식  
![image](https://user-images.githubusercontent.com/45562285/82333611-eaac9a80-9a21-11ea-8ff3-cb5b8e274adb.png)
PSA 방식은 Primary와 Secondary, Arbiter로 구성됩니다.  
P와 S는 위와 동일한 기능을 하며 Arbiter는 Heartbeat를 지속적으로 체크하며 Primary를 선출하는 기능만 합니다.  

P가 선출되는 기본적인 시간은 12초입니다.  

---

### MongoDB Sharded cluster