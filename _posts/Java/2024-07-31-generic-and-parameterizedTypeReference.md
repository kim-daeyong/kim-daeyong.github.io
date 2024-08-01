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
Java의 `Generic`은 컴파일 시 `Object`로 변경되거나 혹은 `bound type(<E extends ExampleObject>)` 같은 경우 `ExampleObject` 로 변경됩니다..  
그렇다면 `ParameterizedTypeReference` 는 어떻게 동작하는걸까요?  

---


### CATCH  

* Generic erasure  
    먼저 제네릭 소거는 다음과 같이 제네릭이 사용된 경우 컴파일 시 소거됩니다.
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
    ```

    이외에 적절한 곳에 Type Casting이 추가되거나 Bridge method가 추가됩니다.  

이처럼 generic은 소거 되는데 어떻게 ParameterizedTypeReference는 타입을 가지고 있을까요

* ParameterizedTypeReference  
    `ParameterizedTypeReference` 는 Spring에서 제공하는 `Super Type Token`이라는 걸 이용한 클래스입니다..  

    이는 익명 내부 클래스를 이용하여 타입 정보를 캡처하고 전달합니다.   
    `ParameterizedTypeReference`는 익명 클래스로 생성되고 내부에서 `getGenericSuperclass` 메소드를 통해 인스턴스 타입의 상속관계 상 부모 타입에 대한 정보를 가져와 타입을 저장합니다.

    `Restteplate`에서는 사용되는 예제로 다음과 같이 익명클래스로 생성하여 사용됩니다.  

    ```java
    restTemplate.exchange(
        url,
        HttpMethod.GET,
        HttpEntity.EMPTY,
        new ParameterizedTypeReference<Object> {}
    );

    ```

    ParameterizedTypeReference 클래스를 보자면 다음과 같습니다.

    ```java
    // abstract를 강제
    public abstract class ParameterizedTypeReference<T> {

	private final Type type;


	protected ParameterizedTypeReference() {
		Class<?> parameterizedTypeReferenceSubclass = findParameterizedTypeReferenceSubclass(getClass());
		Type type = parameterizedTypeReferenceSubclass.getGenericSuperclass(); // getGenericSuperClass 메소드를 통해 type을 저장
		Assert.isInstanceOf(ParameterizedType.class, type, "Type must be a parameterized type");
		ParameterizedType parameterizedType = (ParameterizedType) type;
		Type[] actualTypeArguments = parameterizedType.getActualTypeArguments();
		Assert.isTrue(actualTypeArguments.length == 1, "Number of type arguments must be 1");
		this.type = actualTypeArguments[0];
	}

    private ParameterizedTypeReference(Type type) {
		this.type = type;
	}


    public Type getType() {
        return this.type;
    }

	public static <T> ParameterizedTypeReference<T> forType(Type type) {
		return new ParameterizedTypeReference<>(type) {};
	}

	private static Class<?> findParameterizedTypeReferenceSubclass(Class<?> child) {
		Class<?> parent = child.getSuperclass();
		if (Object.class == parent) {
			throw new IllegalStateException("Expected ParameterizedTypeReference superclass");
		}
		else if (ParameterizedTypeReference.class == parent) {
			return child;
		}
		else {
			return findParameterizedTypeReferenceSubclass(parent);
		}
	}

    ```

    * 참고
        - [docs](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/core/ParameterizedTypeReference.html)
        - [Neal Gafter's blog](https://gafter.blogspot.com/2006/12/super-type-tokens.html)



---

### FINALLY  
`ParameterizedTypeReference` 을 사용하며 어떻게 타입을 가져오는지에 대해 생각해 본 적 없었는데 재밌었습니다.  

끝

---
