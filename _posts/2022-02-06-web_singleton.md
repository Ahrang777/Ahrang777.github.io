---
layout: single
title:  "스프링, 싱글톤 컨테이너"
categories: Spring
tag: singleton
toc: true
author_profile: false
sidebar:
    nav: "docs"
---



# 스프링과 싱글톤

스프링 애플리케이션은 대부분 웹 애플리케이션이다. 웹 애플리케이션의 경우 보통 다수의 고객이 동시에 요청을 한다. 각 고객이 모두 같은 요청을 해서 동일 객체가 요구된다고 가정해보자. 이 경우 매번 객체를 생성한다면 메모리 낭비가 심해질 것이다. 이때 필요한 것이 싱글톤이다. 



## 싱글톤 패턴

싱글톤 패턴은 클래스의 인스턴스가 1개만 생성되는 것을 보장하는 디자인 패턴이다. 

```java
public class SingletonService{
    private static final SingletonService instance = new SingletonService();
    
    private SingletonService(){
        
    }
    
    public static SingletonService getInstance(){
        return instance;
    }
}
```

위 코드는 간단하게 싱글톤 패턴을 적용한 예시이다. 클래스 내부에서 이미 객체를 만들어서 static 영역에 올렸고 static 메서드인 getInstance()를 이용하여 외부에서 이 객체를 받을 수 있다. 그리고 외부에서 임의로 new 키워드로 객체를 생성하는 것을 막기위해 생성자의 접근 지정자를 private으로 했다.



### 싱글톤 패턴 문제점

싱글톤 패턴은 매번 새로운 객체를 생성하지 않고 이미 만들어진 객체를 공유하여 효율적으로 사용할 수 있다는 점에서 매력적이지만 안티패턴으로 불리기도 한다.

싱글톤을 이용하는 경우 대부분 인터페이스가 아닌 구현 클래스의 객체를 미리 생성해 놓고 static 메서드를 이용하여 사용하게 된다. 때문에 **SOLID 원칙을 위반**할 수 있는 가능성을 열어둠과 동시에, 싱글톤을 사용하는 곳과 싱글톤 클래스 사이에 **의존성**이 생기게 된다. 클래스 사이에 강한 의존성, 즉 높은 결합이 생기면 수정, 단위테스트의 어려움 등 다양한 문제가 발생한다 



1.객체 지향과 맞지 않다.

싱글톤은 전역상태를 만들 수 있기 때문에 바람직 하지 못하다. 아무 객체나 자유롭게 접근하고 수정하고 공유할 수 있는 전역 상태를 갖는 것은 객체지향 프로그래밍에서는 지양되어야 할 모델이다. 

싱글톤은 자신만이 객체를 생성할 수 있도록 생성자를 private으로 제한한다. 하지만 상속을 통해 다형성을 적용하기 위해서는 다른 기본생성자가 필요하므로 상속할 수 없다. 또한 싱글톤 구현을 위해 객체 지향적이지 못한 static 필드, 메서드를 사용해야 한다. 



2.테스트하기 어렵다, 유연성이 떨어진다

싱글톤은 만들어지는 방식이 제한적이기 때문에 테스트에서 사용될 때 mock 오브젝트 등으로 대체하기가 힘들다. 



3.서버 환경에서는 싱글톤이 1개만 생성됨을 보장하지 못한다. 

서버에서 클래스 로더를 어떻게 구성하느냐에 따라 싱글톤 클래스임에도 불구하고 1개 이상의 객체가 만들어질 수 있다. 따라서 Java 언어를 이용한 싱글톤 기법은 서버 환경에서 싱글톤이 꼭 보장된다고 볼 수 없다. 또한 여러 개의 JVM에 분산돼서 설치되는 경우에도 독립적으로 객체가 생성된다.



4.싱글톤 패턴 적용을 위한 코드 자체가 많이 들어간다. 

예를 들면 스프링이 없는 상황에서 스프링 빈처럼 싱글톤으로 관리하기 위해 그 많은 클래스들에 싱글톤을 적용한다고 생각해보면 체감이 될것이다. 



### 싱글톤 방식의 주의점

싱글톤 패턴이든, 스프링 같은 싱글톤 컨테이너를 사용하든 객체 인스턴스를 하나만 사용해서 공유하는 싱글톤 방식은 여러 클라이언트에서 공유하기 때문에 상태를 유지(stateful)하게 설계하면  안된다. 무상태(stateless)하게 설계해야한다. 

- 특정 클라이언트에 의존적인 필드가 있으면 안된다. 
- 특정 클라이언트가 값을 변경할 수 있는 필드가 있으면 안된다.
- 가급적 읽기만 가능해야 한다. 
- 필드 대신에 자바에서 공유되지 않는, 지역변수, 파라미터, ThreadLocal 등을 사용해야 한다



## 스프링 컨테이너, 싱글톤 컨테이너

스프링 컨테이너는 앞서 설명한 싱글톤 패턴의 문제점을 해결하면서, 객체 인스턴스를 싱글톤으로 관리한다. 지금까지 우리가 별다른 설정없이 생성했던 스프링 빈이 싱글톤으로 관리되는 빈이다. 

스프링 컨테이너가 스프링 빈을 싱글톤으로 관리하기 대문에 고객의 요청이 올때마다 객체를 생성하지 않고 하나의 객체만 생성하고 공유하는 방식으로 효율적으로 재상용 할 수 있다. 



## @Configuration, 싱글톤

```java
@Configuration
public class AppConfig{
    @Bean
    public MemberService memberService(){
        System.out.println("memberService()");
        return new MemberServiceImpl(memberRepository());
    }
    
    @Bean
    public MemberRepository memberRepository(){
        System.out.println("memberRepository()");
        return new MemoryMemberRepository();
    }
}

```

위와 같은 어노테이션 기반 자바코드 설정을 봤을때 이상한 점을 발견 할 수 있다. 스프링 빈이 싱글톤으로 관리된다고 했는데 @Bean이 붙은 메서드를 보면 memberService()의 return new MemberServiceImpl(memberRepository()); 에서 memberRepository() 를 통해 new MemoryMemberRepository() 로 1번, 그 아래 @Bean 이 붙은 memberRepository() 자체로 2번 총 2번 memberRepository 스프링 빈이 생성된 것처럼 보인다. 하지만 실제로 확인을 해보면 싱글톤으로 memberRepository 스프링 빈은 1개임을 알 수 있다. 어떻게 이런일이 가능할까?

@Configuration 덕분에 가능한 일이다. 스프링 컨테이너는 스프링 빈이 싱글톤이 되도록 보장해줘야 하는데 GCLIB라는 바이트 코드를 조작하는 라이브러리를 통해 이를 해결한다. 위 코드를 예로 들면 GCLIB 를 통해 AppConfig를 상속받는 임의의 다른 클래스를 만들고 이를 스프링 빈으로 등록한다. 이 임의의 다른 클래스가 싱글톤을 보장한다. 때문에 @Configuration이 없다면 처음 예상한대로 memberRepository 가 싱글톤으로 생성되지 않는다. 







reference

[참고1](https://mangkyu.tistory.com/153)

[참고2](https://www.inflearn.com/course/스프링-핵심-원리-기본편)
