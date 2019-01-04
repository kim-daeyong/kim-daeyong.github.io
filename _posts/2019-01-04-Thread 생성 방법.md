---
layout: post
title: Thread 생성 방법 
subtitle:
gh-repo:
gh-badge: [star,fork,follow]
tags: [JAVA]
---

쓰레드를 생성하는 방법은 크게 2가지가 있다.

Thread 클래스를 상속하거나 Runnable<interface>를 쓰는 방법이다.

1. Thread를 상속하는 방법 

Thread 클래스 상속

THread

start()
run()

메소드를 가지고 있다.

2. Runnable인터페이스를 사용하는 방법.

<interface>
runnable

run()

-메소드만 가지고 있다(start메소드가 업다.)


Thread 클래스 

start()

Thread(runnable) - runnable의 자식을 갖는다(인터페이스는 인스턴스를 가질수 없기에)

*Thread t = new Thread(new MyThread)


(implements)

MyThread

run() - 코드 구현


쓰레드 클래스를 생성하여 자식(runnable 인터페이스를 구현하고있는 클래스)의 값을 넣어 쓰레드를 생성하고 스타트한다.
