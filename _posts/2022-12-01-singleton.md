---
layout: single
title:  "싱글톤 패턴(Singleton Pattern)"
categories: sw
tag: [sw, design pattern]
toc: true
author_profile: false
sidebar:
    nav: "docs"
---



# 싱글톤 패턴이란?

GoF 디자인 패턴 중 생성 패턴(Creational Pattern) 중 하나로 객체의 인스턴스가 오직 1 개만 생성되는 패턴을 의미합니다. 다음은 가장 간단한 싱글톤 패턴의 예시 입니다.

```java
public class Singleton {
    private static Singleton instance = new Singleton();
    
    private Singleton() {
        // 생성자는 외부에서 호출못하게 private 으로 지정해야 한다.
    }

    public static Singleton getInstance() {
        return instance;
    }

    // 관련 메서드
    // ...
}
```



## 장점

싱글톤 패턴을 이용하면 인스턴스가 1 개만 존재하는 것을 보장할 수 있습니다. 이러한 싱글톤 패턴의 장점은 다음과 같습니다.

1. **메모리 낭비를 방지할 수 있습니다.** 

   인스턴스가 1 번만 생성되기 때문에 불필요한 메모리 낭비를 방지할 수 있습니다. 

2. **다른 클래스 간에 데이터 공유가 쉽습니다.** 

   싱글톤 인스턴스가 전역으로 사용되는 인스턴스이기 때문에 다른 클래스의 인스턴스들이 접근하여 사용할 수 있습니다. 동시성 문제를 고려해야 합니다. 



웹 서버 개발을 할 때 클라이언트 요청에 따라 매번 객체 생성을 할 수 없기에 싱글톤 패턴은 필수적입니다. 



## 문제점

싱글톤 패턴을 이용하면 효율에서의 이점을 얻을 수 있지만 많은 문제점을 수반하기에 trade-off를 잘 고려해야 합니다. 

1. **싱글톤 패턴을 구현하는데 많은 노력이 요구됩니다.** 

   일반 클래스에 단순히 싱글톤 패턴을 적용하는데 많은 코드들이 필요합니다. 멀티 쓰레딩 환경에서 동시성 문제같은 고려할 사항이 있습니다. 

2. **테스트하기 어렵습니다.** 

   싱글톤 인스턴스는 자원을 공유하고 있기 때문에 격리된 환경으로 테스트를 수행하기 위해서는 매번 인스턴스의 상태를 초기화시켜줘야 합니다. 

3. **의존 관계상 클라이언트가 구체 클래스에 의존하게 됩니다.** 

   new 키워드를 이용하여 클래스 안에서 객체를 생성하고 있으므로, SOLID 원칙 중 DIP를 위반하게 되고 OCP 또한 위반할 가능성이 높습니다.

4. **자식 클래스를 만들 수 없습니다.** 

   private 생성자를 이용하기 때문에 자식 클래스를 만들 수 없습니다. 

5. **내부 상태를 변경하기 어렵습니다.**



싱글톤 패턴은 여러 문제점들이 존재하고 결과적으로 유연성이 많이 떨어지는 패턴이라고 할 수 있습니다. 



## 싱글톤 패턴 적용 

여기서는 싱글톤 패턴을 적용해가며 해당 방법의 문제점과 해결방안을 정리합니다. 



### Eager Initialization

이 방식은 가장 간단하지만 클래스 로딩 시점에 초기화되어 인스턴스가 필요하지 않는 경우에도 생성됩니다. 

```java
public class Singleton {
    private static Singleton instance = new Singleton();
    
    private Singleton() {
        // 생성자는 외부에서 호출못하게 private 으로 지정해야 한다.
    }

    public static Singleton getInstance() {
        return instance;
    }

    // 관련 메서드
    // ...
}
```



### Lazy Initialization

Eager Initialization의 문제점을 이 방식을 통해 해결할 수 있습니다. getInstance() 메서드에서 if문을 통해 해당 객체가 필요할 때 1 번만 초기화 되도록 합니다. 

```java
public class Singleton {
    private static Singleton instance;
    
    private Singleton() {
        // 생성자는 외부에서 호출못하게 private 으로 지정해야 한다.
    }

    public static Singleton getInstance() {
        if (instance == null) {
            instance = new Singleton();
        }
        return instance;
    }

    // 관련 메서드
    // ...
}
```

하지만 이 방법은 멀티 쓰레드 환경에서 race condition때문에 제대로 동작하지 않는 문제가 발생합니다. 

1. Singleton 객체가 아직 생성되지 않았을 경우 쓰레드1이 getInstance 메서드의 if문을 실행하여 instance가 null임을 확인했습니다.
2. 쓰레드1이 생성자를 호출하여 인스턴스를 생성하기 전 쓰레드2 가 getInstance 메서드의 if문을 실행하고 아직 인스턴스가 생성되지 않았기에 생성자를 호출하여 인스턴스를 생성합니다. 
3. 쓰레드1은 이어서 if문을 이전에 통과했기에 이어서 인스턴스를 생성합니다.

이처럼 싱글톤 패턴을 적용했음에도 인스턴스가 여러개 생성되는 문제가 발생합니다. 



### Synchronized 사용

