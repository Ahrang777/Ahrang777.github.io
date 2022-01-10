---
layout: single
title:  "Spring MVC"
categories: Spring
tag: [Spring MVC]]
toc: true
author_profile: false
sidebar:
    nav: "docs"
---

# Spring MVC

![springmvc](https://user-images.githubusercontent.com/59478159/148723742-c30f9f7b-d9df-4eb1-bd15-1c142e5fe514.png)

## 개념설명
Dispatcher servlet
- front controller 로 작동
- application 으로 들어오는 모든 request를 가로채서 controller 로 보낸다.
- request를 처리하기위해 호출 할 controller 에 대한 handler mapping을 참조한다.

Handler Mapping
- 특정 request를 처리하는 적절한 controller 찾기
- request URL과 controller 클래스 간의 매핑은 XML 설정, annotation 을 통해 수행된다.

View Resolver
논리적 이름으로 실제 뷰 파일을 찾는다

Controller 
- 사용자 request를 처리하고 적절한 model 을 만들고 렌더링을 위해 뷰로 전달한다.


View
model data 를 렌더링하고 일반적으로 HTML 출력을 생성한다.

Model
application 데이터를 캡슐화하며 일반적으로 POJO로 구성된다.


## 필요한 설정
Maven, Tomcat 사용중이라면 

Maven configuration
- POM.xml

Web deployment descripter 
- Web.xml

Spring MVC Configuration
- root-context.xml
- servlet-context.xml, dao-context.xml, service-context.xml 



### Web.xml

DispatcherServlet

Spring Container (WebApplicationContext) 를 인스턴스화 한다.
WebApplicationContext는 일부 configuration 메타데이터와 함께 제공되어야 한다.

```xml
    <!-- Processes application requests -->
	<servlet>
		<servlet-name>appServlet</servlet-name>
		<servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
		<init-param>
			<param-name>contextConfigLocation</param-name>
			<param-value>/WEB-INF/spring/appServlet/servlet-context.xml</param-value>
		</init-param>
		<load-on-startup>1</load-on-startup>
	</servlet>

	<servlet-mapping>
		<servlet-name>appServlet</servlet-name>
		<url-pattern>/</url-pattern>
	</servlet-mapping>
```

<serlvet> ⓙ <serlvet> 전체
<servlet-name> 서블릿 닉 네임 </servlet-name> ⓚ
<serlvet-class> 서블릿 클래스 풀네임(패키지 명까지) </servlet-class> 
<init-param>  ⓜ
<param-name> 매개변수명 </param-name>
<param-value> 값 </param-value>
</init-param>
<load-on-startup> 실행 순서 값(0값은 서버임의실행) </load-on-startup> ⓝ
</servlet>


ⓙ 서블릿을 컨테이너에 등록
ⓚ 서블릿 닉네임 등록, 서블릿 이름이 너무 길 때 편의상 사용, 클래스 이름이 짧으면 클래스 이름과 동일하게 등록해도 괜찮다
ⓜ 서블릿 실행될 때 초기값으로 전달될 매개변수 
ⓝ 웹 서버가 구동될 때 서블릿의 init() 메서드를 미리 실행할지 지정하고 있다. 각 서블릿의 생성/초기화 순서를 의미한다. (값이 작은 것이 먼저 실행)







ContextLoaderListener
공유되는 bean들을 포함하는 Spring Container 를 인스턴스화 한다.
DispatcherServlet 이 생성한 bean은 ContextLoaderListener 가 생성한 bean을 참조 할 수 있다.

```xml
    <context-param>
		<param-name>contextConfigLocation</param-name>
		<param-value>
			/WEB-INF/spring/root-context.xml
		</param-value>
	</context-param>

	<!-- Creates the Spring Container shared by all Servlets and Filters -->
	<listener>
		<listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
	</listener>
```


![springmvc1](https://user-images.githubusercontent.com/59478159/148723755-cb153939-4739-4a97-936a-519a59196041.png)

추가 설명
ContextLoaderListener 와 DispatcherServlet 은 각각 WebApplicationContext 인스턴스를 생성한다.
ContextLoaderListener 가 생성한 컨텍스트가 root 컨텍스트가 되고, DispatcherServlet 이 생성한 
인스턴스는 root 컨텍스트를 부모로 사용하는 자식 컨텍스트가 된다. 

자식 컨텍스트들은 root 컨텍스트가 제공하는 bean을 사용할 수 있어서 ContextLoaderListener 를 
이용하여 공유되는 bean을 설정하는 것이다.

참고
ContextLoaderListener 는 ContextConfigLocation 을 명시하지 않으면 /WEB-INF/applicationContext.xml 을 
설정파일로 사용한다.


Model 
Controller 에서 View로 object들을 전달하는데 사용된다
named objects(= model attributes) 의 collection이다.

![springmvc2](https://user-images.githubusercontent.com/59478159/148723761-b9bb8c90-273d-4dec-9855-f5f7cc5d124b.png)

Model 구현방법
1) java.util.map 
2) Model interface
3) ModelMap object 

```java
public String getGreeting(Model model) { ... }
```

Model 대신 Map<String, Object> or ModelMap 가능
Spring이 type에 맞는 객체를 만들어서 넣어준다.


1) java.util.map 
key는 반드시 String이 되야하며 각 model attribute의 이름을 우리가 직접 결정 해야한다는 단점이 있다.

2) Model 
java.util.map 의 단점을 해결했다. 아래 메소드를 이용하면 우리가 이름을 결정할 필요가 없다.


Model addAttribute(Object value)
- value를 추가한다. value의 패키지 이름을 제외한 단순 클래스 이름을 모델 이름으로 사용한다. 이 때 첫 글자는 소문자로 처리한다.
- value가 배열이거나 컬렉션인 경우 첫 번째 원소의 클래스 이름 뒤에 "List"를 붙인 걸 모델 이름으로 사용한다. 이 경우에도 클래스 이름의 첫자는 소문자로 처리한다.

Model addAttribute(String key, Object value)
우리가 직접 이름을 결정할 수도 있다.

3) ModelMap
addAttribute() 메소드를 chained call 할 수 있다.

```java
model.addAttribute("key1", value1)
     .addAttribute("key2", value2)
```

@Component annotation이 있는 경우 bean으로 등록한다.
@Component 를 역할에 따라 구체화하면 @Controller, @Service, @Repository 로 이들 또한 bean으로 등록


@RequestMapping
URL과 class 또는 method를 매핑한다.
해당 URL이 오면 매핑된 method로 처리
보통 공통된 URL을 class level 로 매핑하고 세부적으로 method 에 매핑한다.
