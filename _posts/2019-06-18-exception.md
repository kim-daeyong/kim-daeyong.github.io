---
layout: post
title: 예외처리, 예외종류
subtitle: 
gh-repo: 
gh-badge: [star, fork, follow]
tags: [Study, Spring]
categories : [Other]
---

일단 에러와 예외가 있는데
에러는 개발자가 처리할 수 없다.(JVM, 하드웨어적인 문제)

예외(Exception)는 사용자의 잘못된 조작 또는 개발자의 잘못된 코딩으로 인해 발생하는 프로그램 오류를 말한다. 

자바는 실행중 Exception 발생 시 처리하는 걸 제공한다.   
이때 세가지 방법이 있는데.   
1) 메소드 내에서 직접처리하는 방법이있는데   
try, catch, final은 메소드내에서 예외처리 시에 사용된다.  
예로
~~~
try{

    예외 발생 가능성이 있는 문장들;

}catch(예외 타입1 매개변수명){

    예외타입1의 예외가 발생할 경우 처리 문장들;

}catch(예외 타입 n 매개변수명){

    예외타입 n의 예외가 발생할 경우 처리 문장들;

}finally{

    항상 수행할 필요가 있는 문장들;

}

~~~


2) 또 예외를 메소드를 호출한 다른 곳으로 예외처리를 보내버릴 수 있는데   
예외가 발생한 지점에서 try-catch 또는 try-catch-finally 블록을 이용하여 직접 처리하지 않아도 예외가 발생한 메소드를 호출한 지점으로 예외를 전달하여 처리하는 방법이 있다.   
이때 (throws) 가 사용된다.  

예로  
~~~
void 메소드() throws ClassNotFoundException{
~~~
호출받은 곳에선 try문ㅇ로 메소드를 호출하고 catch문으로 exception을 받아 처리해줘야한다.

3) 또 사용자가 exception class를 작성하여 발생 시킬 수있다.  
사용자가 예외 클래스를 정의하려면 예외 클래스의 최상위 클래스인 java.lang.Exception 클래스를 상속받아 클래스를 정의한다.

~~~
class 예외 클래스 이름 extends Exception
~~~

자바 가상 머신은 프로그램 수행중에 예외가 발생하면 자동으로 해당하는 예외 객체를 발생시킨 다음 catch 블록을 수행한다.
하지만 예외는 사용자가 강제적으로 발생시킬 수도 있다. 자바는 예외를 발생시키기 위해 throw 예약어를 사용한다.

~~~
throw new 예외 클래스 이름(매개변수);
~~~

throw를 이용한 예외 발생시에도 try-catch-finally 구문을 이용한 예외 처리를 하거나, throws를 이용하여 예외가 발생한 메소드를 호출한 다른 메소드로 넘기는 예외 처리 방법을 사용해야 한다.


---

예외의 종류는

1. 일반 exception과 (컴파일)  

2. runtimeException 이 있다.(실행중)

