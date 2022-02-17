---
layout: single
title:  "reqest 스코프"
categories: Spring
tag: [request, scope]
toc: true
author_profile: false
sidebar:
    nav: "docs"
---



# 웹 스코프

웹 스코프는 웹 환경에서만 동작하고 스프링이 해당 스코프의 종료시점까지 관리한다. 

**웹 스코프 종류**

- request: HTTP 요청 하나가 들어오고 나갈 때까지 유지되는 스코프, 각각의 HTTP 요청마다 별도의 빈 인스턴스가 생성되고, 관리된다.
- session: HTTP Session과 동일한 생명주기를 가지는 스코프
- application: 서블릿 컨텍스트( ServletContext )와 동일한 생명주기를 가지는 스코프
- websocket: 웹 소켓과 동일한 생명주기를 가지는 스코프



```java
@Component
@Scope(value="request")
public class MyLogger {

    private String uuid;
    private String requestURL;

    public void setRequestURL(String requestURL) {
        this.requestURL = requestURL;
    }

    public void log(String message){
        System.out.println("[" + uuid + "]" + "[" + requestURL + "] " + message);
    }

    @PostConstruct
    public void init(){
        //전 세계적으로 유니크한 id 하나 생성, 절대 겹치지 않는다.
        uuid = UUID.randomUUID().toString();
        System.out.println("[" + uuid + "] request scope bean create: " + this);
    }

    @PreDestroy
    public void close(){
        System.out.println("[" + uuid + "] request scope bean close: " + this);
    }
}



@Controller
@RequiredArgsConstructor
public class LogDemoController {

    private final LogDemoService logDemoService;
    private final MyLogger myLogger;

    @RequestMapping("log-demo")
    @ResponseBody
    public String logDemo(HttpServletRequest request) {
        String requestURL = request.getRequestURL().toString();
		myLogger.setRequestURL(requestURL);
        
        myLogger.log("controller test");        
        logDemoService.logic("testId");
        return "OK";
    }
}


@Service
@RequiredArgsConstructor
public class LogDemoService{
    private final MyLogger myLogger;
    
	public void logic(String id){
        myLogger.log("service id = " + id);
    }
}
```

로그 출력을 위한 MyLogger 클래스는 @Scope(value="request") 로 request 스코프로 지정하였다. 이 빈은 HTTP 요청 당 하나씩 생성되고 HTTP 요청이 끝나는 시점에 소멸된다.

Controller, Service는 싱글톤 빈이고 MyLogger 를 생성자 주입으로 주입받는다. 하지만 주입할 때는 아직 HTTP 요청이 오지 않아서 request 스코프의 빈이 생성되지 않았다. 때문에 오류가 발생한다. 



## 해결책

**1.Provider**

spring에서 지원하는 ObjectProvider를 사용한다.

```java
@Controller
@RequiredArgsConstructor
public class LogDemoController{
    private final LogDemoService logDemoService;
    private final ObjectProvider<MyLogger> myLoggerProvider;
    
    @RequestMapping("log-demo")
    @ResponseBody
     public String logDemo(HttpServletRequest request) {
         String requestURL = request.getRequestURL().toString();
         MyLogger myLogger = myLoggerProvider.getObject();
         myLogger.setRequestURL(requestURL);
         myLogger.log("controller test");
         logDemoService.logic("testId");
         return "OK";
    }
}



@Service
@RequiredArgsConstructor
public class LogDemoService{
    private final ObjectProvider<MyLogger> myLoggerProvider;
        
	public void logic(String id){
        MyLogger myLogger = myLoggerProvider.getObject();
        myLogger.log("service id = " + id);
    }
}
```

ObjectProvider 를 통해서 myLoggerProvider.getObject() 까지 request scope 빈의 생성을 지연할 수 있다. myLoggerProvider.getObject() 를 호출하는 시점에는 HTTP 요청이 진행중이므로 request scope 빈의 생성이 정상 처리된다. 



**2.프록시 방식**

```java
@Component
@Scope(value="request", proxyMode = ScopedProxyMode.TARGET_CLASS)
public class MyLogger{
    
}


@Controller
@RequiredArgsConstructor
public class LogDemoController {

    private final LogDemoService logDemoService;
    private final MyLogger myLogger;

    @RequestMapping("log-demo")
    @ResponseBody
    public String logDemo(HttpServletRequest request) {
        String requestURL = request.getRequestURL().toString();
		myLogger.setRequestURL(requestURL);
        
        myLogger.log("controller test");        
        logDemoService.logic("testId");
        return "OK";
    }
}


@Service
@RequiredArgsConstructor
public class LogDemoService{
    private final MyLogger myLogger;
    
	public void logic(String id){
        myLogger.log("service id = " + id);
    }
}
```

@Scope(value="request", proxyMode = ScopedProxyMode.TARGET_CLASS) 를 설정하면 CGLIB 라는 바이트 코드 조작 라이브러리를 사용해서 MyLogger를 상속받은 가짜 프록시 객체를 생성한다. 실제로 확인해보면 우리가 등록한 MyLogger 클래스가 아니라 MyLogger$ $EnhancerBySpringCGLIB 라는 클래스로 만들어진 객체가 대신 등록되었음을 알 수 있다. 클라이언트가 myLogger.logic() 을 호출하면 가짜 프록시 객체는 request 스코프의 진짜 myLogger.logic()을 호출한다. 이 가짜 프록시 객체는 MyLogger를 상속받아서 만들어졌기 때문에 클라이언트 입장에서는 원본인지 모르게, 동일하게 사용할 수 있다. 

즉, CGLIB 라이브러리로 MyLogger를 상속받은 가짜 프록시 객체를 만들어서 주입한다. 이 가짜 프록시 객체는 실제 요청이 오면 내부에서 실제 빈을 요청하는 위임 로직이 들어있고 싱글톤 빈처럼 작동한다. 때문에 이 싱글톤 빈처럼 작동하는 프록시 객체는 정상적으로 주입되고 진짜 객체에 대한 이용을 지연할 수 있다. 

