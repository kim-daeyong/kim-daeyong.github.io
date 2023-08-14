---
layout: post 
title: Aws Profile 추가
subtitle: 
gh-repo: 
gh-badge: [star, fork, follow]
tags: [AWS, Shell, Infra, Other]
categories : [Infra]
toc: true
---


### TRY  
aws cli 에서 여러가지 프로파일을 사용할 수도 있는 상황이 생긴다.  
이때 추가하는 방법.  

### CATCH  

* profile 추가  

```
$ aws configure --profile "profile name"

이후 나오는 순서대로 입력

  AWS Access Key ID [None]:
  AWS Secret Access Key [None]: 
  Default region name [None]: 
  Default output format [None]: 
```  


* 확인  

```
$ cat ~/.aws/config

[default]
region = ap-northeast-2
[profile "profile name"]
region = ap-northeast-2

$ cat ~/.aws/credentials  

[default]
aws_access_key_id = ~~~
aws_secret_access_key = ~~~
["profile name"]
aws_access_key_id = ~~~
aws_secret_access_key = ~~~
```  

### FINALLY  
IAM 마다 다른 권한을 가지고 있다면 local에서 사용할떄 여러 access key등을 사용하게 된다.  
이걸 환경변수 등에 넣자니 좋지않고 또 yml이나 다른 곳에 넣자니 좋지않다.  
물론 instance에 올라간 애들은 InstanceProfile로 사용할 수 있지만 로컬환경에선 제한되기에 ProfileProvider로 간편하게 사용하기위해 여러 프로파일을 등록하였다.  


끝

---
