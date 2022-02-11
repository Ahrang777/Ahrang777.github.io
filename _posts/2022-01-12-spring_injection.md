---
layout: single
title:  "생성자 주입 권장"
categories: Spring
tag: DI
toc: true
author_profile: false
sidebar:
    nav: "docs"
---

# 다양한 의존성 주입 방법

## 생성자 주입 (Constructor Injection)

생성자를 통하여 의존관계를 주입하는 방법이다.

```java
@Service 
public class UserServiceImpl implements UserService { 
    private UserRepository userRepository; 
    private MemberService memberService; 
    
    @Autowired 
    public UserServiceImpl(UserRepository userRepository, MemberService memberService) { 
        this.userRepository = userRepository; 
        this.memberService = memberService; 
    } 
}
```

생성자 주입은 생성자 호출 시점에 1회 호출되는 것이 보장된다. 때문에 주입받은 객체가 변하지 않거나, 반드시 객체 주입이 필요한 경우에 강제하기 위해 사용 할 수 있다. 스프링 프레임워크에서는 생성자 주입을 권장하여 생성자가 1개만 있을경우 @Autowired를 생략하여도 주입이 가능하다. 위, 아래 코드는 동일한 내용이다. 
생성자가 여러개 있을 경우 특정 생성자 하나에 @Autowired를 지정해 줘야한다. 여러개의 생성자에 @Autowired를 붙일 수는 없다. 

```java
@Service 
public class UserServiceImpl implements UserService { 
    private UserRepository userRepository; 
    private MemberService memberService; 
    
    public UserServiceImpl(UserRepository userRepository, MemberService memberService) { 
        this.userRepository = userRepository; 
        this.memberService = memberService; 
    } 
}
```


## 수정자 주입 (Setter 주입, Setter Injection)

필드값을 변경하는 수정자를 통해서 의존관계를 주입하는 방식이다.

```java
@Service 
public class UserServiceImpl implements UserService { 
    private UserRepository userRepository; 
    private MemberService memberService; 
    
    @Autowired 
    public void setUserRepository(UserRepository userRepository){
        this.userRepository = userRepository;
    }

    @Autowired 
    public void setMemberService(MemberServicey memberService){
        this.memberService = memberService;
    }
}
```



수정자 주입은 **선택**, **변경** 가능성이 있는 의존관계에 사용된다.  @Autowired로 주입할 대상이 없는 경우에는 오류가 발생한다. 주입할 대상이 없어도 동작하도록 하려면 @Autowired(required=false) 를 통해 설정할 수 있다.




## 필드 주입 (Field Injection)

```java
@Service 
public class UserServiceImpl implements UserService { 

    @Autowired 
    private UserRepository userRepository; 
    @Autowired 
    private MemberService memberService; 

}
```

필드 주입을 하면 코드가 간결해져서 과거에 많이 사용했지만 외부에서 변경이 불가능하여 테스트하기 힘들다는 단점이 있다. 또한 DI 프레임워크가 없다면 주입할 방법이 없다. 때문에 최근에는 거의 사용하지 않는다. 




## 생성자 주입을 사용해야 하는 이유

1. 테스트 코드의 작성

   ```java
   @Service
   public class UserServiceImpl implements UserService{
   
       @Autowired
       private UserRepository userRepository;
       @Autowired
       private MemberService memberService;
   
       @Override
       public void register(String name){
           userRepository.add(name);
       }
   }
   
   
   public class UserServiceTest{
   
       @Test
       public void addTest(){
           UserService userService = new UserServiceImpl();
           userService.register("AAA");
       }
   
   }
   ```

   필드 주입일 경우 스프링 컨테이너의 도움 없이는 의존성 주입을 할 수 없어서 테스트가 불가능하다. 위와 같은 경우 의존성 주입이 이루어지지 않아서 userRepository 가 null 이고 이런 userRepository가 add메서드를 호출하는 경우 NPE(java.lang.NullPointerException) 이 발생한다.

   이를 해결하기 위해 Setter를 사용한다면 싱글톤 패턴 기반의 빈이 변경될 수 있다는 문제가 생긴다. 반대로 테스트 코드에서 @Autowired를 사용하기 위해 스프링 빈을 올리면 컴포넌트들을 등록하고 초기화 하는데 시간이 커져 테스트 비용이 증가하게 된다. 때문에 테스트를 빠르게 자주 돌릴수 없다. 

   생성자 주입을 사용한다면 순수 자바코드로 테스트 코드를 작성할 수 있고 주입할 객체가 누락이 된 경우 컴파일 오류가 발생하여 바로 확인가능하다.





