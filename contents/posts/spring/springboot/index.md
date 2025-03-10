---
title: "스프링을 편하게 스프링 부트를 사용해보자"
date: 2023-02-25
update: 2023-02-25
tags:
  - Spring Boot
series: "Spring"
---
<p align="center"><img src="https://user-images.githubusercontent.com/63226023/219384293-26ebb15a-3b13-45c8-abf8-57622c2d0eee.png"></p>

# 스프링 부트
기존의 스프링은 자바 기반의 프레임워크로 애플리케이션을 개발하려면 사전에 많은 작업(library, dependency 추가등) 을 해야 했다. 스프링 부트는 개발자들이 더욱 쉽게 __스프링 애플리케이션을 개발할 수 있도록 도와준다.__

<h5>스프링 부트의 장점</h5>
- project에 따라 자주 사용되는 library 들이 미리 조합되어 있다.
- 복잡한 설정을 자동으로 처리해 준다.
- WAS에 배포하지 않고도 실행할 수 있는 JAR파일로 웹 애플리케이션을 개발 할 수 있다.

스프링 부트는 스프링 프레임워크와 마찬가지로 `Maven`이나 `Gradle`과 같은 빌드 도구를 사용하고, __Tomcat, Jetty, Undertow 등의 웹 서버를 내장하고 있어 별도로 웹 서버를 설치하거나 설정할 필요가 없다.__

| 이름         | 서블릿 버전 | 자바 버전 |
| ------------ | ----------- | --------- |
| Tomcat 8     | 3.1         | Java 7+   |
| Tomcat 7     | 3.0         | Java 6+   |
| Jetty 9      | 3.1         | Java 7+   |
| Jetty 8      | 3.0         | Java 6+   |
| Undertow 1.1 | 3.1         | Java 7+   |

- - -
# 스프링 부트 시작하기

스프링 프레임워크의 경우 configuration설정을 할 때에 어노테이션, 빈 등록등의 설정을 해주어야 했다. 이에 반해 스프링 부트의 경우 `application.properties` 파일이나 `application.yml` 파일에 설정하면 된다.

스프링 부트는 [Spring initializr](https://start.spring.io) 또는 이클립스, 인텔리제이로 쉽게 시작할 수 있다. 스프링 부트를 생성하면 메인 클래스가 생성되고 실행하게 되면 내장된 톰켓 서버가 동작한다.

## 프로젝트 구조

<img src="https://user-images.githubusercontent.com/63226023/221412311-6974b62c-698e-4ba3-a718-49286573024d.png">

| 프로젝트의 주요 파일   | 설명                                                                                                                   |
| ---------------------- | ---------------------------------------------------------------------------------------------------------------------- |
| src/main/java          | 자바 소스 코드들이 있는 directory.                                                                                     |
| SpringbootApplication  | application을 시작할 수 있는 main method가 존재하는 스프링 구성 메인 클래스이다.<br>최상위 패키지에 두는것을 권장한다. |
| static                 | css, js, img등의 정적 리소스들의  directory.                                                                           |
| templates              | SpringBoot에서 사용 가능한 여러가지 View Template(Thymeleaf, Velocity, FreeMarker등) 위치한다.                         |
| application.properties | application 및 스프링의 설정 등에서 사용할 여러 가지 property를 정의한 파일이다.                                       |
| src/main               | jsp등이 위치한 directory                                                                                               |

## 의존성 관리
- 스프링 부트는 여러 의존성들을 관리해준다.
- `pom.xml`에 parent로 설정되어 있는 `spring-boot-starter-parent`를 살펴보면 관리해주는 의존성들의 version이 나와있는 것을 알 수 있다.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>2.7.9</version>
		<relativePath/> <!-- lookup parent from repository -->
	</parent>
	<groupId>com.example</groupId>
	<artifactId>springboot</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<name>springboot</name>
	<description>Demo project for Spring Boot</description>
	<properties>
		<java.version>1.8</java.version>
	</properties>
```

## @SpringBootApplication

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class SpringbootApplication {

	public static void main(String[] args) {
		SpringApplication.run(SpringbootApplication.class, args);
	}

}
```

> @SpringBootApplication 어노테이션은 스프링 부트에서 설정을 도와준다.<br>이 어노테이션을 제거하고 프로그램을 실행하게 되면, 일반적인 자바 프로그램과 같이 동작하게 된다.<br>해당 어노테이션 덕분에 외부 라이브러리, 내장 서버등이 실행이 가능하다.

<h4>SpringbootApplication 실행</h4>

<img src="https://user-images.githubusercontent.com/63226023/221411738-cd5eeef5-89a7-4099-a1ae-a0add6fff551.png">

- - -

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(excludeFilters = { @Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),
		@Filter(type = FilterType.CUSTOM, classes = AutoConfigurationExcludeFilter.class) })
