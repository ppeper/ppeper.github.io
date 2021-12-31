---
title: '안드로이드 Lifecycle 제대로 알기'
categories: Android
tags: ['Android', 'Lifecycle']
header:
    teaser: /assets/teasers/android-image.png
last_modified_at: 2022-1-1T00:00:00+09:00
---

- - -
# 안드로이드 Lifecycle?
안드로이드를 처음 공부를 하였을때 가장 놀랐던것이 앱이 회전이 되었을때도 앱을 다시 처음부터 그려준다는것이였다.

사용자들이 어플을 쓰게되면 한 화면에서 여러가지 이벤트가 발생할 수 있다(ex. 사용자가 정보를 입력, 게임을 하던도중 전화가 옴..)

1. 정보를 입력하다가 화면이 회전되었음 -> 많은 정보가 사라진다면?
2. 게임을 하던도중 전화를 받고옴 -> 게임이 처음부터 다시시작..(ㅂㄷ)

이러한 문제를 해결하기위해 안드로이드에서는 Lifecycle이 존재하여 Activity의 상태를 저장하고나 복원할 수 있도록 한다.

> 근래에는 `Jetpack` 안드로이드 아키텍처 컴포넌트로 생명주기 클래스가 도입이 되면서 생명주기를 처리하는 더 좋은 방법을 사용할 수 있게 되었다.

---
## 안드로이드 Lifecycle 함수

이러한 Activity의 상태변화가 일어났을때 특정 동작을 수행할 수 있는 여러 `Callback Method`를 안드로이드 프레임워크에서 제공한다. 


<img src="https://user-images.githubusercontent.com/63226023/147640585-d55f8bbc-a22c-4557-aa53-630b64c356a0.png" width="50%"> -> Activity에서의 한 lifecycle을 보여주는 그림

### onCreate(savedInstanceState: Bundle?)
 - 이 함수는 액티비티 인스턴스가 최초 생성될 때 호출되며, 대부분의 초기화 작업을 하는 데 이상적인 곳이다.

### onRestart()
 - 액티비티가 런타임 시스템에 의해 이전에 중단되었다가 바로 다시 시작될 때 호출된다.

### onStart()
- `onCreate()`나 `onRestart()` 함수가 호출된 후 곧바로 호출된다. 그리고 사용자가 곧 보게 된다고 액티비티에서 알려준다. 만일 액티비티가 액티비티 `스택`의 맨 위로 이동하면 이함수가 호출된 다음에 `onResume()`이 호출된다

### onResume()
- 액티비티가 액티비티 `스택`의 맨 위에 있으며, 사용자와 현재 상호작용하는 액티비티임을 나타낸다.

### onPause()
- 이 함수 호출 다음에는 `onResume()` 또는 `onStop()` 함수 중 하나가 호출된다. 즉 액티비티가 포그라운드로 돌아가는 경우 (계속 실행하기 위해) `onResume()`이 호출되며, (액티비티의 사용자 인터페이스를) 사용자가 볼 수 없게 되면 중단되면서 `onStop()`이 호출된다.

 - 이 함수 내부에서는 앱에서 아직 저장하지 않은 데이터를 저장하는 작업을 하면 된다. 단, `onPause()`는 아주 잠깐 실행되므로 저장 작업을 실행하기에는 시간이 부족할 수 있어 애플리케이션 또는 사용자데이터를 저장하거나, 네트워크 호출을 하거나, 데이터베이스 트랜잭션을 싱행해서는 `안 된다` -> 그 대신, 부하가 큰 종료 작업은 `onStop()` 상태일 때 실행해야 한다.

### onStop()
- 이 함수가 호출될 때는 액티비티가 더 이상 사용자에게 보이지 않는다. 이 함수 호출 다음에는 `onRestart()` 또는 `onDestroy()`가 호출된다. `onStop()` 메서드에서는 앱이 사용자에게 보이지 않는 동안 앱은 필요하지 않은 리소스를 해제하거나 조정해야 한다.