다음과 같이 synchronized 를 이용하여 동기화하는 방법을 이용한다면 이 문제를 해결할 수 있습니다. 하지만 synchronized는 자주 사용할 경우 성능을 저하시킬 수 있습니다. 이 경우 getInstance 메서드의 if문 까지 굳이 synchronized 할 필요가 없습니다. 

```java
public class Singleton {
    private static Singleton instance;
    
    private Singleton() {
        // 생성자는 외부에서 호출못하게 private 으로 지정해야 한다.
    }

    public synchronized static Singleton getInstance() {
        if (instance == null) {
            instance = new Singleton();
        }
        return instance;
    }

    // 관련 메서드
    // ...
}
```



참고

synchronized 키워드를 사용할 경우 자바 내부적으로 해당 영역이나 메서드를 lock, unlock 처리하기 때문에 내부적으로 많은 cost가 발생합니다. 따라서 많은 thread 들이 getInstance()를 호출하게 되면 프로그램 전반적인 성능저하가 발생합니다.



### DCL(Double Checked Locking)

이 방법을 통해 if문을 통과한 처음 인스턴스를 생성하는 경우 1 번만 synchronized 를 적용시킵니다.

```java
public class Singleton {
    private static Singleton instance;
    
    private Singleton() {
        // 생성자는 외부에서 호출못하게 private 으로 지정해야 한다.
    }

    public static Singleton getInstance() {
        if (instance == null) {
            synchronized(Singleton.class) {
                if (instance == null) {
                    instance = new Singleton();
                }
            }
        }
        return instance;
    }

    // 관련 메서드
    // ...
}
```

이 방법의 경우 문제가 없어보이지만 실제로 문제가 존재합니다. 

**정상적인 동작**

1. Singleton 인스턴스를 위한 메모리 할당
2. 생성자를 통한 초기화
3. 할당된 메모리를 instance 변수에 할당

**문제 발생**

1. Singleton 인스턴스를 위한 메모리 할당
2. 할당된 메모리를 instance 변수에 할당
3. 생성자를 통한 초기화



**문제 상황**

1. Singleton 인스턴스가 아직 생성되지 않았을 때 쓰레드1이 getInstance 메서드의 첫 번째 if문을 실행하여 instance 상태가 null임을 확인합니다.
2. Singleton 인스턴스를 위한 메모리를 할당합니다.
3. 할당된 메모리를 instance 변수에 할당합니다. 
4. 쓰레드2가 getInstance 메서드의 첫 번째 if문을 실행하여 instance 변수가 생성된 것을 확인하고 생성된(그러나 아직 초기화 되지 않은) 인스턴스를 반환 받습니다.
5. 쓰레드2가 Singleton 인스턴스를 사용하려고 하면 문제가 발생합니다. 

경우에따라 컴파일러가 성능최적화를 위해서 순서를 바꿀수 있습니다. 명령어 reorder



### volatile를 사용하여 명령어 reorder 금지

```java
public class Singleton {
    private volatile static Singleton instance;
    
    private Singleton() {
        // 생성자는 외부에서 호출못하게 private 으로 지정해야 한다.
    }

    public static Singleton getInstance() {
        if (instance == null) {
            synchronized(Singleton.class) {
                if (instance == null) {
                    instance = new Singleton();
                }
            }
        }
        return instance;
    }

    // 관련 메서드
    // ...
}
```



### Initialization on demand holder idiom

**Demand(또는 Lazy) Holder 방식은 가장 많이 사용되는 싱글톤 구현 방식입니다.** 

volatile이나 synchronized 키워드 없이도 동시성 문제를 해결할 수 있습니다. 

처음 getInstance 메서드를 호출하면 그때서야 LazyHolder 가 참조됩니다. 이때 참조가 되는 순간에 LazyHolder 내부 클래스가 로딩되면서 Singleton 객체가 생성됩니다. Singleton 이 로딩된다 하더라도 LazyHolder를 참조하지 않으면 인스턴스가 만들어지지 않습니다. getInstance 메서드를 호출하는 순간에만 LazyHolder가 참조가 되고 그때서야 LazyHolder가 로딩되면서 Singleton instance 값이 생성됩니다. 

```java
public class Singleton {
    private static Singleton instance;
    
    private Singleton() {
        // 생성자는 외부에서 호출못하게 private 으로 지정해야 한다.
    }
    
    private static class LazyHolder {
        private static final Singleton INSTANCE = new Singleton();
    }

    public static Singleton getInstance() {
        return LazyHolder.INSTANCE;
    }

    // 관련 메서드
    // ...
}
```





# 결론

싱글톤 패턴은 여러 장단점이 존재하기에 trade-off 를 고려해야 합니다. 싱글톤 패턴은 안티 패턴으로 불릴만큼 단독으로 사용한다면 객체 지향에 위반되는 사례가 많습니다. 때문에 스프링 프레임워크 같은 싱글톤 컨테이너의 도움을 받으면 문제점들을 보완하며 사용할 수 있습니다. 



다음은 이전에 스프링을 학습하며 정리한 싱글톤 관련 글입니다. 이 글과 함께 참고해 주세요.

[이전 글](https://ahrang777.github.io/spring/web_singleton/)







reference    

[참고1](https://tecoble.techcourse.co.kr/post/2020-11-07-singleton/)

[참고2](https://limkydev.tistory.com/67)