~~~
ClassNotFoundException 클래스를 찾지 못함 
CloneNotSupportedException Cloneable 인터페이스 미구현 
IllegalAccessException 클래스 접근을 못함 
InstantiationException 추상 클래스 또는 인터페이스를 인스턴스화 하고자 할때 
InterruptedException 쓰레드가 중단 되었을때 
NoSuchFieldException 지정된 필드가 없을때 
NoSuchMethodException 지정된 메소드가 없을때 
[IOException] CharConversionException 문자 변환에서 예외가 발생했을때 
[IOException] EOFException 파일의 끝에 도달했을때 
[IOException] FileNotFoundException 파일이 발견되지 않았을때 
[IOException] InterruptedIOException 입출력 처리가 중단 되었을때 
[IOException][ObjectStreamException] InvalidClassException 시리얼라이즈 처리에 관한 문제가 클래스 안에 있을때 [IOException][ObjectStreamException] InvalidObjectException 시리얼라이즈된 오브젝트에서 입력 검증에 실패했을때 [IOException][ObjectStreamException] NotActiveException 스트림 환경이 액티브하지 않을 때 메소드를 호출했을때 [IOException][ObjectStreamException] NotSerializableException 오브젝트를 시리얼라이즈 할 수 없을때 
[IOException][ObjectStreamException] OptionalDataException 오브젝트를 읽을때 기대 이외의 데이터와 만났을때 
[IOException][ObjectStreamException] StreamCorruptedException 읽은 데이터 스트림이 파손되어 있을때 
[IOException][ObjectStreamException] WriteAbortedException 기록중에 예외가 발생한 스트림을 읽었을때 
[IOException] SyncFailedException 시스템 버퍼를 동기시키는 FileDescriptor.sync()의 호출 실패시 
[IOException] UnsupportedEncodingException 지정된 문자 부호화 형식을 지원하고 있지 않을때 
[IOException] UTFDataFormatException 부정한 UTF-8방식 문자열과 만났을때 
[RuntimeException] ArithmeticException 제로제산 등의 산술 예외 발생시 
[RuntimeException] ArrayStoreException 배열에 부정한 형태의 오브젝트를 저장하고자 할때 
[RuntimeException] [IllegalArgumentException] IllegalThreadStateException 쓰레드가 요구된 처리를 하기에 적합한 상태에 있지 않을때   
[RuntimeException] [IllegalArgumentException] NumberFormatException 부적절한 문자열을 수치로 변환하고자 할때 [RuntimeException] IllegalMonitorStateException 모니터 상태가 부정일때 
[RuntimeException] IllegalStateException 메소드가 요구된 처리를 하기에 적합한 상태에 있지 않을때 
[RuntimeException] [IndexOutOfBoundException] ArrayIndexOutOfBoundsException 범위 밖의 배열 첨자 지정시 [RuntimeException] [IndexOutOfBoundException] StringIndexOutOfBoundsException 범위 밖의 String 첨자 지정시 [RuntimeException] NegativeArraySizeException 음의 크기로 배열 크기를 지정하였을때 
[RuntimeException] NullPointerException null 오브젝트로 접근했을때 
[RuntimeException] SecurityException 보안 위반시 
[RuntimeException] UnsupportedOperationException 지원되지 않는 메소드를 호출했을때 

@ Error 
[LinkageError] ClassCircularityError 클래스 초기화중에 순환 참조를 검출시 
[LinkageError] [ClassFormatError] UnsupportedClassVersionError JVM이 지원되지 않는 버전의 번호를 가진 클래스 파일을 읽고자 할때 
[LinkageError] ExceptionInInitializerError 정적 이니셜라이저로 예외가 발생시 
[LinkageError] [IncompatibleClassChangeError] AbstracMethodError 추상 메소드를 호출했을때 
[LinkageError] [IncompatibleClassChangeError] IllegalAccessError 접근할 수 없는 메소드와 필드를 사용하고자 했을때 [LinkageError] [IncompatibleClassChangeError] InstantiationError 인터페이스 또는 추상 클래스를 인스턴스화하고자 했을때 [LinkageError] [IncompatibleClassChangeError] NoSuchFieldError 지정한 필드가 존재하지 않을때 
[LinkageError] [IncompatibleClassChangeError] NoSuchMethodError 지정한 메소드가 존재하지 않을때 
[LinkageError] NoClassDefFoundError 클래스 정의가 발견되지 않았을때
[LinkageError] UnsatisfieldLinkError 클래스에 포함되는 링크 정보를 해결할 수 없을때 
[LinkageError] VerifyError 클래스 파일안에 부적절한 부분이 있을때 ThreadDeath 쓰레드가 정지해야만 한다는 의미 [VirtualMachineError] InternalError 내부에러 
[VirtualMachineError] OutOfMemoryError 메모리부족으로 메모리를 확보 못함 
[VirtualMachineError] StackOverflowError 스택 오버 발생 
[VirtualMachineError] UnknownError 심각한 예외발생

출처: https://silverbullet.tistory.com/entry/Exception의-종류와-발생원인 [계속 계속 공부하자]


~~~