2. final 키워드 작성, Lombok과 결합으로 코드 간결

   생성자 주입을 사용하면 필드를 final로 선언할 수 있다. 때문에 만약 final로 선언된 필드를 생성자에서 초기화하는 것을 실수로 누락했을 경우 컴파일 오류로 확인할 수 있다. 또한 final 키워드 사용으로 불변성을 보장한다. 다른 주입 방법들은 객체의 생성(생성자 호출) 이후에 호출되므로 final 키워드를 사용할 수 없다. 

   그리고 필드에 final 키워드를 사용할 경우 lombok 을 이용하여 코드를 간결하게 만들 수 있다. 

   ```java
   @Service
   public class UserServiceImpl implements UserService{
   
       @Autowired
       private UserRepository userRepository;
       @Autowired
       private MemberService memberService;
   
       @Override
       public void register(String name){
           userRepository.add(name);
       }
   }
   
   @Service
   @RequiredArgsConstructor
   public class UserServiceImpl implements UserService{
   
       @Autowired
       private UserRepository userRepository;
       @Autowired
       private MemberService memberService;
   }
   ```

   위 코드는 서로 같은 내용이다 . @RequiredArgsConstructor 는 final이 붙은 필드를 모아서 생성자를 자동으로 만들어주고 생성자가 1개일 경우 @Autowired를 생략해도 생성자 주입이 되기때문이다.



3. 순환 참조 에러 방지

   애플리케이션 구동 시점(객체의 생성 시점)에 순환 참조 에러를 방지할 수 있다.

   예를 들어 UserServiceImpl의 register 함수가 memberService의 add를 호출하고, memberServiceImpl의 add함수가 UserServiceImpl의 register 함수를 호출한다면 어떻게 될까?

   ```java
   @Service
   public class UserServiceImpl implements UserService {
   
       @Autowired
       private MemberServiceImpl memberService;
       
       @Override
       public void register(String name) {
           memberService.add(name);
       }
   
   }
    
   
   @Service
   public class MemberServiceImpl extends MemberService {
   
       @Autowired
       private UserServiceImpl userService;
   
       public void add(String name){
           userService.register(name);
       }
   
   }
   ```

   위의 두 메소드는 서로를 계속 호출할 것이고, 메모리에 함수의 CallStack이 계속 쌓여 StackOverflow 에러가 발생하게 된다.

   ```
   Caused by: java.lang.StackOverflowError: null
   	at com.mang.example.user.MemberServiceImpl.add(MemberServiceImpl.java:20) ~[main/:na]
   	at com.mang.example.user.UserServiceImpl.register(UserServiceImpl.java:14) ~[main/:na]
   	at com.mang.example.user.MemberServiceImpl.add(MemberServiceImpl.java:20) ~[main/:na]
   	at com.mang.example.user.UserServiceImpl.register(UserServiceImpl.java:14) ~[main/:na]
   ```

   만약 이러한 문제를 발견하지 못하고 서버가 운영된다면 해당 메소드의 호출 시에 StackOverflow 에러에 의해 서버가 죽게 될 것이다.
   하지만 생성자 주입을 이용하면 이러한 순환 참조 문제를 방지할 수 있다.

   ```
   Description:
   
   The dependencies of some of the beans in the application context form a cycle:
   
   ┌─────┐
   |  memberServiceImpl defined in file [C:\Users\Mang\IdeaProjects\build\classes\java\main\com\mang\example\user\MemberServiceImpl.class]
   ↑     ↓
   |  userServiceImpl defined in file [C:\Users\Mang\IdeaProjects\build\classes\java\main\com\mang\example\user\UserServiceImpl.class]
   └─────┘
   ```

   애플리케이션 구동 시점(객체의 생성 시점)에 에러가 발생하기 때문이다. 그러한 이유는 Bean에 등록하기 위해 객체를 생성하는 과정에서 다음과 같이 순환 참조가 발생하기 때문이다.

   ```java
   new UserServiceImpl(new MemberServiceImpl(new UserServiceImpl(new MemberServiceImpl()...)))
   ```

   
