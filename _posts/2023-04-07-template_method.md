---
layout: single
title:  "템플릿 메서드 패턴(Template Method Pattern)"
categories: sw
tag: [sw, design pattern]
toc: true
author_profile: false
sidebar:
    nav: "docs"
---



# 템플릿 메소드 패턴(Template Method Pattern)이란?

템플릿 메소드 패턴은 여러 클래스에서 공통으로 메소드를 상위 클래스에서 템플릿으로 정의하고, 하위 클래스에서는 세부 동작을 다르게 구현하는 패턴입니다. 

즉, 변하지 않는 부분(템플릿)을 상위 클래스에 두고 자주 변경되고 확장될 기능은 하위 클래스에 만들어 상위 클래스의 템플릿 메소드의 실행동작 순서는 고정하고 세부 실행을 다양화 할 수 있습니다. 



# 구조

![template_method1](https://user-images.githubusercontent.com/59478159/230580635-0b78da5f-fdd3-4fd4-b512-c6e91eb7d46f.png)

- AbstractClass(추상 클래스) : 템플릿 메소드를 구현하고, 템플릿 메소드에서 사용되는 세부 메소드를 추상 메소드로 선언합니다. 
- ConcreteClass(구현 클래스) : AbstractClass를 상속하고 추상 메소드인 세부 메소드를 구현합니다. 



## hook 메소드

훅(hook) 메소드는 템플릿 메소드의 영향이나 순서를 제어하고 싶을 때 사용되는 메소드 입니다. 아래 코드처럼 템플린 내에서 실행되는 동작을 hook() 메소드의 참, 거짓에 따라 다르게 동작합니다. 이를 통해 자식 클래스에서 좀더 유연하게 템플릿 메소드의 알고리즘 로직을 다양화 할 수 있습니다. 훅 메소드의 경우 AbstractClass에서 정의해두고 자식 클래스에서 필요할 경우 재정의하여 사용할 수 있습니다. 

```java
public void templateMethod() {
    step1();
    step2();
    if(hook()){
        
    }
    step3();
}

boolean hook(){
    return true;
}
```



# 구현

```java
public abstract class AbstractTemplate {
	// final 을 붙여서 자식 클래스에서 오버라이딩 못하게 막음
	public final void templateMethod() {
		step1();
		step2();

		if (hook()) {	// hook() 을 통해 내부로직 실행여부 결정
			// ...
		}

		step3();
	}

	// 기본적으로 제공하고 원하는 경우 자식 클래스에서 오버라이딩 하도록 함
	// step1, 2, 3 은 무조건 오버라이딩 해야하는것과 달리 hook 은 원하는 경우만 오버라이딩 하도록 구현
	protected boolean hook() {
		return true;
	}

	protected abstract void step1();

	protected abstract void step2();

	protected abstract void step3();
}

public class ImplementationA extends AbstractTemplate {
	@Override
	protected void step1() {

	}

	@Override
	protected void step2() {

	}

	@Override
	protected void step3() {

	}
}

public class ImplementationB extends AbstractTemplate {
	@Override
	protected void step1() {

	}

	@Override
	protected void step2() {

	}

	@Override
	protected void step3() {

	}

	@Override
	protected boolean hook() {
		return false;
	}
}

public class Client {
	public static void main(String[] args) {
		AbstractTemplate templateA = new ImplementationA();

		templateA.templateMethod();
	}
}
```



# 특징



## 패턴 사용 시기

- 클라이언트가 알고리즘의 특정 단계만 확장하고, 전체 알고리즘이나 해당 구조는 확장하지 않도록 할때 사용합니다. 
- **동일한 기능은 상위 클래스에서 정의하면서 확장, 변화가 필요한 부분만 하위 클래스에서 구현할 때 사용합니다.**



## 장점

- 상위 추상 클래스에 템플릿 메소드를 만들면서 코드 중복을 줄일 수 있습니다.
- 구체 클래스에 의존하지 않고 추상 클래스에 의존하면서 다형성을 활용할 수 있습니다. 
- 세부 구현에서 변경이 발생하여도 상위 추상 클래스의 템플릿 메소드는 영향을 받지 않습니다. 



## 단점

- 알고리즘 구조가 복잡할수록 템플릿 로직 형태를 구현하기 어려워집니다. 
- 추상 메소드가 많아지면서 클래스의 생성, 관리가 어려워질 수 있습니다. 
- 상위 클래스에서 선언된 추상 메소드를 하위 클래스에서 구현할 때 해당 메소드가 언제 호출되는지 상위 클래스의 로직을 이해해야 합니다. 
- 로직의 변화가 생겨 상위 클래스를 수정할 경우 하위 클래스도 수정이 필요할 수 있습니다. 





# 활용 예제

```java
public abstract class FileProcessor {
	private String path;

	public FileProcessor(String path) {
		this.path = path;
	}

	public final int process() {
		try (BufferedReader reader = new BufferedReader(new FileReader(path))) {
			int result = getResult();
			String line = null;

			while ((line = reader.readLine()) != null) {
				result = calculate(result, Integer.parseInt(line));
			}
			return result;

		} catch (IOException e) {
			throw new IllegalArgumentException(path + "에 해당하는 파일이 없습니다.", e);
		}
	}

	protected abstract int calculate(int result, int number);
	protected abstract int getResult();
}


public class PlusFileProcessor extends FileProcessor {
	public PlusFileProcessor(String path) {
		super(path);
	}

	@Override
	protected int calculate(int result, int number) {
		return result += number;
	}

	@Override
	protected int getResult() {
		return 0;
	}
}


public class MultiplyFileProcessor extends FileProcessor {
	public MultiplyFileProcessor(String path) {
		super(path);
	}

	@Override
	protected int calculate(int result, int number) {
		return result *= number;
	}

	@Override
	protected int getResult() {
		return 1;
	}
}

public class Client {
	public static void main(String[] args) {
		final String path = "src/main/resources/numbers.txt";

		FileProcessor plusFileProcessor = new PlusFileProcessor(path);
		System.out.println(plusFileProcessor.process());

		FileProcessor multiplyFileProcessor = new MultiplyFileProcessor(path);
		System.out.println(multiplyFileProcessor.process());
	}
}
```





reference    

[참고1](https://inpa.tistory.com/entry/GOF-%F0%9F%92%A0-%ED%85%9C%ED%94%8C%EB%A6%BF-%EB%A9%94%EC%86%8C%EB%93%9CTemplate-Method-%ED%8C%A8%ED%84%B4-%EC%A0%9C%EB%8C%80%EB%A1%9C-%EB%B0%B0%EC%9B%8C%EB%B3%B4%EC%9E%90)

