---
title: "Spring Web MVC 프레임워크 보기"
date: 2023-02-22
update: 2023-02-22
tags:
  - Spring
  - MVC
series: "Spring"
---
# Model-View-Controller (MVC)
[MVVM과 MVC의 패턴의 차이](https://ppeper.github.io/android/android-acc/#-mvc-vs-mvvm)를 알아보면서 간단히 보았던 MVC 아키텍처 패턴은 애플리케이션의 확장을 위해 세가지 영역(Model, View, Controller)으로 분리 한 것으로 이러한 Model 2(Web MVC) 아키텍처 패턴은 화면과 비지니스 로직을 분리해서 작업하기 때문에 __확장성이 뛰어나고__, 표준화된 코드를 사용하여 공동작업이 용이하고 __유지보수성이 좋다.__

# 스프링 MVC
스프링 프레임워크는 자바 언어 기반의 프레임워크 이지만 이중 __웹 애플리케이션을 개발__ 할 때 사용할 수 있는 Servlet 기반의 WEB 개발을 위한 MVC 프레임워크를 제공하여  Model2 Architecture, Front Controller Pattern을 프레임워크 차원에서 제공한다. 스프링 MVC 프레임워크는 스프링을 기반으로 하여 스프링에서 제공하는 DI나 AOP 기능들과 함께 웹 MVC 개발하기 용이하게 해준다.

<h5>스프링 MVC 구조</h5>
<img src="https://user-images.githubusercontent.com/63226023/220362426-46d6197a-2eeb-4dbc-8be9-cd61c5c0980f.png">

<h5>요청 순서</h5>

1. 클라이언트가 HTTP 요청을 서버에 보내면 DispatcherServlet이 요청을 받는다.
2. HandlerMapping이 클라이언트가 요청한 URL과 매핑된 Controller를 찾는다.
3. 해당 Controller가 요청을 처리하고 Model을 반환한다.
4. ViewResolver가 Controller에서 반환한 논리적인 View 이름을 물리적인 View 파일 경로로 변환한다.
5. View가 생성되고 처리되면 DispatcherServlet은 처리된 응답을 클라이언트에 보낸다.

## 구성 요소
- <h5>DispatcherServlet (Front Controller)</h5>
    - 모든 클라이언트의 요청을 전달 받는다.
    - Controller에게 클라이언트 요청을 전달하고, Controller가 리턴한 결과값을 View에게 전달하여 알맞은 응답을 생성한다.
- <h5>HandlerMapping</h5>
    - 클라이언트의 요청 URL을 어떤 Controller가 처리할지를 결정한다.
    - URL과 요청 정보를 기준으로 어떤 핸들러 객체를 사용할지 결정하는 객체이며, DispatcherServlet은 하나 이상의 핸들러 매핑을 가질 수 있다.
- <h5>Controller</h5>
    - 클라이언트의 요청을 처리한 뒤, Model을 호출한 후 그 결과를 DispatcherServlet에 반환한다.
- <h5>ModelAndView</h5>
    - Controller가 처리한 데이터 및 화면에 대한 정보를 보유한 객체이다.
- <h5>ViewResolver</h5>
    - Controller가 반환한 이름을 기반으로 Controller의 처리 결과를 보여줄 View를 결정한다.
- <h5>View</h5>
    - Controller의 처리결과를 보여줄 화면을 생성한다.

- - -
# 스프링 Web MVC 구현
- 스프링 MVC를 이용한 Application 구현의 순서는 다음과 같다.
    - `web.xml` 에 DispatcherServlet과 스프링 설정파일을 등록한다.
    - 설정 파일에 HandlerMapping을 설정한다.
    - Controller 구현 및 `servlet-context.xml`에 등록한다.
    - Controller와 View(JSP)의 연결을 위해 `View Resolver`를 설정해 준다.
    - 클라이언트가 볼 View(JSP) 코드를 작성한다.

## 스프링 MVC 프로젝트 생성해보기
스프링 STS를 통하여 `Spring Legacy Project` 의 템플릿에 있는 `Spring MVC Project`를 생성하면 다음과 같은 프로젝트 구조를 볼 수 있다. 

<img src="https://user-images.githubusercontent.com/63226023/220368311-15d90709-6e23-4dd2-a51d-7d0dd8469bb9.png">

## web.xml
기본적인 `web.xml` 파일의 전체 구성은 다음과 같다.
```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app>

    <!-- 스프링에서 사용할 객체(빈)들을 등록한 최상위 xml -->
    <context-param>
      <param-name>contextConfigLocation</param-name>
      <param-value>/WEB-INF/spring/root-context.xml</param-value>
    </context-param>

    <listener>
        <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
    </listener>

    <servlet>
        <!-- 서블릿의 이름을 appServlet으로 지정하고 DispatcherServlet 로 정의한다-->
        <servlet-name>appServlet</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <!-- 위의 DispatcherServlet이 알아야하는 환경설정들이 있는 servlet-context.xml의 위치를 알려준다. -->
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>/WEB-INF/spring/appServlet/servlet-context.xml</param-value>
        </init-param>
        <load-on-startup>1</load-on-startup>
    </servlet>

  <!-- / 로 들어오는 요청은 appServlet 라는 이름의 servlet 이 처리한다 -> 위에서의 DispatcherServlet-->
  <servlet-mapping>
    <servlet-name>appServlet</servlet-name>
    <url-pattern>/</url-pattern>
  </servlet-mapping>
</web-app>
```

### Root Context
```xml
<!-- 스프링에서 사용할 객체(빈)들을 등록한 최상위 xml -->
<context-param>
    <param-name>contextConfigLocation</param-name>
    <param-value>/WEB-INF/spring/root-context.xml</param-value>
</context-param>

<listener>
    <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
</listener>
```
먼저 Context 파일들을 로드하기 위해 ContextLoaderListener를 설정한다. 리스너가 설정이 되면 해당하는 `<context-param>`으로 표현된 `/WEB-INF/spring/root-context.xml` 파일을 읽어서 공통적으로 사용되는 최상위 Context를 생성한다.

이외에 다른 Context 파일들을 최상위 애플리케이션 Context로 로드하기 위해서는 `<param-value>` 태그 안에 추가하면 된다.

### DispatcherServlet
```xml
<servlet>
    <!-- 서블릿의 이름을 appServlet으로 지정하고 DispatcherServlet 로 정의한다-->
    <servlet-name>appServlet</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <!-- 위의 DispatcherServlet이 알아야하는 환경설정들이 있는 servlet-context.xml의 위치를 알려준다. -->
    <init-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>/WEB-INF/spring/appServlet/servlet-context.xml</param-value>
    </init-param>
    <load-on-startup>1</load-on-startup>
</servlet>

<!-- / 로 들어오는 요청은 appServlet 라는 이름의 servlet 이 처리한다 -> 위에서의 DispatcherServlet-->
<servlet-mapping>
    <servlet-name>appServlet</servlet-name>
    <url-pattern>/</url-pattern>
</servlet-mapping>
```
이후에는 `DispatcherServlet`을 설정을 해주어야 한다. `<init-param>` 을 성정 하지 않으면 기본적으로 `<servlet-name>-servlet.xml` 파일에서 ApplicationContext의 정보를 로드하게 된다.

스프링 컨테이너는 설정파일의 내용을 읽고 ApplicationContext 객체를 생성하게 된다. Servlet이므로 1개 이상의 DispatcherServlet을 설정이 가능하고, `<url-pattern>` 에서 `/`로 설정하여 __/book__, __/list__ 와 같은 요청이 들어오면 __appServlet__ 이 요청을 처리하게 된다.

### servlet-context.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans:beans xmlns="http://www.springframework.org/schema/mvc"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:beans="http://www.springframework.org/schema/beans"
	xmlns:context="http://www.springframework.org/schema/context"
	xsi:schemaLocation="http://www.springframework.org/schema/mvc https://www.springframework.org/schema/mvc/spring-mvc.xsd
		http://www.springframework.org/schema/beans https://www.springframework.org/schema/beans/spring-beans.xsd
		http://www.springframework.org/schema/context https://www.springframework.org/schema/context/spring-context.xsd">

	<!-- DispatcherServlet Context: defines this servlet's request-processing infrastructure -->
	
	<!-- Enables the Spring MVC @Controller programming model -->
	<annotation-driven />

	<!-- Handles HTTP GET requests for /resources/** by efficiently serving up static resources in the ${webappRoot}/resources directory -->
	<resources mapping="/resources/**" location="/resources/" />

	<!-- Resolves views selected for rendering by @Controllers to .jsp resources in the /WEB-INF/views directory -->
	<beans:bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
		<beans:property name="prefix" value="/WEB-INF/views/" />
		<beans:property name="suffix" value=".jsp" />
	</beans:bean>
	
</beans:beans>
```
`servlet-context.xml` 파일을 보게되면 `ViewResolver` 가 설정되어 있는 것을 볼 수 있다. 이로 인하여 ViewResolver에게  문자열 __"book"__ 이 전달 된다고 가정하면 prefix의 경로 `/WEB-INF/views/`를 앞에 더하고 suffix `.jsp` 를 통하여 __/WEB-INF/views/book.jsp__ 파일을 보여 주게 된다.

<p class="notice--info">📍따라서 이 경로에 해당하는 파일이 있는지 확인해야한다</p>

## root-servlet vs servlet-context
설정파일을 보게되면 root-servlet과 servlet-context 두곳에서 빈에 대한 설정을 하는 것을 볼 수 있다. 

여기서 __ApplicationContext는 ContextLoaderListener 클래스__ 에 의해 만들어지고, __WebApplicationContext는 DispatcherServlet 클래스__ 에 의해 만들어 진다. 해당 Context들의 관계는 다음과 같다.

<p align="center"><img src="https://user-images.githubusercontent.com/63226023/220616153-acc474b3-2554-466a-a725-2c2e37ba2488.png" width="50%"></p>

> __ApplicationContext__
>   - 최상위 컨텍스트로 `root-context`에서 등록되는 빈들은 모든 컨텍스트에서 사용이 가능하다. (공유 가능)
>   - 따라서 서로 다른 서블릿 컨텍스트에서 공유해야 하는 빈들을 등록해 놓고 사용이 가능하다.
>   - __servlet-context 내 빈들은 사용이 불가능하다.__

> __WebApplicationContext__
>   - servlet-context에 등록되는 빈들은 해당 컨텍스트에서만 사용이 가능하다.
>   - DispatcherServlet이 직접 사용하는 Controller를 포함한 웹 관련 빈들을 등록하는 데 사용한다.
>   - __root-context 내 빈들을 사용이 가능하다.__

- - -
# @Controller
클래스파일 내에 `@Controller` 어노테이션이 적용된 클래스는 클라이언트의 요청을 처리한다. 
```java
@Controller
public class HomeController {

    @RequestMapping(value = "/test", method = RequestMethod.GET)
    public String home(Locale locale, Model model) {
        model.addAttribute("hello", "Hello Spring");
        return "home";
    }
	
}
```
`@Controller` 어노테이션은 스프링에서 관리해야할 빈 객체인 것을 알아야한다. 빈을 등록하기 위해서는 빈을 설정하는 방법과 자동 스캔을 통한 방법이 있다.
```xml
<!-- servlet-context.xml -->
<beans:bean id="myController" class="com.spring.mvc.controller.HomeController">
    <!-- property에 서비스등을 사용하고 있다면 -->
    <beans:property name="myService" ref="MyService">
    ...
</beans:bean>
<!-- 자동 스캔을 통하여 해당 패키지 아래에 있는 component들을 모두 등록한다.-->
<context:component-scan base-package="com.spring.mvc" />
```
위에서의 간단한 예시를 보게되면 RequestMapping의 `/test` 로 GET 요청이 들어오면 view에서 사용할 model에 `"hello"`의 이름에 __Hello Spring__ 문자열이 담겨 `home` 이라는 이름의 View를 반환한다.

`home`은 위에서 보았던 __ViewResolver__ 가 경로에 맞는 __home.jsp__ 파일을 찾아 보여주게 된다.

```jsp
<html>
<head>
    <title>Home</title>
</head>
<body>
<h1> Message: ${hello}. </h1>
</body>
</html>
```

<img src="https://user-images.githubusercontent.com/63226023/220625753-87adb565-52a7-4f9b-ac39-0fa903dc7958.png">

## @RequestMapping
클래스타입과 메소드에 설정이 가능하며 __요청 URL mapping 정보를 설정할 때 사용__ 할 수 있으며 HTTP 메소드에 따라 서로 다른 메소드를 mapping 할 수 있다.

```java
@RequestMapping(value = "/test", method = RequestMethod.GET)
@RequestMapping(value = "/test", method = RequestMethod.POST)
---
```
> 이외에도 컨트롤러 메소드 parameter로 다양한 Object를 받을 수 있다.

| Parameter Type                                           | 설명                                                                                                                                         |
| -------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------- |
| HttpServletRequest<br>HttpServletResponse<br>HttpSession | 필요시 Servlet API를 사용할 수 있다.                                                                                                         |
| @PathVariable                                            | URI 템플릿 변수에 접근 할 때 사용한다.                                                                                                       |
| @RequestParam                                            | HTTP 요청 파라미터를 매핑한다.                                                                                                               |
| @RequestHeader                                           | HTTP 요청 헤더를 매핑한다.                                                                                                                   |
| @CookieValue                                             | HTTP 쿠키를 매핑한다.                                                                                                                        |
| @RequestBody                                             | HTTP 요청의 body 내용에 접근할 때 사용한다.                                                                                                  |
| Map, Model, ModelMap                                     | View에 전달한 model data를 설정할 때 사용 (예시에선 Model 사용)                                                                              |
| DTO                                                      | HTTP 요청 parameter를 저장한 객체이다.<br>기본적으론 클래스 이름을 모델명으로 사용한다.<br>@ModelAttribute 설정으로 모델명을 설정할 수 있다. |

> 반환하는 타입또한 다양하게 줄 수 있다.

| Return Type   | 설명                                                                                                                                                                                                           |
| ------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| ModelAndView  | Model 정보 및 View 정보를 담고 있는 ModelAndView 객체이다.                                                                                                                                                     |
| Model, Map    | View에 전달할 객체 정보를 담고 있는 Model/Map 객체이다.<br>이때 View 이름은 요청 URL로부터 결정된다.(RequestToViewNameTranslator)                                                                              |
| String        | View의 이름을 반환한다.                                                                                                                                                                                        |
| void          | Method가 ServletResponse나 HttpServletResponse 타입의 parameter를 갖는 경우 method가 직접 응답을 처리한다고 가정한다. 그렇지 않을 경우 요청 URL로부터 결정된 View를 보여준다.<br>(RequestToViewNameTranslator) |
| @ResponseBody | Method에서 @ResponseBody 어노테이션이 적용된 경우, 반환 객체를 HTTP 응답으로 전송한다.<br>HttpMessageConverter를 이용해서 객체를 HTTP 응답 스트림으로 변환한다.                                                |


- - -
# References 
- [https://kingofbackend.tistory.com/78](https://kingofbackend.tistory.com/78)