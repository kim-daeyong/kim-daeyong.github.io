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
primary는 주로 write/read를 하고 replication을 하며 secondary는 read를 합니다.  
구성된 pss에서 p는 지속적으로 s에 replication을 실행하여 동일한 data를 갖도록 합니다.  
p와 s들은 서로 Heartbeat를 날려 서로의 상태를 확인하다 p에 문제 생긴걸 파악하면 s가 p를 대신하여 Primary로 변경됩니다.    
이때 만약 장애가 한번 더 발생한다면 s가 한개밖에 남지 않기때문에 서버장애가 발생하게됩니다.  
이를 방지하려면 밑에 서술하려는 Arbiter가 존재한다면 하나 남은 secondary가 primary로 변경됩니다.  

Secondary에서 데이터를 읽거나 등등의 쿼리를 하려면 오류가 뜨며 불가능한데 이런 쿼리들은 P에서 가능합니다.
rs.slaveOk()로 설정 후 확인 가능합니다.  

2. PSA 방식  
![image](https://user-images.githubusercontent.com/45562285/82333611-eaac9a80-9a21-11ea-8ff3-cb5b8e274adb.png)
PSA 방식은 Primary와 Secondary, Arbiter로 구성됩니다.  
P와 S는 위와 동일한 기능을 하며 Arbiter는 Heartbeat를 지속적으로 체크하며 Primary를 선출하는 기능만 합니다.  

P가 선출되는 기본적인 시간은 12초입니다.  

---

### MongoDB Sharded cluster

Mongodb의 Replica Set을 이용하여 Sharded cluster를 구성할 수 있습니다.  
Sharded cluster는 많은 양의 데이터를 받아 분산처리가 필요할 경우 사용합니다.  

![image](https://user-images.githubusercontent.com/45562285/82446081-eb583600-9ae0-11ea-845a-bfe0dcc3e238.png)  

sharded cluster는 여러 shard들로 구성되는데 이 shard들은 replica set으로 구성되어 있습니다.  
이 sharding 서비스에는 3가지 member(rs.status()로 member를 확인가능합니다.)로 구성됩니다.  

1. mongos : 클라이언트나 서버에서 요청을 받는 라우터 입니다.
            이 mongs가 쿼리를 받아 configserver에서 sharding에 대한 데이터를 받아 각 shard에 chunck를 분배하고 결과를   리턴합니다.
            (chunk는 쪼개진 데이터입니다.)  
            mongos는 각 클라이언트, 서버당 하나씩 구성되는게 좋다고 합니다.  

2. configserver : configserver 에선 sharded cluster에 구성된 shard들에 대한 meta data를 가지고 있습니다.  
                  이를 이용해 mongos에서 shard로 배분하고, shard 된 데이터를 불러옵니다.
                  보통 3개이상으로 구성하는게 좋다고 합니다.  
                  또한 각 데이터에 대한 정보와 샤드에 대한 정보를 갖기에 어느정도 용량이 필요합니다.  

3. Shard : replica set으로 구성된 shard입니다.

sharding은 db의 collection 단위로 이루어 집니다.  
mongodb는 autosharding을 지원하나 sharding할 정보를 설정해줘야 합니다.
그렇기에 sharding 할 db, collection을 설정해줘야 하며, index를 설정해줘야 합니다.  
정보를 입력하지않는다면 분산처리가 되지않고 특정 shard에 들어갑니다.

DB지정 - db.runCommand({enablesharding:"DB Name"}) / sh.enableSharding("DB Name")  
Collection지정 - db.runCommand({shardcollection:"DB Name.Collection Name",key:{ _id:hashed}}) /  
                sh.shardCollection("test.things", {_id : "hashed"})  
                (admin 에서 진행) 마지막 id는 인덱싱을 지정해주는 설정입니다.  
shard 상태 확인 - printShardingStatus();  

---

mongodb를 구성해보며 이론이나 원리에 대해 정말 큰 필요성을 느꼈습니다.  
