---
layout: post 
title: Java Optional의 orElse 와 orElseGet
subtitle: 
gh-repo: 
gh-badge: [star, fork, follow]
tags: [Java, Optional]
categories : [Java]
toc: true
---


### TRY  
Java의 Optional을 사용하면서 `orElse`와 `orElseGet`, `orElseThrow`는 정말 많이 사용한다.  
그중 평소 `orElse`와 `orElseGet`는 그냥 null 값일때 빈 객체등을 받기위해 사용했었는데  
이번에 `map()`등 `stream api` 메서드들과 사용하면서 의도와는 다르게 동작해 의아했고 또 재밌었다.  
같이 보자.  

### CATCH  
```java

data = dataRepository.findById(id)
                    .map(data -> check(data))
                    .orElse(new Data());

```  

다음과 같이 repository로 데이터를 찾아 데이터가 있다면 체크하고 없다면 새로운 객체를 리턴하려고 했다.  
하지만 데이터가 존재할땐 체크도 하고 새로운 데이터를 리턴 하였다.  

다음 코드를 보자(별로지만 참자)  

```java
public class Prac {
    public static void main(String[] args) {
        Test t = Optional.ofNullable(new Test())
                .map(test -> {
                    test.getData();
                    return test;
                })
                .orElse(new Test("babo"));

        t.getData();
    }
}

class Test {
    private String data = "test";

    Test() {}

    Test(String data) {
        System.out.println(data);
        this.data = data;
    }

    public String getData() {
        System.out.println(this.data);
        return data;
    }
}

```  

결과는 어떨까?  

다음과 같다. 

```java

test
babo
test

```

`map()` 이 실행되고 또 `orElse`까지 실행된다.  

하지만 test 값은 변경되지 않았다.  

이게 아닌데! map()만 실행될 줄 알았는데!  

그럼 어떻게 해결해야 할까?  

`orElseGet` 을 보자.  

```java

public static void main(String[] args) {
    Optional.ofNullable(new Test())
            .map(test -> {
                test.getData();
                return test;
            })
            .orElseGet(() -> new Test("babo"));
}

// result

test

```

`orElseGet`은 의도대로 동작한다.  

차이가 뭘까  

```java
    /**
     * If a value is present, returns the value, otherwise returns
     * {@code other}.
     *
     * @param other the value to be returned, if no value is present.
     *        May be {@code null}.
     * @return the value, if present, otherwise {@code other}
     */
    public T orElse(T other) {
        return value != null ? value : other;
    }

    /**
     * If a value is present, returns the value, otherwise returns the result
     * produced by the supplying function.
     *
     * @param supplier the supplying function that produces a value to be returned
     * @return the value, if present, otherwise the result produced by the
     *         supplying function
     * @throws NullPointerException if no value is present and the supplying
     *         function is {@code null}
     */
    public T orElseGet(Supplier<? extends T> supplier) {
        return value != null ? value : supplier.get();
    }

```  

`orElse`는 파라미터가 항상 호출된다.  
`orElseGet`은 null 경우에만 Supplier 객체를 통해 메소드가 호출된다.  
`orElse`와 비슷하게 테스트 해보면 다음과 같다.  

```java

    public static void main(String[] args) {
        System.out.println(test(anyMethod()));
    }

    private static String value = null;

    private static <T> T test(T other) {
        System.out.println("in test");
        return value != null ? (T) value : other;
    }

    private static String anyMethod() {
        System.out.println("in method");
        return "anyValue`";
    }

    // result 

    in method
    in test
    anyValue`

        
    private static String value = "test";

    private static <T> T test(T other) {
        System.out.println("in test");
        return value != null ? (T) value : other;
    }

    private static String anyMethod() {
        System.out.println("in method");
        return "anyValue`";
    }

    //result
    in method
    in test
    test

```


참고로 Supplier는 함수형 인터페이스다.  

```java

/**
 * Represents a supplier of results.
 *
 * <p>There is no requirement that a new or distinct result be returned each
 * time the supplier is invoked.
 *
 * <p>This is a <a href="package-summary.html">functional interface</a>
 * whose functional method is {@link #get()}.
 *
 * @param <T> the type of results supplied by this supplier
 *
 * @since 1.8
 */
@FunctionalInterface
public interface Supplier<T> {

    /**
     * Gets a result.
     *
     * @return a result
     */
    T get();
}

```


### FINALLY  
물론 사용법이 별로일 순 있다.  
리턴값이 필요 없었다면 ifPresent()나 ifPresentOrGet()을 사용해도 좋은 것 같다.  

끝

---
