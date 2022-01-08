---
layout: single
title:  "github blog에서 commit기록이 녹색으로 안 남는 문제해결"
categories: Spring
tag: [AOP]
toc: true
author_profile: false
sidebar:
    nav: "docs"
---

# Spring MVC

<serlvet> ⓙ <serlvet> 전체
<servlet-name> 서블릿 닉 네임 </servlet-name> ⓚ
<serlvet-class> 서블릿 클래스 풀네임(패키지 명까지) </servlet-class> 
<init-param>	m
<param-name> 매개변수명 </param-name>
<param-value> 값 </param-value>
</init-param>
<load-on-startup> 실행 순서 값(0값은 서버임의실행) </load-on-startup> ⓝ
</servlet>


j 서블릿을 컨테이너에 등록
k 서블릿 닉네임 등록, 서블릿 이름이 너무 길 때 의상 사용, 클래스 이름이 짧으면 클래스 이름과 동일하게 등록해도 괜찮다
m 서블릿 실행될 때 초기값으로 전달될 매개변수 
n 웹 서버가 구동될 때 서블릿의 init() 메서드를 미리 실행할지 지정하고 있다. 각 서블릿의 생성/초기화 순서를 의미한다. (값이 작은 것이 먼저 실행)





ContextLoaderListener 와 DispatcherServlet 은 각각 WebApplicationContext 인스턴스를 생성
ContextLoaderListener 가 생성한 컨텍스트가 root 컨텍스트가 되고, DispatcherServlet 이 생성한 
인스턴스는 root 컨텍스트를 부모로 사용하는 자식 컨텍스트가 된다. 

자식 컨텍스트들은 root 컨텍스트가 제공하는 bean을 사용할 수 있어서 ContextLoaderListener 를 
이용하여 공통빈을 설정하는 것이다.

ContextLoaderListener 는 ContextConfigLocation 을 명시하지 않으면 /WEB-INF/applicationContext.xml 을 
설정파일로 사용한다.



Model addAttribute(Object value)
- value를 추가한다. value의 패키지 이름을 제외한 단순 클래스 이름을 모델 이름으로 사용한다. 이 때 첫 글자는 소문자로 처리한다.
- value가 배열이거나 컬렉션인 경우 첫 번째 원소의 클래스 이름 뒤에 "List"를 붙인 걸 모델 이름으로 사용한다. 이 경우에도 클래스 이름의 첫자는 소문자로 처리한다.
