---
title: static 메소드 vs @component 메소드 (feat.싱글톤 패턴)
author: Huey-J
date: 2022-07-08 12:00:00 +0800
categories: [backend, spring]
tags: [spring, java]
---

# 들어가며

스프링 프로젝트를 하면서 공통 클래스(예를 들면 유틸 클래스)를 만들 때 보통 static 방식과 @Component방식을 자주 사용합니다.

이번 포스트에서는 이 두 방식의 차이점과 장단점을 살펴보고 각각 어느 상황에 써야 더 효율적이고 스프링다운지 알아보도록 합시다.

# 예시 상황

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


# Static? Singleton?

> 싱글톤 패턴(Singleton Pattern)이란 인스턴스가 오직 하나만 생성되는 것을 보장하고 어디에서든 이 인스턴스에 접근할 수 있도록 하는 디자인 패턴 이다.
> *(출처: [해시넷 위키](http://wiki.hash.kr/index.php/싱글톤패턴))*
{: .prompt-info }

## Singleton

**@Component 어노테이션을 통해 생성한 객체는 빈으로 등록되어 IoC 컨테이너에서 관리됩니다.**
그리고 스프링 컨테이너는 등록된 스프링 빈들을 모두 **싱글톤**으로 관리합니다.

즉 @Component 어노테이션을 붙힌 클래스는 스프링의 싱글톤 패턴을 통해 관리된다는 의미가 됩니다.

따라서 어디서든 접근 가능한 IoC 컨테이너에 저장이 되며 애플리케이션이 시작될 당시 최초로 한 번만 메모리에 할당됩니다.

## Static

static은 인스턴스 변수를 클래스 변수로 만들어 줍니다.
이를 통해 `클래스명.메소드명`을 통해 해당 메소드에 접근할 수 있게 됩니다. 마치 `클래스명.변수명` 처럼 말이죠.

**static 키워드를 통해 생성된 정적 멤버들은 Heap영역이 아닌 Static 영역에 할당됩니다.**

해당 영역에 할당된 메모리는 모든 객체가 공유하여 하나의 멤버를 어디서든지 참조할 수 있는 장점을 가지지만
Garbage Collector의 관리 영역 밖에 존재하기에 static영역에 있는 멤버들은 프로그램의 종료시까지 메모리가 할당된 채로 존재하게 됩니다.

그렇기에 static을 너무 남발하게 되면 만들고자 하는 시스템 성능에 악영향을 줄 수 있습니다.

> static에 대해 좀 더 이해가 필요하면 [여기](https://velog.io/@yulhee741/java-static-변수)에 간단히 정리가 되어있으니 참고해주세요~


# 차이점

> 아래에서는 편의상 @Component 어노테이션 등으로 스프링 컨테이너에 등록한 빈(POJO)을 싱글톤이라고 명명합니다.
> 실제로는 이 둘의 정의가 조금 다르니 꼭 참고하시길 바랍니다.


## 1. 저장 방식

싱글톤은 다른 인스턴스 클래스들과 마찬가지로 힙 영역에 저장됩니다.
이를 통해 애플리케이션에서 필요할 때 마다 싱글톤 객체를 불러올 수 있다는 장점을 가집니다.

하지만 static 클래스는 위에서 언급했듯 컴파일 시 스택에 할당됩니다.
따라서 이를 JVM에서 접근하기 위해선 항상 새로 불러와야 하는 단점이 있습니다.

## 2. 런타임 다형성

static 키워드를 통해 정의한 정적 메소드는 컴파일시 해석되기 때문에 애플리케이션을 종료하기 전까지는 재정의할 수 없습니다.

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

위 테스트코드는 모두 통과합니다. 런타임 안에서 해당 메소드 내용이 변경된 것이죠.

즉 싱글톤은 런타임 다형성을 가진다고 볼 수 있습니다.

## 기타 차이점

- **변수 상태**
  - 싱글톤 클래스: 인스턴스 변수이므로 다른 객체와 마찬가지로 해당 변수 상태를 유지할 수 있음
  - static 클래스: 클래스 변수와 static 메소드만 가지므로 특정 상태를 가지지 않음
- **직렬화**
  - 싱글톤 클래스: 직렬화 가능
  - static 클래스: 직렬화 불가능
- **효율성**
  - 앞서 말했듯 static 클래스는 초기화가 필요하지 않습니다. 이 덕분에 컴파일 시 정적 바인딩을 통해 싱글톤보다 효율적이고 더 빠른 경향이 있습니다.
  - 하지만 static 메소드는 객체보다 클래스와 더 밀접하다고 볼 수 있기 때문에 단위 테스트 시 mocking 하거나 dummy나 stub으로 덮어씌우는게 어려워진다는 문제가 있습니다.
  - 싱글톤의 경우 느리더라도 필요할 때만 불러와 상태를 유지하며 사용할 수 있습니다.


# Conclusion

[여기](http://kwon37xi.egloos.com/4844149)에서 권남님이 정의해주신 말이 있다.

> static 함수 모음 클래스의 모든 함수는 인자가 동일할 경우 항상 동일한 결과를 리턴해야 한다. 이 규칙을 지킬 수 없으면 POJO Bean으로 만들라.
{: .prompt-tip }

즉 static은 외부 자원(예를 들면 데이터 베이스)에 하나도 의존하면 안된다는 것이다.

그렇다면 가장 맨 처음 봤던 `StringUtil` 클래스는 static일까? 싱글톤일까?

```java
public class StringUtil {

  public String getOnlyNumerics(String str) {
    return str.replaceAll("[^0-9]", "");
  }
}
```

간단하게 위에서 언급한 정의대로 생각해보자.

> Q. `StringUtil` 내에 있는 메소드는 항상 동일한 결과를 반환하는가?\
> A. Yes.

따라서 해당 클래스는 static 클래스로 정의하는 것이 올바르다.

다른 예시를 들어보자.

```java
public class Util {

  @Autowired
  private UserRepository userRepository;

  public boolean isEmailDuplicated(String email) {
    Optional user = userRepository.findByEmail(email).orElse(null);
    return (user == null);
  }
}
```

해당 클래스의 `isEmailDuplicated` 데이터베이스에서 해당 메일로 가입된 유저가 있는지 확인하는 메소드이다.

너무 쉬운 예시인 듯 하지만 해당 메소드는 외부 자원인 데이터베이스에 저장된 값에 따라 결과값이 달라지므로 `@Component` 어노테이션을 통해 선언하는게 올바르다.


# References

[https://www.baeldung.com/java-static-class-vs-singleton](https://www.baeldung.com/java-static-class-vs-singleton)\
[https://enterkey.tistory.com/300](https://enterkey.tistory.com/300)\
[https://www.geeksforgeeks.org/difference-between-singleton-pattern-and-static-class-in-java](https://www.geeksforgeeks.org/difference-between-singleton-pattern-and-static-class-in-java)\
[https://okky.kr/article/291799](https://okky.kr/article/291799)\
[http://kwon37xi.egloos.com/4844149](http://kwon37xi.egloos.com/4844149)\
[https://stackoverflow.com/questions/13746080/spring-or-not-spring-should-we-create-a-component-on-a-class-with-static-meth](https://stackoverflow.com/questions/13746080/spring-or-not-spring-should-we-create-a-component-on-a-class-with-static-meth)
