---
layout: post 
title: Java generic erasure와 ParameterizedTypeReference
subtitle: 
gh-repo: 
gh-badge: [star, fork, follow]
tags: [Java, Spring]
categories : [Programming]
toc: true
---


### TRY  
Java의 `Generic`은 컴파일 시 `Object`로 변경되거나 혹은 `bound type(<E extends ExampleObject>)` 같은 경우 `ExampleObject` 로 변경된다.  
그렇다면 `ParameterizedTypeReference` 는 어떻게 동작하는걸까?  

---


### CATCH  

* Generic erasure  
    먼저 제네릭 소거는 다음과 같이 제네릭이 사용된 경우 컴파일 시 소거된다.
    ```java
    // 소거 전
    public class ExamClass<T> {
        private T value;

        public void set(T value) {
            this.value = value;
        }

        public T get() {
            return value;
        }
    }

    // 소거 후

    public class Exam<T> {
        private Object value;

        public void set(Object value) {
            this.value = value;
        }

        public Object get() {
            return value;
        }
    }

    // bouding
    public class ExamClass<T extends Exam> {
        private T value;

        public void set(T value) {
            this.value = value;
        }

        public T get() {
            return value;
        }
    }

    public class ExamClass<T extends Exam> {
        private Exam value;

        public void set(Exam value) {
            this.value = value;
        }

        public Exam get() {
            return value;
        }
    }

    이외에 적절한 곳에 Type Casting이 추가되거나 Bridge method가 추가된다.  

*  



    ```

---

### FINALLY  
`ParameterizedTypeReference` 에 대해 생각해 본적 없었는데 재밌는 경험이었습니다.

끝

---
