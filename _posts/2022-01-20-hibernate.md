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



### SessionFactory 객체

SessionFactory는 애플리케이션마다 1개만 존재한다.

SessionFactory 객체는 session 을 생성한다. Thread safe 한 객체이며 애플리케이션의 모든 스레드에서 사용된다. 무거운 객체라서 일반적으로 애플리케이션 시작 중에 생성이 되고 나중에 사용하기 위해 보관된다. 



### Session 객체

애플리케이션과 DB 간 communication, 일반적으로 Thread safe 하지 않아서 오랫동안 열어두면 안된다. 매핑된 entity class 의 객체에 대한 CRUD 를 제공한다. 

C : Create

R : Read

U : Update

D : Delete



<사진 4>

캐시를 이용하여 작업한 것들을 모았다가 한번에 DB에 반영한다. 매번 바로 DB에 기록하는것은 퍼포먼스에 영향을 주기때문에 캐시를 이용한다. 



#### session 메소드

- Object get(Class clazz, Serializable id) throws HibernateException 

주어진 식별자를 갖는 주어진 클래스의 인스턴스 반환한다. 없을 경우 null을 반환한다.



- Serializable save(Object object) throws HibernateException 

DB에 객체를 저장한다. 



- void saveOrUpdate (Object object) throws HibernateException

주어진 객체가 DB에 이미 있는 경우 update(), 없는 경우 save() 호출한다.



- void delete (Object object) throws HibernateException

DB 에서 주어진 객체 제거  



- void flush() throws HibernateException 

캐시 내용을 DB 반영한다. session 객체에서 save() 할 경우 바로 DB에 저장되는게 아니라 캐시에 있다가 flush()를 통해서 DB에 저장한다. 



- Connection close() throws HibernateException

JDBC 연결을 해제하고 정리하여 Session을 종료한다.  Spring에서 알아서 이 작업을 수행해준다. 



- Query createQuery(String queryString)

SQL 또는 HQL 문자열에 대한 쿼리 객체를 만든다. 



```java
public List<Product> getProductList() {

        Session session = sessionFactory.getCurrentSession();
        Query<Product> query = 
	  		session.createQuery("from Product order by name“, Product.class);
        List<Product> products = query.getResultList();

        return products;
 }

```



HQL(Hibernate Query Language) 특징

- SQL 과 유사하다

- 완전히 객체지향적이다. 
  - HQL은 table, column 이름을 사용하지 않고 class, property 이름을 사용한다. 

- keyword 에 대한 대소문자 구분 없다.
  - SQL에서 SELECT, select 가 구분이 없는것과 같다.

- Java class, property 에 대한 대소문자 구분이 있다. 
  - Person, person 은 다르다.



1. HQL 문 작성

   String hql = "HQL 문"

2. Session으로 부터 Query 생성

   Query query = session.createQuery(hql)

3. 쿼리 실행

   1. select 내용
      1. List list = query.getResultList()
   2. Update, Delete 내용
      1. int rowsAffected = query.executeUpdate()



HQL 예시

Select Query

```java
String hql = "from Product where category.name = 'Computer'";
Query<Product> query = session.createQuery(hql, Product.class);
List<Product> listProducts = query.getResultList();

```



의미 : Category 이름이 Computer인 Product 조회

Product, name 이런 부분은 table, column 이 아니다. 



Update Query

```java
String hql = "update Product set price = :price where id = :id";

// Update/Delete queries can not be typed.
@SuppressWarnings("rawtypes")
Query query = session.createQuery(hql);

query.setParameter("price", 488.0);
query.setParameter("id", 43);
 
int rowsAffected = query.executeUpdate();
```

:price, :id 는 placeholder



Delete Query

```java
String hql = "delete from OldCategory where id = :catId";
 
@SuppressWarnings("rawtypes")
Query query = session.createQuery(hql);
query.setParameter("catId",  1);
 
int rowsAffected = query.executeUpdate();
```











 

​    









```java
@Entity
@Table(name="product")
public class Product {

  @Id
  @GeneratedValue
  @Column(name=“product_id”)
  private int id;

  private String name;

  private int price;

  private String description;

  @ManyToOne(cascade=CascadeType.ALL)
  @JoinColumn(name="category_id")
  private Category category;
}
```

@Entity : Entity Class(DB 테이블과 매핑된 Java Class) 임을 나타냄

@Table : 

@Id : 기본키

@GeneratedValue

@Column(name="column_name")

@JoinColumn(name="column_name")

@ManyToOne(cascade=CascadeType.ALL)









