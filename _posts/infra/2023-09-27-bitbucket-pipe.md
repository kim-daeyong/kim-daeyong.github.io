---
layout: post 
title: BitBucket Pipeline with code deploy(feat S3)
subtitle: 
gh-repo: 
gh-badge: [star, fork, follow]
tags: [Bitbucket, Git, CICD]
categories : [Infra]
toc: true
---


### TRY  
`Bitbucket`에선 `Git Action` 같은 `Pipeline` 이라는 게 있다.  
Git Action은 사용해봤는데 Bitbucket Pipeline은 처음 써봤다.  
Code Deploy를 통해 배포해보자.  

### CATCH  

1. Bitbucket Pipeline 생성  
    ![image](https://github.com/kim-daeyong/kim-daeyong.github.io/assets/45562285/5c63e45b-56e8-4e18-bd72-04968bef73f6) 
    다음과 같이 본인의 레파지토리에 들어가 파이프라인 탭에 들어가 파이프라인을 생성한다.  
    본인이 원하는 걸 클릭하고 옵션을 `deploy to s3`로 해준다.  
    ![image](https://github.com/kim-daeyong/kim-daeyong.github.io/assets/45562285/991a42d3-2f92-428b-a24d-c536df4e3f2f)  
    다음과 같이 보이는데 각 내용은 다음과 같다.  
        
    ```shell
            image: gradle:jdk17-jammy         #자신에게 알맞는 image를 사용 jdk, node, ...

            pipelines:
                default:                      #모든 브랜치에서 동작
                    - step:                   #이 pipeline의 각 step을 의미
                        name: Build and Test  #이름
                        caches:               #Gradle or npm 등의 cache 설정
                        - gradle
                        script:               #Shell Script 처럼 해당 서비스 Build 로직 수행 
                        - cd app
                        - chmod +x gradlew
                        - ./gradlew wrapper --gradle-version 8.0 build
                - step:
                    name: Lint the node package
                    script:
                        - npm install eslint
                        - npx eslint src
                    caches:
                        - node
                branches:                     #특정 브랜치에서 동작
                    master:                   #master 브랜치에서 동작
                    - parallel:               #안에있는 각 Step이 병렬적으로 돌아가게 해줌
                        - step:
                            name: Build and Test
                            caches:
                            - gradle
                            script:
                            - chmod +x gradlew
                            - apt-get update && apt-get install -y zip
                            - ./gradlew wrapper --gradle-version 8.0 clean build
                            - ...
                            - #codedeploy를 쓰니 appspec과 각 단계에서 실행될 script 파일을 잘 챙기자 
                            artifacts:       #결과물 경로
                            - {$BUILD_FILE_PATH}
                        - step:
                            ...
                        - step:
                            ...
                    - step:
                        name: Deploy to S3
                        deployment: Master           #Pipeline 탭 밑에 deployments에서 표시되는 이름
                        trigger: manual              #수동 여부 / 자동으로 실행되지않는다.
                        clone:
                            enabled: false
                        script:
                            # sync your files to S3
                            - pipe: atlassian/aws-s3-deploy:{version}
                            variables:
                                AWS_ACCESS_KEY_ID: {$AWS_ACCESS_KEY_ID}
                                AWS_SECRET_ACCESS_KEY: {$AWS_SECRET_ACCESS_KEY}     
                                AWS_DEFAULT_REGION: {$AWS_DEFAULT_REGION}   
                                S3_BUCKET: {$S3_OBJECT_KEY}   
                                LOCAL_PATH: {$BUILD_FILE_PATH}         #script에서 빌드된 결과물의 경로(S3에 올릴 파일)
                pull-requests:           #특정 PR에서 동작
    ```  
    민감한 정보나 변경되니 않는 값들은 Repository Setting에서 Repository variables에 넣어놓자.  

2. 배포하고 테스트  
    Pipeline tab에서 Run Pipeline을 눌러 Repository의 Branche와 위에서 생성한 Pipeline을 설정하고 실행한다.  
    S3에 실제로 올라갔는지 확인해보자  
    다만 code deploy를 사용하기에 빌드한 프로젝트엔 appspec.yml과 각 hook step에 알맞은 script등이 있어야 한다. 
    
    appspec.yml 
    ``` shell
    version: 0.0
    os: linux
    files:
    - source: {source file}
        destination: {source file 위치}
        permissions:
        - object:  {source file 위치}
            pattern: "build"
            owner: {OS User}
            group: {OS User Group}
            mode: 755
            type:
            - file
        file_exists_behavior: OVERWRITE
    hooks:
    ValidateService: # validation 용
        - location: hello.sh # validation에 사용할 파일 위치
        timeout: 180 # 각 스탭 작업 시간
        runas: {OS User}
    ApplicationStart: # product app 실행
        - location: start.sh # product app 실행에 사용할 파일 위치
        timeout: 300
        runas: {OS User}
    ApplicationStop: # product app 종료
        - location: shutdown.sh # product app 종료에 사용할 파일 위치
        timeout: 300
        runas: {OS User}
        
    ```

3. Code deploy 설정  
    1. IAM 설정  
        S3와 code deploy 권한 추가
    2. Code deploy의 applications 탭에서 애플리케이션을 생성
        ![image](https://github.com/kim-daeyong/kim-daeyong.github.io/assets/45562285/200fc7e1-0bd9-4cb5-9e26-f0efcf56b57b)
    3. 이후 Deployment Group 생성  
        ![image](https://github.com/kim-daeyong/kim-daeyong.github.io/assets/45562285/b89ea154-4fe4-4eaa-b82d-bd45cbbe83c9)
        ![image](https://github.com/kim-daeyong/kim-daeyong.github.io/assets/45562285/ab653935-6fca-4173-b134-49c418a3274b)
        1. 배포 그룹 이름 : 배포 그룹 이름 ( 규칙 : [서비스명]-[env]-codedeploy-group )
        2. 서비스 역할 : 위에서 만들어둔 IAM role 사용
        3. 배포 유형 :  
                In-Place : EC2를 더 생성 하지 않고 배포  
                Blue/Green : EC2만큼 생성해서 교체하는 방식으로 배포
        4. 환경구성 : 배포 형태  
        5. 배포설정: 몇개씩 배포 할것인지
        6. 로드 밸런서 : 로드 밸런서 선택
        7. 트리거
        8. 경보
        9. 롤백 : 배포하다가 실패 시 어떻게 할건지 

4. Code Pipeline 설정  
    ![image](https://github.com/kim-daeyong/kim-daeyong.github.io/assets/45562285/98b3ca12-44cd-4eab-913f-faeac8208dad)
    1. 파이프라인 이름
    2. Role name : 생성해주거나 생성해준 Role 설정
    3. adcanced setting > Custom location ( S3 버켓 선택 )
    4. Encryption key는 뭐 그냥 디폴트로 해줌
    ![image](https://github.com/kim-daeyong/kim-daeyong.github.io/assets/45562285/d6541d69-4598-49d4-85eb-579fa157f771)  
    5. S3 버켓 설정
    6. 빌드 스테이지 : 빌드 스테이지는 Bitbucket pipeline에서 했으니 스킵 
    ![image](https://github.com/kim-daeyong/kim-daeyong.github.io/assets/45562285/56baee20-8c3d-4c66-b03a-648a0ea370f7)
    7. Deploy stage 설정
        아까만든 code deploy와 deployment group을 넣어준다.

5. 확인
    repository에서 한번 실행해보자.


* 트러블 슈팅  
    * 파일 잘못된 것을 인지한 후 변경하여 재배포 했는데 이전 것만 읽음.
        * 상황
            1. 스크립트 파일이 잘못된 것을 인지하고 변경하였으나 Fail이 발생.
            2. deployment lifecycle events 에서 View events 확인.
            3. event에서 이전(처음 실패한)의 잘못된 스크립트 파일을 계속 읽는 것을 확인. 
        
        * 해결
            인스턴스 내부의 `/opt/codedeploy-agent/deployment-root/` 에 code deploy에 관련된 파일들이 저장되고 있었음.  
            UUID 같은 이름의 디렉토리(code deploy의 id아닐까) 밑에 d-xxxxxxx라는 디렉토리들이 있었음  
            이는 실패한 `code deploy task id`와 같았다.  
            처음 배포 실패하였던 디렉토리 안에 들어가 `appspec`과 실패한 `shell script`를 변경해주었음.  
            이후 다시 시도했을 때 성공하는 것을 확인  



### FINALLY  
중간중간 확인할 수 있는 방법이 이벤트에서 보여주는 짧막한 것들 밖에 없어서 트러블 슈팅이 힘들었던 것 같다.  
Bitbucket pipeline은 git action과 비슷하지만 참고할만한 자료가 정말 적었다.  
프로세스대로 진행되니 각 단계에세 필요한걸 잘 챙겨두자  

끝

---
