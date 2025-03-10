---
title: "안드로이드 Jetpack에 대해"
date: 2022-01-28
update: 2022-01-28
tags:
  - Android
  - Jetpack
series: "Android"
---
<img src="https://user-images.githubusercontent.com/63226023/151514551-8a7c0b07-7e4b-4f1e-8b69-627943968edc.png">

 > 안드로이드 Jetpack은 2018년도에 릴리스가 되어 많은 시간이 지났지만 공부하기전에 이해하기 쉽게 정리를 하고가는것은 공부하는데 매우 효과적이라고 생각하여 조금 늦었지만 Jetpack에 대해서 정리해 보려고 한다.~~(나를 위한 포스팅😅)~~

- - -

# 안드로이드 JetPack이란

<img src="https://user-images.githubusercontent.com/63226023/151520051-09f02d64-7e6a-440c-b688-ee8210acc3a5.png">

안드로이드 Jetpack은 개발자가 __고품질 앱을 손쉽게 개발할 수 있도록 지원하는 라이브러리, 도구, 권장사항__ 등의 모음이다.

구글은 안드로이드 앱을 만드는 특정한 방법을 제공하지 않았던것에 비해 `Jetpack 컴포넌트`를 출시한 후에는 개발자들이 아키텍처 지침의 핵심 원리를 준수하며 앱 개발시 `더 빠르고 쉽게 공통 작업`을 수행할 수 있게 해준다.

> 안드로이드 Jetpack은 앱 개발시 더 빠르고 쉽게 개발하도록 도와주는 라이브러리의 모음이다😀.

이전에는 안드로이드 앱 개발시 하나의 Activity에 UI를 보여주고 로직을 처리하는 코드를 모든 코드를 포함하였었다😅. Jepack이 출시된 이후에 안드로이드 아키텍처 컴포넌트(ACC)의 구글에서 권장하는 아키텍처 패턴을 통하여 각 역할을 분리하여 코드의 가독성과 재 사용성을 높이게 되었다.


# 안드로이드 Jetpack 종류

안드로이드 Jetpack의 구성요소는 기존의 지원 라이브러리(Support library)와 아키텍처 구성 요소를 통합하여 4가지 섹션으로 분류된다. 또한 안드로이드 Jetpack 라이브러리의 네이밍은 `[androidx.*(name)]`으로  구성되어 있다.


<img src="https://user-images.githubusercontent.com/63226023/151522973-e36a254c-9612-41bd-844a-e5fef84e87f4.png">

- - -

# References
- [Android Jetpack](https://developer.android.com/jetpack?hl=ko)
- [Android Developers Blog](https://android-developers.googleblog.com/2018/05/use-android-jetpack-to-accelerate-your.html)