public @interface SpringBootApplication {
```

- 메인 클래스에 붙어 있는 `@SpringBootApplication`은 자동 설정을 해주기 위한 어노테이션으로 크게 3개가 합쳐진 것이라고 생각할 수 있다.
1. `@SpringBootConfiguration`
2. `@ComponentScan`
3. `@EnableAutoConfiguration`

@SpringBootApplication은 Bean을 2번 등록한다. 처음에 ComponentScan으로 등록하고, EnableAutoConfiguration으로 추가적인 Bean을 읽어서 등록한다.

## @SpringBootConfiguration
- @SpringBootConfiguration은 스프링의 @Configuration을 대체하는 스프링 부트 전용 어노테이션이다.
- 테스트 어노테이션을 사용할 때 이 어노테이션을 찾기 때문에 스프링부트에서는 필수 어노테이션이다.

## @ComponentScan
- @ComponentScan은 해당 패키지에서 @Component 어노테이션을 가진 `Bean` 들을 스캔해서 등록한다.
  - @Component
  - @Configuration
  - @Repository
  - @Service
  - @Controller
  - @RestController
  - ....

## @EnableAutoConfiguration
- 이름에서와 같이 `Bean` 을 등록하는 자바 설정 파일이다.
`spring-boot-autoconfigure/MEFA-INF/spring.factories` 에 정의되어 있는 configuration 대상 클래스들을 빈으로 등록한다.

`spring.factories`를 열어 보게되면 수 많은 자동 설정 조건에 따라 적용되어 Bean들이 생성된다.
```
# Initializers
org.springframework.context.ApplicationContextInitializer=\
org.springframework.boot.autoconfigure.SharedMetadataReaderFactoryContextInitializer,\
org.springframework.boot.autoconfigure.logging.ConditionEvaluationReportLoggingListener

# Application Listeners
org.springframework.context.ApplicationListener=\
org.springframework.boot.autoconfigure.BackgroundPreinitializer
...
```

이 리스트에 있는 클래스들은 `@Configuration` 어노테이션이 없어도 자동으로 Bean으로 등록되어 스프링에서의 해줘야했던 작업들을 자동으로 해준다.

## 내장 서버
스프링 부트의 특징 중 하나는 `톰캣 서버`가 내장되어 있다는 것으로 톰캣 내장 서버는 자동 설정의 일부로, 스프링 부트에서는 톰캣과 서블릿 서버가 자동으로 설정되어 있다.

따라서 스프링 부트 애플리케이션을 실행하면 톰캣이 생성되고 서블릿이 추가가 되면서 애플리케이션이 작동하게 된다.

서블릿 웹 서버 생성 설정 파일인 `org.springframework.boot.autoconfigure.web.servlet.ServletWebServerFactoryAutoConfiguration`을 찾을 수 있다.

<img src="https://user-images.githubusercontent.com/63226023/221413811-6b84f5b7-da5a-4633-8f80-6d847e3c083e.png">
