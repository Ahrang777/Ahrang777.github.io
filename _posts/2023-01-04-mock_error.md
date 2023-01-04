---
layout: single
title:  "[Spring][solution] 테스트에서 TemplateEngine 이용시 NPE 문제"
categories: solution
tag: [Spring, test, error, solution]
toc: true
author_profile: false
sidebar:
    nav: "docs"
---



# 문제 상황

회원가입시 인증코드를 메일로 전송하는 기능을 구현하기 위해 타임리프로 인증코드 메일 템플릿을 만들고 MailContentBuilder 라는 클래스에서 해당하는 템플릿을 채우는 코드를 구현하였습니다.

```java
public class MailContentBuilder {	
	private final TemplateEngine templateEngine;

	public String build(final String template, final Map<String, String> contents) {
		Context context = new Context();

		for (String key : contents.keySet()) {
			context.setVariable(key, contents.get(key));
		}

		return templateEngine.process(template, context);
	}
}
```

코드 실행은 문제가 없었지만 아래와 같이 테스트 코드를 작성하였더니 TemplateEngine 관련해서 NPE가 발생했습니다.

```java
@ExtendWith(MockitoExtension.class)
class MailContentBuilderTest {
	@Mock
	private TemplateEngine templateEngine;

	@InjectMocks
	private MailContentBuilder mailContentBuilder;

	@DisplayName("메일 템플릿 채우기")
	@Test
	public void mail_content_build() {
		//given
		String template = "test template";
		Map<String, String> contents = new HashMap<>();

		//when
		mailContentBuilder.build(template, contents);

		//then
		then(templateEngine).should(times(1)).process(anyString(), any(Context.class));
	}
}
```





# 해결

아래와 같이 TemplateEngine 이 아닌 ITemplateEngine 으로 변경하면 정상적으로 동작합니다.

```java
public class MailContentBuilder {	
	private final ITemplateEngine templateEngine;

	public String build(final String template, final Map<String, String> contents) {
		Context context = new Context();

		for (String key : contents.keySet()) {
			context.setVariable(key, contents.get(key));
		}

		return templateEngine.process(template, context);
	}
}

@ExtendWith(MockitoExtension.class)
class MailContentBuilderTest {
	@Mock
	private ITemplateEngine templateEngine;

	@InjectMocks
	private MailContentBuilder mailContentBuilder;

	@DisplayName("메일 템플릿 채우기")
	@Test
	public void mail_content_build() {
		//given
		String template = "test template";
		Map<String, String> contents = new HashMap<>();

		//when
		mailContentBuilder.build(template, contents);

		//then
		then(templateEngine).should(times(1)).process(anyString(), any(Context.class));
	}
}
```



reference    

[참고1](https://stackoverflow.com/questions/43656246/issue-when-unit-testing-a-service-using-thymeleaf-template-engine)
