---
title: 스프링 프레임워크의 등장
categories: Spring
tags: ['Java', 'Spring']
header:
    teaser: /assets/teasers/spring-image.png
last_modified_at: 2022-2-18T00:00:00+09:00
---

# 스프링 등장 배경
<img src="https://user-images.githubusercontent.com/63226023/219701751-ef422546-2a51-4ffa-ad98-b67e761e66ac.png">

스프링 프레임워크는 자바 플랫폼을 위한 오픈 소스 애플리케이션 프레임워크이다. 스프링이 등장하기 이전에는 __EJB (Enterprise JavaBeans)__ 를 포함한 __J2EE(Java 2 Platform, Enterprise Edition)__ 기반의 애플리케이션 개발이 주류였다. 하지만 EJB의 불편한 API와 높은 부하, 복잡한 구성 등의 문제점이 있어 __로드 존슨은 경량화된 프레임워크를 개발__ 하면서 이러한 문제점을 해결하고자 하였다.

2002년 로드 존슨이 출판한 책 "Expert One-on-One J2EE Design and Development" 에서 처음 EJB를 사용하지 않고 엔터프라이즈 애플리케이션을 개발하는 방법을 소개하였다. (지금 스프링의 모태)

이후 AOP, DI와 같은 새로운 프로그래밍 방법론으로 가능해지고 POJO로 선언적은 프로그래밍 모델이 가능해 졌다.

> POJO (Plain Old Java Object)
> - 특정 프레임워크나 기술에 의존적이지 않은 자바 객체이다.
> - 특정 기술에 의존적이지 않아 생산성, 이식성이 향상된다.
> - Plain : component interface를 상속되지 않는 특징 (특정 framework에 종속되지 않는다)
> - Old : EJB 이전의 java class를 의미한다.

# 스프링의 특징
## 경량 컨테이너
- 스프링은 자바객체를 담고 있는 컨테이너이다.
- 스프링 컨테이너에서는 자바 객체 생성과 소멸과 같은 LifeCycle을 관리한다 -> `Bean`으로 불린다.

## DI (Dependency Injection) 패턴
- 스프링은 설정 파일, 어노테이션을 통해서 객채간의 의존 관계를 설정할 수 있다.
- 따라서, 객체는 의존하고 있는 객체를 직접 생성하거나 검색하지 않아도 됨

- [Dependency Injection](https://ppeper.github.io/cs/dependency-Injection/)

## AOP (Aspect Oriented Programming) 지원
`관점`을 기준으로 프로그래밍하는 기법으로 이는 문제를 해결하기 위한 __핵심 관심사항__ 과 __전체에 적용되는 공통 관심사항__ 을 기준으로 프로그래밍 함으로서 공통 모듈을 여러 코드에 쉽게 적용할 수 있도록 한다.

## POJO 지원
- 특정 인터페이스를 구현하거나 또는 클래스를 상속하지 않는 일반 자바 객체 지원.
- 스프링 컨테이너에 저장되는 자바객체는 특정한 인터페이스를 구현 하거나, 클래스 상속 없이도 사용이 가능하다.

## IoC (Inversion of Control - 제어의 반전) 
기존의 자바의 객체 생성 및 의존관계에 있어 모든 제어권은 개발자에게 있었다.

IoC는 __제어의 반전(역전)으로 불리며 말 그대로 이젠 개발자가 위와 같은 제어권을 갖지 않고 외부에서 결정__ 되는 것을 말한다. 이로 인하여 객체의 의존성을 역전시켜 객체간의 결합도를 줄이고 유연한 코드 작성이 가능하게 해준다.

스프링에서도 __객체에 대한 생성과 생명주기를 관리할 수 있는 기능__ 을 제공하고 있는데 이런 이유로 `Spring Container` 또는 `IoC Container`라고 부르기도 한다. 

위에서 `Bean`이라고 불리는 것이 Spring (IoC) 컨테이너가 관리는 자바 객체를 의미하며, 스프링이 __모든 의존성 객체(Bean)들을 만들어 주고 필요한곳에 주입__ 시켜 준다.
- - -


