---
layout: post 
title: Aws Instance Monitoring with Prometheus and Grafana
subtitle: 
gh-repo: 
gh-badge: [star, fork, follow]
tags: [AWS, Monitoring, Spring, Prometheus, Grafana, Infra]
categories : [Infra]
toc: true
---


### TRY
배포된 서버의 모니터링은 중요하다.  
서버의 상태는 각 인스턴스나 LB등에서 확인가능하나 하나씩 들어가 확인하기 너무 귀찮고 ui도 별로다.  
원하는 metrics와 ui로 확인하고 싶어진다.  
그라파나와 프로메테우스를 사용해보자.  

### CATCH

* prometheus.yml  
먼저 데이터를 수집할 Prometheus의 설정 파일을 만들어 준다.  

```
global:
    scrape_interval: 15s 
    # evaluation_interval: 15s

scrape_configs:
  - job_name: 'node_exporter'
    metrics_path: '/metrics'
    scrape_interval: 15s
    static_configs: # 고정된 ip와 port를 통해 metrics를 수집하고 싶다면.
    - targets: ['{ip}:{port}']

  - job_name: 'node_exporters'
    metrics_path: '/metrics'
    ec2_sd_configs: # node_exproter가 배포되어있는 ec2 instance 들의 metrics를 수집하고 싶다면.
     - region: ap-northeast-2
       port: 9100
       access_key: ~~~
       secret_key: ~~~
         #filters:
         #- name : tag:Name
         #values:
         # - ~~~
    #relabel_configs: # 기본적으로 private ip가 설정되기에 relabel이 필요하다면 추가해야한다.
    #  - source_labels: [__meta_ec2_tag_Name]
    #    target_label: instance
    #    action: keep
    #  - source_labels: [__meta_ec2_public_ip]
    #    regex: '(.*)'
    #    replacement: '${1}:9100'
    #    target_label: __address__
  
  - job_name: 'jvm_exporter'
    metrics_path: '/actuator/prometheus'
    ec2_sd_configs: # spring ec2 instance 들의 actuator metrics를 수집하고 싶다면.
     - region: ap-northeast-2
       port: 8080
       access_key: ~~~
       secret_key: ~~~
         #filters:
         #- name : tag:Name
         # values:
         # - ~~~
    #relabel_configs: # 기본적으로 private ip가 설정되기에 relabel이 필요하다면 추가해야한다.
    #  - source_labels: [__meta_ec2_tag_Name]
    #    target_label: instance
    #    action: keep
    #  - source_labels: [__meta_ec2_public_ip]
    #    regex: '(.*)'
    #    replacement: '${1}:9100'
    #    target_label: __address__
``` 

* node_exporter 설치  
다음 각 인스턴스의 metrics를 export 해줄 node_exporter를 설치해준다.  
만약 docker 컨테이너의 메트릭스를 확인하고 싶다면 `cAdvisor`를 사용해도 좋다.  
도커 컴포즈등을 통해 도커로 설치도 가능하다.  
다만 예전에 docker로 설치했을때 도커 컨테이너의 메트릭스를 주고 호스트의 메트릭스를 주지않았던 기억이 있어서 이렇게 설치하는 방법만 적어놨다.  

```
    $ wget https://github.com/prometheus/node_exporter/releases/download/v1.2.2/node_exporter-1.2.2.linux-amd64.tar.gz

    $ tar xvzf node_exporter-1.2.2.linux-amd64.tar.gz

    $ mv node_exporter-1.2.2.linux-amd64 node_exporter

    $ cd node_exporter

    $ node_exporter &
```  

* docker-compose.yml  
다음은 Prometheus와 Grafana를 설치할 docker compose 이다.  
시스템의 서비스등으로 올릴 수도 있겠으나 아무래도 도커가 관리하기가 훨씬 편하다.  

```
version: '3.7'
services:
  prometheus:
    image: prom/prometheus
    container_name: prometheus
    privileged: true
    volumes:
      - {prometheus.yml path}/prometheus.yml:/etc/prometheus/prometheus.yml
      - {your path}/prometheus/volume:/prometheus
    ports:
      - 9090:9090 # 포트
    command:
      - '--web.enable-lifecycle'
      - '--config.file=/etc/prometheus/prometheus.yml'
    restart: always
    network_mode: "host"
  grafana:
    image: grafana/grafana
    container_name: grafana
    ports:
      - 3000:3000 # 포트
    volumes:
      - {your path}/grafana/volume:/var/lib/grafana
    restart: always
    network_mode: "host"
  #cAdvisor:
  #  image: gcr.io/google-containers/cadvisor:v0.33.0
  #  restart: always
  #  privileged: true
  #  volumes:
  #   - /:/rootfs:ro
  #   - /var/run:/var/run:rw
  #   - /sys/fs/cgroup:/sys/fs/cgroup:ro
  #   - /dev/disk/:/dev/disk:ro
  #  ports:
  #   - 8080:8080
```  

* Spring actuator

```
dependencies {  
    // prometheus
    implementation("org.springframework.boot:spring-boot-starter-actuator")  
    implementation("io.micrometer:micrometer-registry-prometheus")  
}
```  

```
management:  
  endpoints:  
    prometheus:  
      enabled: true  
    web:  
      exposure:  
        include: 
          - prometheus
```  

이후 프로메테우스의 Targets Tab에서 원하는 job들이 정상적으로 실행되는지 확인한다.  

만약 static_config로 프로메테우스 설정을 해뒀다면 private ip를 적어야한다.  

public으론 수집이 되지않았다.  

또 만약 Security Group을 사용하고 있다면 inbound 설정을 해줘야한다.  

다음 Grafana에서 DataSource를 등록 후 대쉬보드를 구성하면 된다.  

대쉬보드 구성에 부담된다면 [Grafana-dashboard-lab](https://grafana.com/grafana/dashboards/) 에서 마음에 드는 대쉬보드를 받아 커스터마이징 하자.  

### FINALLY 

사실 Aws를 사용한다면 대부분의 데이터는 CloudWatch등의 모니터링을 통해 대부분 확인가능하다.  

비용이 문제일 뿐이지  

끝

---
