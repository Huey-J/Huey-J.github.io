---
title: static 메소드 vs @component 메소드
author: Huey-J
date: 2022-06-23 12:00:00 +0800
categories: [backend, spring]
tags: [spring, java]
---

이번 포스트에서는 싱글톤 클래스와 static 클래스의 차이점과 보다 더 올바르게 사용하려면 어떻게 해야 하는지 알아보겠습니다.

예시로 상황을 그려봅시다. 스프링 프로젝트를 진행하면서 공통으로 사용할 클래스(예를 들면 util 함수)를 추가하고자 합니다.
이 때 우리는 아래와 같이 간단하게 static 클래스를 통해 해결할 수 있습니다.

```java
public class StringUtil {

  public static String getOnlyNumerics(String str) {
    return str.replaceAll("[^0-9]", "");
  }

}
```

그리고 위에서 선언한 메소드를 사용하는 방법은 다음과 같습니다.
```java
import 경로.StringUtil;

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
```

```java
import 경로.StringUtil;

@Service
public class service {

  @Autowired
  private StringUtil stringUtil;

  public void tmp() {
    System.out.println(stringUtil.getOnlyNumerics("hello 123 world"));
  }

}
```



https://www.baeldung.com/java-static-class-vs-singleton\
https://enterkey.tistory.com/300

이 간단한 튜토리얼에서는 Singleton 디자인 패턴에 프로그래밍하는 것과 Java에서 정적 클래스를 사용하는 것의 몇 가지 현저한 차이점에 대해 논의합니다.
우리는 두 코딩 방법론을 검토하고 프로그래밍의 다른 측면과 관련하여 그것들을 비교할 것입니다.

Singleton은 응용프로그램 수명 동안 클래스의 단일 인스턴스를 보장하는 설계 패턴입니다.
또한 해당 인스턴스에 대한 글로벌 액세스 지점을 제공합니다.

static – reserved 키워드는 인스턴스 변수를 클래스 변수로 만드는 수식어입니다.
따라서 이러한 변수는 클래스와 연관됩니다(모든 개체와).
메소드와 함께 사용하면 클래스 이름만으로 메소드에 액세스할 수 있습니다.
마지막으로 정적 중첩된 내부 클래스도 만들 수 있습니다.
이 컨텍스트에서 정적 클래스에는 정적 메서드와 정적 변수가 포함됩니다.


3. 싱글톤 대 정적 유틸리티 클래스

두 클래스의 몇 가지 중요한 차이점을 알아보자. (객체 지향적으로)



---
자바 공식문서로 변경 (위)
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




# memo

singleton = @component?



# 원문

[When To Choose Static Utility And When Spring Component
](https://www.adoclib.com/blog/when-to-choose-static-utility-and-when-spring-component.html)

https://stackoverflow.com/questions/13746080/spring-or-not-spring-should-we-create-a-component-on-a-class-with-static-meth
https://okky.kr/article/291799
http://kwon37xi.egloos.com/4844149
