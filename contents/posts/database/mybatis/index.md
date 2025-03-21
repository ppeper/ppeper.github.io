---
title: "Spring + MyBatis 사용하기"
date: 2023-02-25
update: 2023-02-25
tags:
  - Spring
  - MyBatis
series: "MyBatis"
---
기존의 자바에서의 데이터베이스의 연결은 JDBC를 사용하여 Connection과 Statement를 가져와 SQL문을 전송하고 결과값을 받았었다.

이러한 코드를 통해 기존에는 JDBC의 연동과정과 자바 코드안에 SQL문이 들어가기 때문에 코드가 복잡하였다. 이에 대한 편의성을 제공해 주는 __MyBatis__ Framework에 대해서 정리해 보려고 한다.[(MyBatis 시작하기)](https://mybatis.org/mybatis-3/ko/getting-started.html)
- - -
<img src="https://user-images.githubusercontent.com/63226023/218615605-d818077a-1d12-4077-91f3-134bd4481407.png">

# MyBatis
MyBatis는 `Java Object`와 `SQL`문 사이의 자동 Mapping 기능을 지원하는 __ORM (Object Relational Mapping) Framework__ 이다.

- MyBatis는 SQL을 별도의 파일로 분리해서 관리한다.
- Object와 SQL 사이의 `parameter mapping` 작업을 자동으로 해준다.
- SQL을 그대로 이용하면서 도메인 객체나 VO 객체를 중심을 개발이 가능하다.

> MyBatis는 데이터베이스의 연동을 도와주어 SQL문과 분리해 주는 Framework이다.

<h4>MyBatis의 특징</h4>

- 쉬운 접근성과 코드의 간결함
	- 가장 간단한 persistence framework이다
	- JDBC의 모든 기능을 MyBatis가 대부분 제공
	- JDBC 관련 코드를 걷어내어 소스코드가 간결해짐.
	- 수동적인 parameter 설정과 Query 결과에 대한 mapping 구문을 제거
- SQL문과 프로그래밍 코드의 분리
	- SQL에 변경이 있을 때마다 자바 코드를 수정하거나 컴파일 하지 않아도 된다.
- 다양한 프로그래밍 언어로 구현가능
    - JAVA, C#, .NET, Ruby, …

<h4>MyBatis를 사용한 DB Access Architecture</h4>

<img src="https://user-images.githubusercontent.com/63226023/221234211-60dd3f6f-0bbf-417d-b04b-d6600e13a74e.png">

## MyBatis 3의 주요 Component

<h4>MyBatis의 주요 Component의 Data Access</h4>

<img src="https://user-images.githubusercontent.com/63226023/221233890-8679c44f-b977-4c6b-8aa0-8a160f70f84c.png">

| 파일                                        | 설명                                                                                                                                                   |
| ------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------ |
| MyBatis 설정 파일 <br> (mybatis-config.xml) | 데이터데이스의 접속 주소 정보나 객체의 alias, Mapping 파일의 경로 등의 고정된 환경 정부를 설정한다.                                                    |
| SqlSessionFactoryBuilder                    | MyBatis 설정 파일을 바탕으로 SqlSessionFactory를 생성한다.                                                                                             |
| SqlSessionFactory                           | SqlSession을 생성한다.                                                                                                                                 |
| SqlSession                                  | 핵심적인 역할을 하는 Class로 __SQL 실행 및 Transaction 관리를 실행__ 한다.<br>SqlSession 객체는 Thread-safe 하지 않아 thread마다 필요에 따라 생성한다. |
| mapping 파일<br>(member.xml)                | SQL 문과 ORMMapping을 설정한다. |

## 기본 사용
```xml
<dependency>
	<groupId>org.mybatis</groupId>
	<artifactId>mybatis</artifactId>
	<version>x.x.x</version>
</dependency>
```

<h4>Java Config를 통하여 SqlSessionFactory 파일 설정</h4>

```java
import java.io.IOException;
import java.io.Reader;

import org.apache.ibatis.io.Resources;
import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.session.SqlSessionFactoryBuilder;

public class SqlMapConfig {

	private static SqlSessionFactory factory;

	static {
		String resource = "mybatis-config.xml";
		try {
			Reader reader = Resources.getResourceAsReader(resource);
			factory = new SqlSessionFactoryBuilder().build(reader);
		} catch (IOException e) {
			e.printStackTrace();
		}
	}

	public static SqlSession getSqlSession() {
		return factory.openSession();
	}
}
```

SqlSessionFactory를 빌드한 후 `openSession()` 메소드를 통하여 SqlSession을 가져올 수 있다. SqlSession은 데이터베이스에 대해 SQL 명령문을 실행하기 위한 모든 메소드들을 가지고 있다.

```java
try (SqlSession session = SqlMapConfig.getSqlSession()) {
  Blog blog = session.selectOne("org.mybatis.example.BlogMapper.selectBlog", 101);
}
```

# Mapper Interface
Mapper Interface는 mapping 파일에 기재된 SQL을 호출하기 위한 Interface이다.
- Mapper Interface는 SQL을 호출하는 프로그램을 Type Safe하게 기술하기 위해 `MyBatis 3.x` 부터 등장하였다.
- Mapping 파일에 있는 SQL을 __java interface를 통해 호출__ 할 수 있도록 해준다.

Mapper Interface를 사용하지 않았을 경우에는 SQL을 호출하는 프로그램은 SqlSession의 메소드의 인자로 __namespace.(SQL ID)__ 으로 지정 해야한다.

```xml
<mapper namespace="com.test.UserDao">
	<select id="search" parameterType="String">
	...
</mapper>
```

DAO에서 사용할때는 `sqlSession.selectOne("com.test.UserDao.search", userid)` 로 사용

Mapper Interface를 사용하지 않는다면 문자열로 지정하기때문에 오타에 의한 버그가 생기거나, IDE에서 제공하는 code assist를 사용할 수 없다.

<h3>Mapper Interface를 사용했을 경우</h3>

Mapper Interface를 사용할 경우 __UserMapper Interface__ 는 개발자가 작성해 준다.

namespace 속성에는 package를 포함한 Mapper Interface의 이름을 작성하고 SQL ID에는 __mapping하는 method의 이름과 같게 지정해 주어야한다.__

```xml
<mapper namespace="com.test.dao.UserMapper">
	<select id="search" parameterType="String">
	...
</mapper>
```

```java
// UserMapper 인터페이스
public interface UserMapper {
	// 메소드 이름과 id의 이름이 동일하다.
	public UserDto search(userid: String)
}

```

해당 Mapper Interface는 구현할 필요없이 `mapper.search(userid);` 로 사용이 가능하다.
- - -
# MyBatis - Spring
- MyBatis를 Stand alone 형태로 사용하는 경우에는 , `SqlSessionFactory` 객체를 직접 사용한다.
- Spring을 사용하는 경우, Spring Container에 MyBatis 관련 `Bean` 을 등록하여 MyBatis를 사용한다.
- MyBatis를 Spring과 연동하기 위해서는 MyBatis에서 제공하는 Spring 연동 라이브러리가 필요하다.

## dependency 추가하기
Maven 프로젝트에서의 dependency를 추가해준다.

```xml
<dependency>
	<groupId>org.mybatis</groupId>
	<artifactId>mybatis</artifactId>
	<version>x.x.x</version>
</dependency>

<!-- mybatis와 spring 연동 -->
<dependency>
	<groupId>org.mybatis</groupId>
	<artifactId>mybatis-spring</artifactId>
	<version>x.x.x</version>
</dependency>

<!-- spring에서 db와 transaction 처리 -->
<dependency>
	<groupId>org.springframework</groupId>
	<artifactId>spring-tx</artifactId>
	<version>${org.springframework-version}</version>
</dependency>

<!-- database 처리 -->
<dependency>
	<groupId>org.springframework</groupId>
	<artifactId>spring-jdbc</artifactId>
	<version>${org.springframework-version}</version>
</dependency>
```

## DataSource 설정
스프링을 사용하는 경우, 스프링에서 DataSource를 관리하므로 MyBatis 설정파일에서는 일부 설정을 생략한다.

스프링 환경 설정파일(application-context.xml) 에 DataSource를 설정 한다. MyBatis와 스프링을 연동하면 데이터베이스 설정과 트랜잭션 처리는 스프링에서 관리한다.

- 일반 Connection 설정

```xml
<bean id="ds" class="org.springframework.jdbc.datasource.SimpleDriverDataSource" destroy-method="close"> 
	<property name="driverClass" value="com.mysql.cj.jdbc.Driver"></property> 
	<property name="url" value="jdbc:mysql://localhost:3306/mydb?serverTimezone=UTC&amp;useUniCode=yes&amp;characterEncoding=UTF-8"/> 
	<property name="username" value="id"/>
	<property name="password" value="pw"/>
</bean>
```

- Connection Pool 설정
	- 시스템 자원인 Connection을 pool 형태로 관리하여, 연결하고 반납할때의 시간을 절약.
	- 서버당 Connection의 수를 통제하여 전체 시스템 장애로 전파되는것을 방지한다.
	- apache common dbcp2, hikari dbcp 등이 있다.

```xml
<bean id="ds"
	class="org.apache.commons.dbcp2.BasicDataSource"
	destroy-method="close">
	<property name="driverClassName"
		value="com.mysql.cj.jdbc.Driver" />
	<property name="url"
		value="jdbc:mysql://localhost:3306/mydb?serverTimezone=UTC&amp;useUniCode=yes&amp;characterEncoding=UTF-8" />
	<property name="username" value="id" />
	<property name="password" value="pw" />
	<!-- connection pool 크기 설정 -->
	<property name="initialSize" value="10" />
	<property name="maxTotal" value="10" />
	<property name="maxIdle" value="10" />
	<property name="minIdle" value="10" />
	<!-- connection pool에 여유가 없을때 대기시간 -->
	<property name="maxWaitMillis" value="3000" />
</bean>
```

## 트랜잭션 관리자 설정
transactionManager 아이디를 가진 빈은 트랜잭션을 관리하는 객체이다. MyBatis는 JDBC를 그대로 사용하기 때문에 __DataSourceTransactionManager__ 타입의 빈을 사용한다.

`tx:annotation-driven` 요소는 트랜잭션 관리방법을 어노테이션으로 선언하도록 설정하여 스프링은 메소드, 클래스에 __@Transactional__ 이 선언되어 있으면, AOP를 통해 트랜잭션을 처리한다.

```xml
<bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
	<property name="dataSource" ref="ds"/>
</bean>
```

<h4>어노테이션 기반 트랜잭션 설정</h4>

```xml
<tx:annotation-driven transaction-manager="transactionManager"/>
```

## SqlSessionFactoryBean
MyBatis 애플리케이션은 SqlSessionFactory를 중심으로 수행하여 스프링에서 객체를 생성하기 위해서는 빈으로 __SqlSessionFactoryBean__ 를 등록해야 한다.

해당 SqlSessionFactoryBean을 등록할 때, 사용할 DataSource 및 mybatis의 설정파일 정보가 필요하다.

```xml
<bean id="ds"
	class="org.apache.commons.dbcp2.BasicDataSource"
	destroy-method="close">
	...
</bean>
<!-- MyBatis-Spring 설정 -->
<bean id="sqlSessionFactoryBean" class="org.mybatis.spring.SqlSessionFactoryBean">
	<property name="dataSource" ref="ds"/>
	<property name="configLocation" value="classpath:mybatis-config.xml"/>
	<property name="mapperLocations">
		<list>
			<value>classpath:guestbook.xml</value>
			<value>classpath:member.xml</value>
		</list>
	</property>
</bean>
```

해당하는 dataSource, configuration을 설정하고 `mapperLocations` 로 해당하는 SQL문이 있는 mapping파일을 각각 등록해 줄 수 있다.

__classpath:__ 의 경로는 default로 `projectname/src/main/resources/`, `projectname/src/main/java/` 의 두 곳을 의미하고 있다.

<img src="https://user-images.githubusercontent.com/63226023/221350133-e3de69e0-0ad2-4725-b7d1-e76d5fbdd54b.png">

mapper 파일들이 많아질 경우 일일이 다 등록을 해주기에 번거로울 수 있으므로 __mapper scanner__ 를 사용하여 자동으로 등록할 수 있다.

```xml
<!-- bace-package 하위의 모든 매퍼 인터페이스가 자동으로 등록된다. -->
<mybatis-spring:scan base-package="com.test.mapper"/>
```

## MyBatis Configuration (mybatis-config.xml)
스프링을 사용하면 DB 접속정보 및 Mapper 관련 설정은 스프링 빈으로 등록하여 관리하므로 MyBatis 환경설정 파일에서는 스프링에서 관리하지 않는 일부 정보만 설정한다. (ex. typeAlias, typeHandler..)

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration PUBLIC "-//mybatis.org//DTD Config 3.0//EN" 
    "http://mybatis.org/dtd/mybatis-3-config.dtd">
    
<configuration>
    <typeAliases>
        <typeAlias type="com.xxx.model.MemberDto" alias="member" />
    </typeAliases>
</configuration>
```

## Mapper 파일
mapper 파일들에서 위의 typeAlias로 설정한 name으로 사용이 가능하다.

```xml
<mapper namespace="com.xxx.mapper.UserDao">

    <select id="login" parameterType="map" resultType="member">
        select *
        from mydb
        where userid = #{userid} and userpwd = #{userpwd}
    </select>
    
</mapper>
```

<h4>위에서 Mapper Interface에서 보았던 SQL ID는 mapping하는 메소드이름과 동일해야 한다.</h4>

```java
public interface UserDao {
    public MemberDto login(Map<String, String> map) throws SQLException;
}
```

<h4>Mapper가 설정이 완료 되었으면 어노테이션을 통하여 사용하려는 Mapper Interface를 데이터접근 객체와 의존관계를 설정하여 사용한다.</h4>

```java
@Service
public class UserServiceImpl implements UserService {
    
   @Autowired
   private UserDao dao;
    
    @Override
    public MemberDto login(Map<String, String> map) throws Exception {
        if(map.get("userid") == null || map.get("userpwd") == null)
            return null;
       return dao.login(map);
    }
}
```