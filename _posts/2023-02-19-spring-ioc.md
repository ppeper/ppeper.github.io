---
title: 스프링 IoC와 컨테이너 알아보기
categories: Spring
tags: ['Java', 'Spring']
header:
    teaser: /assets/teasers/spring-image.png
last_modified_at: 2023-02-19T00:00:00+09:00
---

# IoC(제어의 역전)
[Dependency Injection](https://ppeper.github.io/cs/dependency-Injection/) 에서 보았던 자바에서의 객체 간의 결합도가 강하게 되면 __클래스와 결합된 다른 클래스도 같이 유지보수__ 되어야 할 가능성이 높아지게 된다.

IoC(제어의 역전)은 이름과 같이 제어하는 주체가 바뀌었다는 의미로 __프로그램의 제어 흐름을 개발자가 제어권을 갖지 않고 외부에서 결정__ 하는 것을 말한다.

이러한 IoC의 구현 방법 중 하나가 의존성 주입(Dependency Injection)으로 객체에서 필요로 하는 다른 객체를 외부에서 주입 받아 사용하게 되어 객체간의 결합도를 낮추고 유연성과 재사용성을 높일 수 있다.
- - -
# 스프링 컨테이너
`컨테이너`는 객체의 생성, 사용, 소멸에 해당하는 라이프사이클을 담당하여 이러한 라이프사이클을 기본으로 애플리케이션의 사용에 필요한 주요 기능을 제공한다.

스프링에서는 자바 객체를 `빈(Bean)`이라고 불리며 기존의 개발자가 객체의 생성, 사용, 소멸에 대한 것을 직접 해주었다면 Ioc, DI의 개념이 들어간 __스프링 컨테이너가 이 역할을 대신해 주며, 객체들 간의 의존 관계를 스프링 컨테이너가 런타임 과정에서 결정해 준다.__

## 스프링 컨테이너 종류
스프링 컨테이너에는 `BeanFactory` 와 `ApplicationContext` 가 있다.

### BeanFactory vs ApplicationContext
`BeanFactory`는 빈을 등록, 생성, 조회, 반환하는 기능을 담당하고 `getBean()` 메소드를 통하여 빈을 인스턴스화 할 수 있다.

`ApplicationContext`는 BeanFactory를 확장한 인테페이스로 IoC 컨테이너로 빈을 동록하고 관리하는 기본적인 기능들은 BeanFactory와 동일하다.

둘의 차이점은 __BeanFactory는 빈을 사용할 때(getBean() 호출)마다 빈을 생성__ 하지만 __ApplicationContext는 Context를 초기화 할 시점에 빈을 미리 로드하고 캐시에 저장__ 하기 때문에, BeanFactory보다 빠른 애플리케이션 시작 시간을 제공할 수 있다.

또한 ApplicationContext는 BeanFactory의 모든 기능 이외 다국어 처리, 이벤트 발행 및 구독, AOP(Aspect-Oriented Programming) 등의 기능을 지원한다.
- - -
# 스프링에서 Bean 의존관계
스프링에서 자바 객체(Bean)을 사용하기 위한 방법으로는 크게 3가지가 있다.
1. XML로 빈 설정
2. Annotation으로 빈 설정
3. Java 코드로 빈 설정

## 1. XML 문서이용
- Application에서 사용할 Spring 자원들을 설정하는 파일이다.
- 스프링 컨테이너는 설정파일에 설정된 내용들을 읽어 Application에서 필요한 기능들을 제공한다.
- Root tag는 `<beans>`

빈 객체 생성 및 주입의 기본 설정은 아래와 같다.
> 주입할 객체를 설정파일에 설정
> - &lt;bean&gt; : 스프링 컨테이너가 관리할 빈 객체를 설정한다.
> 기본속성
> - name : 주입 받을 곳에서 호출할 이름을 설정한다.
> - id : 주입 받을 곳에서 호출할 이름을 설정한다 (유일 값이어야한다.)

자바 객체파일에서 필요한 주입 받을 dataSource
```java
public class Some1DaoImpl implements Some1Dao {
    
    private DataSource dataSource;

    // set을 통한 주입
    public void setDataSource(DataSource dataSource) {
        this.dataSource = dataSource;
    }
}

public class Some2DaoImpl implements Some2Dao {
    
    private DataSource dataSource;

    // set을 통한 주입
    public void Some2DaoImpl(DataSource dataSource) {
        this.dataSource = dataSource;
    }
}
```

beans 설정 파일에 `<bean>` 들을 등록
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

	<bean id="ds" class="org.springframework.jdbc.datasource.SimpleDriverDataSource">
		<property name="driverClass" value="com.mysql.cj.jdbc.Driver"></property>
		<property name="url" value="jdbc:mysql://127.0.0.1:3306/mydb"></property>
		<property name="username" value="id"></property>
		<property name="password" value="pw"></property>
	</bean>

        <!-- 라이프 사이클을 Spring이 대신해준다. -->
	<bean id="dao1" class="패키지위치">
		<!-- 위에서 만든 id ds를 setDataSource를 호출하라는 의미 -->
		<property name="dataSource" ref="ds"></property>
	</bean>
	
	<!-- 라이프 사이클을 Spring이 대신해준다. -->
	<bean id="dao2" class="패키지위치">
		<!-- 생성자를 호출 -> 이름은 같을것으로 ref만 설정 -->
		<constructor-arg ref="ds"></constructor-arg>
	</bean>
</beans>
```
`<property>`, `<constructor-arg>` tag등을 통해여 속성, 생성자를 통한 주입을 해 줄 수 있다. 해당하는 빈 객체들은 __ClassPathXmlApplicationContext__ 로 context를 생성하여 설정한 빈 들을 주입 받을 수 있다.

```java
import java.io.IOException;

public class GuestBookMain {

    public static void main(String[] args) throws IOException {
        
        ApplicationContext context = new ClassPathXmlApplicationContext(xml이 있는 패키지 + xml이름.xml);
        
        // bean 설정파일의 id(유일 값)으로 지정한 이름으로 주입
        Some1Dao some1Dao = context.getBean("dao1", Some1DaoImpl.class);
        Some2Dao some1Dao = context.getBean("dao2", Some1DaoImpl.class);
    }
}
```
## 2. Annotation으로 빈 설정
스프링에 프레임워크에서는 __Stereotype annotation__ 을 통하여 컴포넌트 스캔을 할 수 있다. 기본적으로 제공하는 어노테이션은 `@Component`, `@Controller`, `@Service`, `@Repository` 이 있다.

| StereoType   | 적용 대상                                                  |
| ------------ | ---------------------------------------------------------- |
| @Repository | Data Access Layer의 DAO 또는 Repository 클래스에 사용한다. |
| @Service     | Service Layer 클래스에 사용한다.                           |
| @Controller  | Presentation Layer의 MVC에서 컨트롤러 역할을 수행하는 클래스에 사용한다. |
| @Component   | 위의 Layer 구분하기 어려운 일반적인 경우에 사용한다.       |

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Component
public @interface Repository {

	/**
	 * The value may indicate a suggestion for a logical component name,
	 * to be turned into a Spring bean in case of an autodetected component.
	 * @return the suggested component name, if any (or empty String otherwise)
	 */
	@AliasFor(annotation = Component.class)
	String value() default "";

}
```
위의 어노테이션들은 모두 __@Component__ 와 연관 되어 해당하는 어노테이션을 클래스에 명시하면 스프링 컨테이너에서 __컴포넌트 스캔을 통하여 해당 클래스의 객체(Bean)들을 관리해 준다.__

## 빈 의존 관계 설정
스프링의 빈을 명시해 주었으면 관리되고 있는 클래스간의 의존 관계를 연결해 주어야 한다. 스프링에서 빈 의존 관계를 위한 어노테이션을 제공하여  멤버 변수에 직접 정의하는 경우 __어노테이션을 통하여 setter method를 만들지 않아도 된다.__

| Annotation | 설명                                                                                                                                                  |
| ---------- | ----------------------------------------------------------------------------------------------------------------------------------------------------- |
| @Resource  | 멤버변수, setter method에서 사용이 가능하며 __타입__ 에 맞춰서 연결된다.                                                                              |
| @Autowired | 스프링 2.5부터 지원하며 __Spring에서는 사용가능하다.__ <br> 멤버변수, setter, constructor, 일반 method에 사용이 가능하며 __타입__ 에 맞춰서 연결된다. |
| @Inject    | 스프링 3.0부터 지원하며 Framework에 종속적이지 않다. <br> `@Autowired`와 마찬지로 사용이 가능하며 __이름__ 으로 연결된다.                             |

스프링 빈 설정을 어노테이션으로 할 경우 반드시 `component-scan`을 설정 해야 한다.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:context="http://www.springframework.org/schema/context"
	xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
		http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-4.3.xsd">

        <!-- 스캔할 어노테이션이 있는 패키지 -->
	<context:component-scan base-package="패키지 경로"/>
</beans>
```
## 3. Java 코드로 설정
컴포넌트 어노테이션을 사용하지 않고 직접 빈을 등록할 클래스들을 정의하기 위해서는  자바 클래스를 `@Configuration` 으로 설정파일인것을 명시해주어야 한다. 또한 `@ComponentScan(basePackages = {})` 를 통하여 component 스캔 할 패키지를 설정해 줄 수 있다. 자바 코드에서 관리될 빈 객체는 `@Bean` 어노테이션을 통하여 스프링이 관리되어야 할 클래스임을 알 수 있게 해준다.
> @Configuration
> - 설정파일인것을 명시한다.
>
> @ComponentScan(basePackages = {})
> - 컴포넌트 스캔할 패키지를 설정한다.

```java
import javax.sql.DataSource;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;
import org.springframework.jdbc.datasource.SimpleDriverDataSource;

@Configuration
@ComponentScan(basePackages = {"패키지 경로"})
public class ApplicationConfig {
    
    @Bean
    public DataSource dataSource() {
        SimpleDriverDataSource ds = new SimpleDriverDataSource();
        ds.setDriverClass(com.mysql.cj.jdbc.Driver.class);
        ds.setUrl("jdbc:mysql://127.0.0.1:3306/mydb");
        ds.setUsername("id");
        ds.setPassword("pw");
        return ds;
    }
}
```