- 또한 `onStop()`을 사용하여 CPU를 비교적 많이 소모하는 종료 작업을 실행해야 한다. 예를 들어 정보를 데이터베이스에 저장할 적절한 시기를 찾지 못했다면 `onStop()` 상태일 때 저장할 수 있다.

### onDestroy()
- 이 함수는 다음과 같은 이유로 액티비티가 곧 소멸될 때 호출된다.

1. 액티비티가 자신의 작업을 완료하고 `finish()` 함수를 호출했을 경우.
2. 메모리가 부족하거나 구성변경(예: 장치의 기기 회전 또는 멀티 윈도우 모드)으로 인해 시스템이 일시적으로 활동을 소멸시키는 경우.

---

# Fragment에서의 Lifecycle
지금까지는 안드로이드 Activity의 생명주기를 보았다.

Fragment에서의 생명주기 함수는 다음이 추가된다.

![image](https://user-images.githubusercontent.com/63226023/147833664-4d80ade8-c6e5-4b8f-a9c3-a2af081246f2.png)

## onAttach()
- 프래그먼트가 액티비티에 지정될 때 호출된다.

- 프래그먼트가 완벽하게 생성된 상태는 아니다.

## onCreateView()
- 프래그먼트의 사용자 인터페이스 레이아웃 뷰 계층을 생성하고 반환하기 위해 호출된다.

## onActivityCreated()
- 프래그먼트와 연관된 액티비티의 `onCreate()` 함수가 실행 완료되면 호출된다.

## onViewStatusRestored()
- 프래그먼트의 저장된 뷰 계층이 복원될 때 호출된다.

---
# 동적 상태를 저장하고 복원
지금까지 이야기한 생명주기 함수와 더불어 액티비티의 동적 상태를 저장하고 복원허가 위하여 만들어진 두 개의 함수가 있다. 

## onRestoreInstanceState(savedInstanceState: Bundle?)
- 상태 정보가 저장되었던 이전 액티비티 인스턴스로부터 다시 생성되어 시작될때 `onStart()` 함수가 호출된 후 곧바로 이 함수가 호출된다.

- `onCreate()`, `onStart()`에서 액티비티의 초기화가 수행된 후에 이전 상태 데이터를 복원해야 할 때 이함수가 호출된다.

## onSaveInstanceState(outState: Bundle?)
- 현재의 동적 상태 데이터가 저장될 수 있도록 액티비티가 소멸되기 전에 호출된다. 

- 이 Bundle 객체는 이후에 액티비티가 다시 시작될 때 `onCreate()`와 `onRestoredInstanceState()` 함수에 전달된다.                                 

> 동적 상태 데이터가 저장될 필요가 있다고 런타임이 판단할 경우에만 이 함수가 호출!

> 이 함수에서는 우리가 보존할 필요가 있는 동적 데이터를 Bundle 객체에 저장하면 된다.

---
# Bundle 클래스?
`onCreate()`, `onRestoredInstanceState()`의 변수에 들어있는 Bundle 클래스는 데이터를 저장하는 역할을 하고 저장된 데이터를 복원한다고 위에서 설명하였다. -> Bundle??

Bundle 클래스는 `키-값 쌍`(Map)으로 구성되는 데이터를 저장하는 역할을 한다. 키는 문자열의 값이며, 키와 연관된 값은 기본형 데이터 값이거나 또는 안드로이드 Parcelable 인터페이스를 구현하는 어떤 객체도 될 수 있다.

- Bundle 클래스의 키값은 Parcelable, Serializable, 기본형(Byte, Short, Long, Float, Double, Char, Boolean)등 포함되어 사용.

---


# References
- [안드로이드 Activity 생명주기](https://developer.android.com/guide/components/activities/activity-lifecycle?hl=ko#kotlin)
- [안드로이드 Fragment](https://developer.android.com/guide/components/fragments)

