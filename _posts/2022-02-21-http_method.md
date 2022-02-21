---
layout: single
title:  "HTTP 메서드"
categories: HTTP
tag: [http method]
toc: true
author_profile: false
sidebar:
    nav: "docs"
---



# HTTP 메서드

API URI를 설계할때 URI는 Uniform Resource Identifier 라는 말 그대로 리소스를 식별하는 용도로 사용하고 이 리소스에 대한 행위를 HTTP 메서드로 표현한다. 이 HTTP 메서드를 어떤식으로 사용해야할지 알아보자.



## 종류



### GET

GET 메서드는 주로 리소스를 조회하는 용도로 사용한다. 메시지 바디를 이용하여 데이터를 전달할 수 있지만 지원하지 않는 곳이 많고 권장하지 않는다. 서버에 데이터를 전달하고자 할때 대부분 쿼리 파라미터를 통해서 전달한다. 

GET /members/100

### POST

POST는 요청 데이터를 처리하는 용도로 사용된다. 메시지 바디에 데이터를 넣어 서버로 전달한다. 서버는 메시지 바디를 통해 넘어온 데이터를 처리하는 기능을 수행한다. 주로 신규 리소스 등록, 프로세스 처리에 사용한다. 하지만 다른 메서드로 처리하기 애매한 경우 사용되기도 한다. 

POST /members

### PUT

리소스를 대체할때 사용된다. 기존에 해당 리소스가 있다면 전체가 대체되고(덮어쓰기 생각) 없다면 새로 생성이된다. 사용시 클라이언트가 리소스 위치를 알고 URI를 지정해야한다. 

**변경 전**

```json
{
    "username":"young",
    "age":20
}
```



**적용**

PUT /members/100

```json
{
    "age":50
}
```





**변경 후**

```json
{    
    "age":50
}
```







### PATCH

PUT은 리소스를 완전히 덮어쓰는 형식으로 전체가 변경이 되었다면 PATCH는 리소스를 부분 변경한다. 

**변경 전**

```json
{
    "username":"young",
    "age":20
}
```



**적용**

PATCH /members/100

```json
{
    "age":50
}
```





**변경 후**

```json
{
    "username":"young",
    "age":50
}
```



### DELETE

리소스를 제거할때 사용된다.

DELETE /members/100



### HEAD

GET 처럼 조회하는 용도로 사용하지만 메시지 부분은 제외하고, 상태줄과 헤더만 반환한다. 



## 속성

