---
layout: single
title:  "JPA와 Hibernate"
categories: JPA
tag: [JPA, hibernate]
toc: true
author_profile: false
sidebar:
    nav: "docs"
---



# ORM (Object-Relational Mapping)

객체와 관계형 데이터베이스 매핑을 시켜주는 프레임워크이다. 이를 통해 우리는 SQL 쿼리를 이용하는게 아닌 DB도 객체 지향적으로 프로그래밍 할 수 있다. 



# JPA

JPA는 Java Persistence API(자바 ORM 기술에 대한 API 표준 명세) 의 약자로 자바에서 ORM을 사용하기 위한 인터페이스의 모음이다. 인터페이스기 때문에 JPA만으로 사용할 수 없고 구현체가 필요하다.



# Hibernate 

JPA 를 사용하기 위해서 JPA를 구현한 ORM 프레임워크 중 하나이다.



<사진1>



Hiberante을 사용함으로 JDBC 코드의 양을 최소하 할 수 있다. 우리는 객체 지향적으로 프로그래밍 하고 내부적으로 JDBC 코드가 자동으로 생성된다.

<사진 2>

우리가 Hibernate API를 사용하면 JDBC API로 변환된다. 결국 Hibernate은 DB 사용을 위해 JDBC 코드를 사용한다. 

- SessionFactory

  세션을 만들어 DB와 communication

- hibernate.cfg.xml

  hibernate 설정

- *.hbm.xml class mappings

  객체와 테이블의 매핑관계 확인



<사진 3>











