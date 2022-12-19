---
layout: single
title:  "추상 팩토리 패턴(Abstract Factory Pattern)"
categories: sw
tag: [sw, design pattern]
toc: true
author_profile: false
sidebar:
    nav: "docs"
---



# 추상 팩토리 패턴(Abstract Factory Pattern)이란?

GoF 디자인 패턴 중 생성 패턴(Creational Pattern) 중 하나입니다. 관련성 있는 여러 종류의 객체들을 일관된 방식으로 생성할 때 유용합니다. 구체적으로 어떤 클래스의 인스턴스(concrete product)를 사용하는지 감출 수 있습니다. 

![abstract](https://user-images.githubusercontent.com/59478159/208392065-e7f6615e-270e-47f0-8db8-cb4140c4f005.png)



팩토리를 추상화(abstract, interface ,,)한 형태로 구체적인 팩토리(ConcreteFactory)에서 구체적인 인스턴스(ConcreteProduct)를 만드는 것은 팩토리 메서드 패턴과 유사합니다.



클라이언트 코드(팩토리로부터 생성된 인스턴스를 사용하는 코드)를 인터페이스 기반으로 활용할 수 있도록 해줍니다. 



# 추상 팩토리 패턴 적용 과정

요구사항을 구현한 코드를 개선해가며 최종적으로 추상 팩토리 패턴을 적용



## 요구사항: 맞춤 메일 생성 시스템

채용사이트에서 사용자 맞춤 추천 메일을 보내는 메일 생성 시스템이 있습니다. 

메일을 받을 사용자를 추출하고, 메일 내용을 채워서 메일을 발송하는 시스템입니다. 

주요 흐름은 다음과 같습니다.

1. 발송 대상자 추출
2. 맞춤 메일 내용 생성
3. 메일 발송



맞춤/추천 메일은 여러 종류가 있습니다.

- 사용자 검색 데이터 기반 메일(검색 추천 메일)
- 사용자 정보 기반 메일(개인 맞춤 메일)



여러 종류의 메일을 생성하더라도 메일 생성 시스템(클라이언트) 코드의 변경은 최소화하고자 합니다. 



## 추상화 할 수 있는 기능 찾기: 템플릿 메서드 패턴 적용

메일의 종류가 다른 경우, **발송 대상자 모수**과 **메일 내용**이 다를 것입니다.

경우에 따른 발송 대상자 획득, 경우에 따른 메일 내용 추가 

메일을 만들어주는 MailTemplate을 예시로 보면 발송자/수신자 설정, 하단 Footer 추가 하는 부분은 동일하고 메일 내용관련 부분만 다릅니다. 발송 대상자를 추출하는 부분에서도 경우에 따라 구현해 줘야 하지만 여기서는 MailTemplate 으로만 확인해 보겠습니다.

템플릿 메서드 패턴을 적용하여 코드의 중복을 줄입니다.

![abstract1](https://user-images.githubusercontent.com/59478159/208392091-3e183bb2-07d9-4a00-8715-fcdba34ced70.png)

```java
// 메일 템플릿 생성 추상 클래스
public abstract class MailTemplate {
    public Map<String, String> template = new HashMap<>();
 
    /**
     * 발신자 Setting
     * @param sender 수신자
     */
    public void setSender(String sender) {
        template.put("sender", sender);
    }
 
    /**
     * 수신자 Setting
     * @param receiver 발신자
     */
    public void setReceiver(String receiver) {
        template.put("receiver", receiver);
    }
 
    /**
     * 하단 Footer Setting
     * @param footer 하단 내용
     */
    public void setFooter(String footer) {
        template.put("footer", footer);
    }
 
    /**
     * 메일 내용 Setting
     * @param content 메일 내용
     */
    protected abstract void setContent(String content);
}
 
// 개인 검색 데이터 기반
public class SearchMailTemplate extends MailTemplate {
    @Override
    protected void setContent(String content) {
        System.out.println("Set Content For Search Mail");
        template.put("content", content);
    }
}
 
// 경력, 직무 기반
public class AvatarMailTemplate extends MailTemplate {
    @Override
    protected void setContent(String content) {
        System.out.println("Set Content For Avatar Mail");
        template.put("content", content);
    }
}
```



### 문제점

현재 코드에서의 문제점은 메일 생성 시스템(클라이언트)입장에서, 구체적인 메일 객체(AvatarMailTemplate, SearchMailTemplate)가 필요하게 됩니다. 만약 처음에 사용자 검색 데이터 기반 메일에 대해 구현했다가 사용자 정보 기반 메일로 구성을 변경하려면 코드가 수정되야 합니다. 

구체적인 객체를 new를 이용하여 직접 생성하고 조합하여 사용하고 있기 때문에 문제가 발생했습니다. 이를 해결하기 위해서는 new를 통해 직접 생성하지 말고 구체적인 객체가 아닌 다형성을 이용할 수 있어야 합니다. 



## 확장성 있는 코드로 변경: 팩토리 메서드 패턴 적용

![abstract2](https://user-images.githubusercontent.com/59478159/208392101-3af1713c-e269-4940-909f-e64a4dc7fb04.png)



이전 코드에서 new를 통해 직접 객체를 생성하지 말고 구체적인 클래스가 아닌 추상적인 클래스로 다형성을 활용해야 할 필요가 있었습니다. 팩토리 메서드 패턴을 이용한다면 이 문제를 해결할 수 있습니다. 

```java
// Enum 타입으로 Mail 종류 선언
public enum MailType { AVATAR, SEARCH }
 
// 메일 템플릿 펙토리
public class MailTemplateFactory {
 
    /**
     * Mail Type에 따라 Search, Avatar 메일 템플릿 객체 생성
     *
     * @param mailType 메일 종류
     * @return MailTemplate 메일 템플릿
     */
    public static MailTemplate createMailTemplate(MailType mailType) {
        MailTemplate mailTemplate = null;
 
        switch (mailType) {
            case AVATAR:
                mailTemplate = new AvatarMailTemplate();
                break;
            case SEARCH:
                mailTemplate = new SearchMailTemplate();
                break;
        }
        return mailTemplate;
    }
}
 
// 클라이언트 호출 코드
public class Client {
    public static void main(String[] args) {
        // 팩토리 메소드 호출
        ValidUser avatarValidUser = ValidUserFactory.createValidUser(MailType.AVATAR);
        MailTemplate avatarMailTemplate = MailTemplateFactory.createMailTemplate(MailType.AVATAR);
 
        // 대상자 추출
        Set<Long> validUser = avatarValidUser.getValidUser();
        
        // 메일 내용 Setting
        avatarMailTemplate.setContent("신입/경력 기반 개인 맞춤 메일 내용");
        String sender = avatarMailTemplate.template.get("sender");
        String receiver = avatarMailTemplate.template.get("receiver");
        // ..생략
        
        // 대상자에게 메일 템플릿 설정하여 메일 발송 로직 추가
    }
}
```



### 문제점

1. 여전히 클라이언트 코드의 수정이 필요합니다. 

   팩토리 메서드 패턴을 통해 수정이 줄어들었지만 다른 타입의 메일을 발송하려면 수정이 필요합니다. ex) MailType.AVATAR -> MailType.SEARCH

```java
public class Client {
    public static void main(String[] args) {
        // AS-IS
        ValidUser avatarValidUser = ValidUserFactory.createValidUser(MailType.AVATAR);
        
        // TO-BE
        ValidUser searchValidUser = ValidUserFactory.createValidUser(MailType.SEARCH);
        
        // ...
    }
}
```

2. 새로운 타입의 메일이 추가되는 경우

   모든 팩토리 클래스(ValidFactory, MailTemplateFactory)에 새로운 타입에 대한 처리가 추가되야 합니다. 

3. 필요한 클래스의 객체가 많다면 팩토리 클래스가 증가하여 오히려 복잡해집니다.

   현재는 ValidUser, MailTemplate 에 대한 팩토리 클래스(ValidFactory, MailTemplateFactory) 2개만 필요하지만 만약 메일 발송을 위해 더 많은 종류의 클래스 객체가 필요하다면 그 만큼 팩토리 클래스가 증가할 것입니다. 



## 관련 객체를 일관성 있게 생성: 추상 팩토리 패턴 적용

결국 검색 추천 메일을 생성할 때는 SearchMailTemplate, SearchMailValidUser 객체가 필요하고 개인 맞춤 메일을 생성할 때는 AvatarMailTemplate, AvatarMailValidUser 객체가 필요합니다. 때문에 ValidUserFactory, MailTemplateFactory 처럼 팩토리 클래스를 생성하는 것이 아니라 검색 추천 메일 관련 팩토리 클래스, 맞춤 메일 관련 팩토리 클래스를 구현하여 해당 클래스에서 필요한 객체들을 생성하도록 합니다. 

![abstract3](https://user-images.githubusercontent.com/59478159/208392311-8641e8b5-e97a-4335-b6dd-d1bf9ab1f7d0.png)



이전의 ValidUserFactory, MailTemplateFactory 클래스처럼 **기능 단위**로 팩토리 클래스를 정의하지 않고, SearchMailFactory, AvatarMailFactory 클래스와 같이 **실제 발송할 메일 타입 단위**로 팩토리 클래스를 정의했습니다.

```java
// 추상 대상자, 추상 메일 템플릿을 생성하는 추상 팩토리 클래스
public abstract class MailFactory {
    public abstract Set<Long> createValidUser();
    public abstract MailTemplate createMailTemplate();
}
 
 
// 개인 정보 맞춤 메일 생성 팩토리 클래스
public class AvatarMailFactory extends MailFactory {
    @Override
    public Set<Long> createValidUser() {
        return new AvatarMailValidUser().validUserList;
    }
 
    @Override
    public MailTemplate createMailTemplate() {
        return new AvatarMailTemplate();
    }
}
 
// 검색 기반 맞춤 메일 생성 팩토리 클래스
public class SearchMailFactory extends MailFactory {
    @Override
    public Set<Long> createValidUser() {
        return new SearchMailValidUser().validUserList;
    }
 
    @Override
    public MailTemplate createMailTemplate() {
        return new SearchMailTemplate();
    }
}
 
public class Client {
    public static void main(String[] args) {
        MailFactory mailFactory;
        // 메일 타입을 런타임에 받음
        String mailType = args[0];
 
        if (mailType.equals("AVATAR")) {
            mailFactory = new AvatarMailFactory();
        } else {
            mailFactory = new SearchMailFactory();
        }
 
        // 해당하는 mail 타입에 따른 대상자, 메일 템플릿 생성
        Set<Long> validUser = mailFactory.createValidUser();
        MailTemplate mailTemplate = mailFactory.createMailTemplate();
 
        // (추가) 발송하는 로직 ...
    }
}
```

클라이언트 코드에서 런타임에 인자로 MailType에 따라 메일 생성 팩토리를 생성합니다. 

다른 메일 타입으로 변경되어도 클라이언트 코드를 변경할 필요가 없습니다. (첫 번째 문제 해결)

메일 타입 별 팩토리 클래스를 정의하여 새로운 메일 타입을 기존 코드 변경 없이 적용할 수 있습니다. (두 번째 문제 해결)

메일 타입 별 팩토리 클래스를 정의하기에 새로운 타입이 추가되면 팩토리 클래스만 봤을때는 해당 타입의 팩토리 클래스만 추가하면 됩니다. (세 번째 문제 해결)

![abstract4](https://user-images.githubusercontent.com/59478159/208392331-28fc8e83-0ed7-4381-9f92-38d10e4879d6.png)



# 결론

관련성 있는 여러 객체를 일관적으로 생성하는 다양한 경우(A 클래스 객체를 생성하는데 AProduct1, AProduct2 가 필요하고 다른 케이스인 B클래스 객체 생성하는데 BProduct1, BProduct2 가 필요하며 생성 방식이 동일)가 생길 가능성이 있다면, 추상 팩토리 패턴을 이용하면 Client 와 구체적인 객체 간의 결합을 피할 수 있고 OCP를 지킬 수 있습니다. 



reference    

[참고1](https://dev-youngjun.tistory.com/230)

