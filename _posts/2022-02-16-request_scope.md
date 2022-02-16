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
        //전 세계적으로 유니크한 id 하나 생성 >> 절대 겹치지 않는다.
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

