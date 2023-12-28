---
layout: post 
title: Aws Lambda 배포 및 direct(sdk) trigger(spring boot3) 
subtitle: 
gh-repo: 
gh-badge: [star, fork, follow]
tags: [Java, Spring, Cloud, Aws, Lambda]
categories : [Programming]
toc: true
---


### TRY  
스프링 부트3에서 Aws Lambda를 사용하고 싶어졌다.  
그리고 API gateway나 다른 트리거가 아닌 코드에서 호출하고 싶었다.  

### CATCH  
* Lambda(Java 환경) 정보
    ![https://docs.aws.amazon.com/ko_kr/lambda/latest/dg/lambda-java.html](https://github.com/kim-daeyong/kim-daeyong.github.io/assets/45562285/58205752-48ca-4491-b600-24c220925554)  
    [Aws Docs](https://docs.aws.amazon.com/ko_kr/lambda/latest/dg/lambda-java.html)  

* Spring Cloud 와 Spring boot 버전 호환 정보
    ![https://spring.io/projects/spring-cloud#overview](https://github.com/kim-daeyong/kim-daeyong.github.io/assets/45562285/597bb98f-6fdb-445b-a8ba-808d45f50f2d)  
    [Spring Cloud Docs](https://spring.io/projects/spring-cloud#overview)  

    스프링부트 3 에선 `2022.0` 이후 버전을 사용해야 한다.  
    나는 `2022.0.4` 버전을 사용하였다.

* Lambda Function  
    Lambda Funtion은 `Function`, `Consumer`, `Suppiler`로 구현될 수 있으며  
    properties에 `spring.cloud.function.definition`을 명시하지않으면 Funtion, Consumer, Supplier 순으로 구현된 Lambda Function을 찾는다고 한다.  

    예)

    ```java 

    @Configuration
    public class MyFunctionConfig {

        @Bean
        public Function<String, String> myFunction() {
            return s -> "Hello, " + s;
        }

        @Bean
        public Consumer<String> myConsumer() {
            return s -> System.out.println("Consumed: " + s);
        }

        @Bean
        public Supplier<String> mySupplier() {
            return () -> "Supplier Output";
        }
    }

    //application.properties
    spring.cloud.function.definition=myFunction
    ```

    나는 자바 Function을 사용하였다.  
    테스트 코드는 다음과 같다.  
    
    * 먼저 Gradle

        ```gradle
        plugins {
            id 'java'
            // 테스트 스프링부트 버전은 3이다.
            id 'org.springframework.boot' version '3.1.5'
            id 'io.spring.dependency-management' version '1.1.3'
        }

        group = 'com.example.demo'
        version = '0.0.1-SNAPSHOT'

        java {
            sourceCompatibility = '17'
        }

        configurations {
            compileOnly {
                extendsFrom annotationProcessor
            }
        }

        repositories {
            mavenCentral()
        }

        ext {
            // Spring cloud 버전
            set('springCloudVersion', "2022.0.4")
        }

        dependencies {
            // 로컬 테스트 용이다.
            // localhost:8080/function 이름으로 호출 가능하게해준다.
            implementation 'org.springframework.cloud:spring-cloud-starter-function-web'

            // Spring cloud
            implementation 'org.springframework.cloud:spring-cloud-function-adapter-aws'
            implementation 'com.amazonaws:aws-lambda-java-core:1.2.1'
            implementation 'com.amazonaws:aws-lambda-java-events:3.9.0'

            // Aws Lambda SDK다.
            implementation group: 'software.amazon.awssdk', name: 'lambda', version: '2.21.21'
            
            testImplementation 'org.springframework.boot:spring-boot-starter-test'
        }

        // 프로젝트를 zip 파일로 빌드하기위해.
        // build/distributions 아래 저장된다.
        task buildZip(type: Zip) {
            from compileJava
            from processResources
            into('lib') {
                from configurations.runtimeClasspath
            }
        }

        dependencyManagement {
            imports {
                mavenBom "org.springframework.cloud:spring-cloud-dependencies:${springCloudVersion}"
            }
        }

        tasks.named('test') {
            useJUnitPlatform()
        }

        ```

    * Test Functuion  

        테스트는 간단하게 request의 숫자가 100보다 작은지 구분하는 함수이다.  

        Request  
        ```java
            public class NumberRequest {
                private int number;

                public int getNumber() {
                    return number;
                }

                public int setNumber() {
                    return number;
                }

                public NumberRequest() {

                }

                public NumberRequest(int num) {
                    this.number = num;
                }
                
            }

        ```

        response
        ```java
            public class NumberResponse {
                private String result;

                public String getResult() {
                    return result;
                }

                public String setResult() {
                    return result;
                }

                public NumberResponse(String result) {
                    this.result = result;
                }
            }

        
        ```

        Lambda function 이다.
        ```java
            @Configuration
            public class CheckNumberFunction {
                
                @Bean
                public Function<NumberRequest, NumberResponse> checkNumber() {
                    return (number) -> {
                        if (number.getNumber() >= 100) {
                            return new NumberResponse(number.getNumber() + " 은(는) 100보다 크거나 같습니다.");
                        }
                
                        return new NumberResponse(number.getNumber() + " 은(는) 100보다 작습니다.");
                    };
                }
            }
        
        ```

        자바 Function으로 만들었고 Bean으로 등록되어야한다.  

        이제 bootrun을 해보고 `localhost:8080/checkNumber` 로 날려보면 정상적으로 동작하는 것을 확인할 수 있다.

        <img width="641" alt="image" src="https://github.com/kim-daeyong/kim-daeyong.github.io/assets/45562285/bd83bc0c-37b7-420b-b223-936589ec4b9a">

        이제 gradle task를 실행해 zip 파일을 만들고 람다를 생성해서 올리면 된다.   


        이때 이전버전엔 handler를 따로 구현해줬으나 이젠 필요없다.  
        [Handler](https://docs.spring.io/spring-cloud/docs/current/reference/htmlsingle/#aws-request-handlers)  

        ```
        org.springframework.cloud.function.adapter.aws.functioninvoker::handleRequest
        ```
        를 Lambda Handler 칸에 넣어주자

        또 환경변수에 FUNCTION_NAME과 MAIN_CLASS를 넣어준다. (Configurtaion - Environment variables - Edit)
        이때 `FUNCTION_NAME`은 만든 AWS Function의 Bean 이름을 , `MAIN_CLASS`에는 main 메서드가 위치한 클래스명을 입력해준다.  

        ![image](https://github.com/kim-daeyong/kim-daeyong.github.io/assets/45562285/3f2883f2-e56b-48b7-bbd9-9f170ca9d52d)

        이후 테스트 해보면 동작한다.  

    * Direct trigger  

        Lambda는 다양한 트리거를 가질 수 있다.  
        S3, CloudFront, Api gateway 등등..  

        나는 코드에서 요청을 받아 직접 호출하려고 한다.  
        테스트 코드는 다음과 같다.  

        controller
        ```java
            @RestController
            @RequestMapping("/api/lambda")
            public class TestController {
                private final TestService testService;

                public TestController(TestService testService) {
                    this.testService = testService;
                }

                @GetMapping()
                public String requestLambdaJob() throws JsonProcessingException {
                    return testService.invokeFunction(); 
                }
            }
        ```

        service

        ```java

            @Service
            public class TestService {
                public String invokeFunction() throws JsonProcessingException {
                    String result = "";
                    LambdaClient awsLambda = LambdaClient.builder()
                        .region(Region.AP_NORTHEAST_2)
                        .credentialsProvider(ProfileCredentialsProvider.create("sdk"))
                        .build();

                    InvokeResponse res = null ;
                    try {
                        NumberRequest numberRequest = new NumberRequest(1000);

                        ObjectMapper objectMapper = new ObjectMapper();

                        SdkBytes payload = SdkBytes.fromUtf8String(objectMapper.writeValueAsString(numberRequest)) ;

                        // Setup an InvokeRequest.
                        InvokeRequest request = InvokeRequest.builder()
                            .functionName("checkNumber")
                            .payload(payload)
                            .build();

                        res = awsLambda.invoke(request);
                        result = res.payload().asUtf8String() ;
                        System.out.println(result);

                        return result;

                    } catch(LambdaException e) {
                        System.err.println(e.getMessage());
                        System.exit(1);
                    }

                    return result;
                }  
            }
        
        ```

        서비스는 간단하게 [여기](https://docs.aws.amazon.com/lambda/latest/dg/example_lambda_Invoke_section.html) 를 참조 하였다.  

        이후 `localhost:8080/api/lambda` 를 호출해보면.

        <img width="437" alt="image" src="https://github.com/kim-daeyong/kim-daeyong.github.io/assets/45562285/7b59e734-57b4-4496-9e3a-326bd0fae3bc">

        다음과 같이 확인해볼 수 있다.  

        아 먼저 일단 Aws lambda 권한을 가진 iam이 필요하다.  


### FINALLY  

음 끝났다.  

끝

---
