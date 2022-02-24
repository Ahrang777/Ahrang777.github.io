---
layout: single
title:  "빈 생명주기 콜백"
categories: Spring
tag: [bean, lifecycle]
toc: true
author_profile: false
sidebar:
    nav: "docs"
---



# 빈 생명주기 콜백

데이터베이스 커넥션 풀이나, 네트워크 소켓처럼 애플리케이션 시작 시점에 필요한 연결을 미리 해두고,  애플리케이션 종료 시점에 연결을 모두 종료하는 작업을 진행하려면, 객체의 초기화와 종료 작업이 필요하다. 스프링 빈의 생성, 의존관계 주입이 끝나야 필요한 데이터를 사용할 수 있는 상태가 된다. 때문에 객체의 초기화 작업은 의존관계 주입까지 끝난 후에 호출되어야 하고 종료작업은 객체가 소멸되기 직전에 해준다. 하지만 여기서 개발자가 스프링 빈의 의존관계 주입이 끝났는지 안 끝났는지 어떤식으로 알 수 있을지 의문이 생긴다. 



## 스프링 빈의 이벤트 생명주기

**스프링 컨테이너 생성 → 스프링 빈 생성 → 의존관계 주입 → 초기화 콜백 → 사용 → 소멸전 콜백 → 스프링 종료**

스프링 빈은 scope 에 따라서 생명주기가 다르겠지만 일반적으로 가장 많이 사용되는 싱글톤 빈으로 설명하겠다. 싱글톤 빈의 경우 스프링 컨테이너가 생성되고 빈이 생성된다. 생성자 주입일 경우 생성과 동시에 주입이 이루어지고 그 외 주입 방법일 경우 빈이 생성 된 후, 의존관계가 주입이 된다. **스프링은 의존관계 주입이 완료되면 스프링 빈에게 콜백 메서드를 통해 초기화 시점을 알려주는 다양한 기능을 제공한다. 또한 스프링 컨테이너가 종료되기 직전에 소멸 콜백을 준다.** 



## 빈 생명주기 콜백

스프링은 크게 3가지 방법으로 빈 생명주기 콜백을 지원한다.

1. 인터페이스 (InitializingBean, DisposableBean)
2. 설정 정보에 초기화 메서드, 종료 메서드 지정
3. @PostConstruct, @PreDestroy 애노테이션 지원



### 인터페이스 (InitializingBean, DisposableBean)

```java
import org.springframework.beans.factory.DisposableBean;
import org.springframework.beans.factory.InitializingBean;

public class NetworkClient implements InitializingBean, DisposableBean {
    
    private String url;
    
    public NetworkClient(){
        System.out.println("생성자 호출, url = " + url);
    }
    
    public void setUrl(String url){
        this.url = url;
    }
    
    // 서비스 시작시 호출
    public void connect(){
        System.out.println("connect: " + url)
    }
    
    public void call(String message){
        System.out.println("call: " + url + " message = " + message);
    }
    
    // 서비스 종료시 호출
    public void disconnect(){
        System.out.println("close + " + url);
    }
    
    @Override
    public void afterPropertiesSet() throws Exception {
        connect();
        call("초기화 연결 메시지");
    }
    
    @Override
	public void destroy() throws Exception {
     	disconnect();   
    }
}
```

외부 네트워크에 미리 연결하는 객체를 생성한다고 가정해보자. 예시는 단순히 문자만 출력한다.

- InitializingBean 은 afterPropertiesSet() 메서드로 초기화를 지원한다. 
- DisposableBean 은 destroy() 메서드로 소멸을 지원한다.



**InitializingBean, DisposableBean 인터페이스**

InitializingBean, DisposableBean 인터페이스를 이용하면 초기화, 소멸 메서드 이름을 변경할 수 없고 내가 수정할 수 없는 외부 라이브러리에 적용할 수 없다. 또한 InitializingBean, DisposableBean 인터페이스는 스프링 전용 인터페이스라서 이를 이용하면 스프링 전용 인터페이스에 의존한다는 단점이 있다. 여기서 스프링 전용 인터페이스에 의존하는 것은 스프링이 아닌 다른 컨테이너에서는 이용할 수 없기때문에 단점이다. 요즘에는 이 방법은 거의 사용하지 않고 3가지 방법중 나머지 두 방법을 혼용한다. 



### 설정 정보에 초기화 메서드, 종료 메서드 지정

설정 정보에 @Bean(initMethod = "init", destroyMethod = "close") 로 초기화, 소멸 메서드를 지정할 수 있다. 

```java
@Override
public void afterPropertiesSet() throws Exception {
	connect();
    call("초기화 연결 메시지");
}
    
@Override
public void destroy() throws Exception {
	disconnect();   
}
```

앞에서 설명했던 예시에서 위 내용을 아래 내용으로 수정하고 설정 정보에서 @Bean(initMethod = "init", destroyMethod = "close")를 통해 스프링 빈 등록을 하며 초기화, 종료 메서드를 지정한다. 

```java

public void init(){
	connect();
	call("초기화 연결 메시지");
}


public void close(){
	disconnect();
}
```



@Bean 의 destroyMethod 는 기본값이 (inferred) (추론) 로 등록되어있다. 라이브러리는 대부분 close, shutdown 이라는 이름의 종료 메서드를 사용하는데 이 추론 기능을 이용하면 close, shutdown 이라는 이름의 메서드를 종료 메서드로 추론하여 호출한다. 



설정 정보에 초기화 메서드, 종료 메서드를 지정하는 방식을 사용하면 메서드 이름을 자유롭게 줄 수 있고 스프링 빈의 코드가 스프링 의존적이지 않다. 또한 설정 정보를 이용하기 때문에  수정할 수 없는 외부 라이브러리에도 초기화, 종료 메서드를 적용할 수 있다. 



### 애노테이션 @PostConstruct, @PreDestroy

설정 정보에 초기화 메서드, 종료 메서드를 지정하는 방식의 예시에서 아래와 같이 코드를 수정해도 결과는 동일하다. 

```java
import javax.annotation.PostConstruct;
import javax.annotation.PreDestroy;

@PostConstruct
public void init(){
	connect();
	call("초기화 연결 메시지");
}

@PreDestroy
public void close(){
	disconnect();
}
```

이 방식은 최신 스프링이 가장 권장하는 방법으로 애노테이션만 붙이면 되서 매우 간편하다. 또한 스프링 종속적인 기술이 아니라 자바 표준이라서 스프링이 아닌 다른 컨테이너에서도 사용 가능하다. 하지만 수정할 수 없는 외부 라이브러리에는 적용할 수 없기때문에 이 경우 앞에서 설명한 설정 정보에 초기화 메서드, 종료 메서드를 지정하는 방식을 이용한다. 



참고: [스프링 핵심 원리 - 기본편 - 인프런 | 강의 (inflearn.com)](https://www.inflearn.com/course/스프링-핵심-원리-기본편)







