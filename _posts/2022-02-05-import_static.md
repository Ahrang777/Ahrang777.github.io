---
layout: single
title:  "import static"
categories: Java
tag: static
toc: true
author_profile: false
sidebar:
    nav: "docs"
---



# static import사용

자바 클래스의 static 메서드는 객체 생성없이 메서드를 이용할 수 있다. 예를 들면 절대값을 구하는 java.lang.Math 클래스의 abs() 메서드는 Math.abs() 로 사용 할 수 있다.

이러한 정적(static) 메서드를 더 쉽게 사용하기 위해서 static import 을 사용 할 수 있다. 다음과 같이 static import 을 하면 클래스명 없이 바로 메서드를 이용할 수 있다. 만약 같은 클래스에 동일한 이름의 메서드가 있으면 그 메서드가 우선된다.

```java
import static java.lang.Math.abs;

int n = abs(-1);
```



정적 메서드외에 정적 멤버 변수도 static import의 대상이다.

```java
import static java.lang.Math.PI;

System.out.println(PI);
```
