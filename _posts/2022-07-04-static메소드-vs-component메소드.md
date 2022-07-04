---
title: static 메소드 vs @component 메소드 (feat.싱글톤 패턴)
author: Huey-J
date: 2022-07-04 12:00:00 +0800
categories: [backend, spring]
tags: [spring, java]
---

# 들어가며

스프링 프로젝트를 하면서 공통 클래스(예를 들면 유틸 클래스)를 만들 때 보통 static 방식과 @Component방식을 자주 사용합니다.
이번 포스트에서는 이 두 클래스의 차이점과 장단점을 살펴보고 각각 어느 상황에 써야 더 효율적이고 스프링다운지 알아보도록 하겠습니다.

- @Component 어노테이션을 통해 생성한 객체는 빈으로 등록되어 IoC 컨테이너에서 관리됩니다.
- 싱글톤 패턴이란 애플리케이션이 시작될 당시 최초 한번만 메모리를 할당하고 그 메모리에 인스턴스를 만들어 사용하는 디자인 패턴을 말한다. [자세히](https://tecoble.techcourse.co.kr/post/2020-11-07-singleton)


# Static? Singleton?

> 싱글톤패턴(Singleton Pattern)이란 인스턴스가 오직 하나만 생성되는 것을 보장하고 어디에서든 이 인스턴스에 접근할 수 있도록 하는 디자인 패턴 이다.
> *(출처: [해시넷 위키](http://wiki.hash.kr/index.php/싱글톤패턴))*
{: .prompt-info }

@Component 어노테이션을 통해 생성한 객체는 빈으로 등록되어 IoC 컨테이너에서 관리됩니다.
그리고 스프링 컨테이너는 등록된 스프링 빈들을 모두 싱글톤으로 관리합니다.
즉 @Component 어노테이션을 붙힌 클래스는 스프링의 싱글톤 패턴을 통해 관리된다는 의미가 됩니다.
따라서 어디서든 접근 가능한 IoC 컨테이너에 저장이 되며 애플리케이션이 시작될 당시 최초로 한번만 메모리에 할당됩니다.

static 은 인스턴스 변수를 클래스 변수로 만들어주며 이는 곧 클래스와 연관관계를 맺게 됩니다.
이를 통해 '클래스명.메소드명' 을 통해 해당 메소드에 접근할 수 있게 됩니다. 마치 '클래스명.변수명' 처럼 말이죠.
따라서 정적 클래스에는 정적 메소드와 정적 변수가 포함됩니다.


# 차이점

예시로 상황을 그려봅시다. 스프링 프로젝트를 진행하면서 공통으로 사용할 클래스(예를 들면 util 함수)를 추가하고자 합니다.
이 때 우리는 아래와 같이 간단하게 static 클래스를 통해 해결할 수 있습니다.

```java
public class StringUtil {

  public static String getOnlyNumerics(String str) {
    return str.replaceAll("[^0-9]", "");
  }
}

@Service
public class service {

  public void tmp() {
    System.out.println(StringUtil.getOnlyNumerics("hello 123 world"));
  }
}
```

또는 @component 어노테이션을 붙혀 빈으로 등록하거나 static 클래스로 선언하는 방법이 있습니다.

```java
@Component
public class StringUtil {

  public String getOnlyAlphabet(String str) {
    return str.replaceAll("[^a-zA-Z]", "");
  }
}

@Service
public class service {

  @Autowired
  private StringUtil stringUtil;

  public void tmp() {
    System.out.println(stringUtil.getOnlyNumerics("hello 123 world"));
  }
}
```

위에서 살펴본 두 방식은 생긴것도 다르지만 내부적으로는 훨씬 다릅니다.

## 1. 런타임 다형성

첫 번째로 JAVA의 정적 메소드는 컴파일시 해석되기 때문에 애플리케이션을 종료하기 전까지는 재정의할 수 없습니다.
반면 싱글톤은 다른 기본 클래스와 마찬가지로 런타임 내에 재정의될수있습니다.

쉽게 예제코드로 살펴봅시다.

```java
public class SuperUtility {

    public static String echoIt(String data) {
        return "원본";
    }
}

public class SubUtility extends SuperUtility {

    public static String echoIt(String data) {
        return data;
    }
}
```

위와 같이 "원본"을 출력하는 SuperUtility.echoIt과 이를 상속받아 재정의하는 SubUtility.echoIt이 있습니다.
그렇다면 `SuperUtility.echoIt("입력")`의 출력값은 뭐가 될까요?

정답은 "원본"이 됩니다. 반대로 `SubUtility.echoIt("입력")`의 출력값은 "입력"이 됩니다.

이와는 대조적으로 싱글톤 클래스는 런타임 다형성을 가진다고 하였습니다.
이것도 예제코드로 살펴봅시다.

```java
public class MyLock {

    protected String takeLock(int locks) {
        return "Taken Specific Lock";
    }
}

public class SingletonLock extends MyLock {

    // private constructor and getInstance method

    @Override
    public String takeLock(int locks) {
        return "Taken Singleton Lock";
    }
}

@Test
public void whenSingletonDerivesBaseClass_thenRuntimePolymorphism() {
    MyLock myLock = new MyLock();
    Assert.assertEquals("Taken Specific Lock", myLock.takeLock(10));
    myLock = SingletonLock.getInstance();
    Assert.assertEquals("Taken Singleton Lock", myLock.takeLock(10));
}
```

위 테스트코드는 모두 통과합니다. 런타임 내에서 해당 메소드 내용이 변경된 것이죠.

## 2. Method Parameters

static 클래스 역시 본질적으로는 객체이기 때문에 싱글톤 패턴을 사용할 수 있습니다.
하지만 이는 좋지 않은 생각입니다.

## 3. Object State, Serialization, and Cloneability

싱글톤 클래스는 인스턴스 변수를 가질 수 있으며 다른 객체와 마찬가지로 해당 변수 상태를 유지할 수 있습니다.
또한 싱글톤은 직렬화(serialize)가 가능하다.
인스턴스가 존재하는 경우 객체 복제 메소드를 사용하여 이를 복제할 수 있습니다.

하지만 static 클래스의 경우 클래스 변수와 static 메소드만 가지므로 특정 상태를 가지지 않습니다.
직렬화할 수 없으며 복제할 객체가 없기 때문에 이를 복제하는 것은 의미가 없습니다.

## 4. Loading Mechanism and Memory Allocation

싱글톤은 다른 인스턴스 클래스들과 마찬가지로 힙 영역에 저장됩니다.
이를 통해 애플리케이션에서 필요할 때 마다 싱글톤 객체를 불러올 수 있다는 장점을 가집니다.

하지만 static 클래스는 컴파일 시 스택에 할당됩니다.
따라서 이를 JVM에서 접근하기 위해선 항상 불러와야 하는 단점이 있습니다.

## 5. Efficiency and Performance

앞서 말했듯이 static 클래스는 초기화가 필요하지 않습니다.
이 덕분에 컴파일 시 정적 바인딩을 통해 싱글톤보다 효율적이고 더 빠른 경향이 있습니다.

따라서 우리는 싱글톤을 사용해야 하는 이유를 성능과 효율성보다는 설계적 측면에서 찾아야 합니다.

## 6. Other Minor Differences

물론 static 클래스 없이 싱글톤으로만 구성하는 것도 리팩토링 측면에서 이점이 있을 수 있습니다.

의심할 여지 없이 싱글톤은 클래스 객체 입니다. 그러므로 우리는 쉽게 다중 인스턴스 패턴으로 변경할 수 있습니다.

static 메소드는 객체 없이 클래스 이름으로 호출되므로 다중 인스턴스 환경으로 마이그레이션하는 것이 상대적으로 큰 어려움을 초래하도록 만들 수 있습니다.

추가적으로 static 메소드는 객체보다 클래스와 더 밀접하다고 볼 수 있기 때문에 단위 테스트 시 mocking 하거나 dummy나 stub으로 덮어씌우는게 어려워진다는 문제도 있습니다.


# Making the Right Choice

Go for a singleton if we:

- Require a complete object-oriented solution for the application
- Need only one instance of a class at all given times and to maintain a state
- Want a lazily loaded solution for a class so that it's loaded only when required

Use static classes when we:

- Just need to store many static utility methods that only operate on input parameters and do not modify any internal state
- Don't need runtime polymorphism or an object-oriented solution

# Conclusion

In this article, we reviewed some of the essential differences between static classes and the Singleton pattern in Java. We also inferred when to use either of the two approaches in developing software.



# References

[https://www.baeldung.com/java-static-class-vs-singleton](https://www.baeldung.com/java-static-class-vs-singleton)\
[https://enterkey.tistory.com/300](https://enterkey.tistory.com/300)\
[https://www.geeksforgeeks.org/difference-between-singleton-pattern-and-static-class-in-java](https://www.geeksforgeeks.org/difference-between-singleton-pattern-and-static-class-in-java)\
[https://okky.kr/article/291799](https://okky.kr/article/291799)\
[http://kwon37xi.egloos.com/4844149](http://kwon37xi.egloos.com/4844149)

---


스프링 프로젝트를 진행하면서 공통으로 사용할 클래스(예를 들면 util 함수)를 추가하고자 할 때 @component 어노테이션을 붙혀 빈으로 등록하거나 static 클래스로 선언하는 방법이 있다.
이 두개의 차이가 궁금하기도 하고 언제 무엇을 쓰는것이 유리한지 궁금했는데 이를 잘 정리한 글이 있어 번역하여 공유합니다.

- @Component 어노테이션을 통해 생성한 객체는 빈으로 등록되어 IoC 컨테이너에서 관리됩니다.
- 싱글톤 패턴이란 애플리케이션이 시작될 당시 최초 한번만 메모리를 할당하고 그 메모리에 인스턴스를 만들어 사용하는 디자인 패턴을 말한다. [자세히](https://tecoble.techcourse.co.kr/post/2020-11-07-singleton)
- 아래 본문에서는 @Component 어노테이션을 통해 생성한 객체를 싱글톤 클래스라고 칭합니다.

# 서문

스프링에서는 애플리케이션의 중추가 되며 IoC 컨테이너에 의해 관리되는 객체를 빈이라고 합니다.
싱글톤 클래스와 static 클래스의 차이는 싱글톤 클래스가 서버와 클라이언트 간에 인스턴스화 된 경우 생깁니다.

[//]: # (싱글톤 클래스의 경우 서버와 클라이언트 간에 인스턴스화되는 경우 값을 가집니다.)

이 방법에 의하면 스프링 컨테이너에 의해 빈이 생성된 후 static 영역에 넘기는 것입니다.



대부분의 스프링 부트 애플리케이션들은 스프링 설정이 필요하지 않게 되었으며
보다 간편하게 자바 애플리케이션을 생성해 `java jar`를 통해 실행할 수 있습니다.

싱글톤 클래스는 상속받은 인터페이스를 구현하거나 다른 클래스에게 상속해줄 수 있습니다.
하지만 정적 클래스는 인스턴스 구성원을 상속할 수 없습니다.


# 원문

[When To Choose Static Utility And When Spring Component
](https://www.adoclib.com/blog/when-to-choose-static-utility-and-when-spring-component.html)

https://stackoverflow.com/questions/13746080/spring-or-not-spring-should-we-create-a-component-on-a-class-with-static-meth
https://okky.kr/article/291799
http://kwon37xi.egloos.com/4844149
