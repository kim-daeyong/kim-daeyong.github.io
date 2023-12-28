---
layout: post 
title: Java의 List, Set, Map의 of() static method (Immutable Collections)
subtitle: 
gh-repo: 
gh-badge: [star, fork, follow]
tags: [Java, Spring, JPA, QueryDsl]
categories : [Java]
toc: true
---


## TRY  
지금까지 `List.of`, `Set.of`, `Map.of`를 그냥 Collection을 간단하게 생성할때 사용했었습니다.  
그동안 List등 Of()로 생성한 Object의 요소를 변경하지않았기에 몰랐는데 오늘 메서드를 통해 생성된 Collection 들이 Immutable이란걸 알게 되었습니다.  

충격  

## CATCH  

* Of() static method    
    Jdk 9 부터 추가 된 Factory method 입니다.  
    Immutable 한 collection을 생성합니다.  
    그래서 add, put이나 set등 변경할 수 없게 됩니다.  

    사용할때 다음과 같은 특징을 갖습니다.  
    * null 요소로 만드려고 하면 NullPointerException 이 발생
        <img width="1024" alt="image" src="https://github.com/kim-daeyong/kim-daeyong.github.io/assets/45562285/42776349-f6cc-483d-a550-d816b5a173ab">
    * of()로 생성된 Collection은 불변이므로 요소를 추가 및 변경등을 하면 UnsupportedOperationException 이 발생
        <img width="875" alt="image" src="https://github.com/kim-daeyong/kim-daeyong.github.io/assets/45562285/83120911-02f8-417e-b5e0-b19f2e737f3b">

    docs에는 다음과 같이 써있습니다.(Docs 가서 보자)  
    > Immutable List Static Factory Methods
The List.of() static factory methods provide a convenient way to create immutable lists. 
The List instances created by these methods have the following characteristics:
They are structurally immutable. 
Elements cannot be added, removed, or replaced. 
Calling any mutator method will always cause UnsupportedOperationException to be thrown. 
However, if the contained elements are themselves mutable, this may cause the List's contents to appear to change.
They disallow null elements. Attempts to create them with null elements result in NullPointerException.
They are serializable if all elements are serializable.
The order of elements in the list is the same as the order of the provided arguments, or of the elements in the provided array.
They are value-based. 
Callers should make no assumptions about the identity of the returned instances. 
Factories are free to create new instances or reuse existing ones. 
Therefore, identity-sensitive operations on these instances (reference equality (==), identity hash code, and synchronization) are unreliable and should be avoided.
They are serialized as specified on the Serialized Form page.  

[docs](https://docs.oracle.com/javase%2F9%2Fdocs%2Fapi%2F%2F/index.html?overview-summary.html)  
[geeksforgeeks](https://www.geeksforgeeks.org/java-convenience-factory-methods-for-collections/)  
[openjdk](https://openjdk.org/jeps/269)

## FINALLY  
잘 알고 써야겠다는 생각이 들었습니다.  

끝

---
