---
title: 안드로이드 코루틴의 기초 알아보기
categories: kotlin
tags: ['Android', 'Kotlin', 'Coroutine', '비동기']
header:
    teaser: /assets/teasers/kotlin-coroutine-image.png
last_modified_at: 2022-2-14T00:00:00+09:00
---       
<img src="https://user-images.githubusercontent.com/63226023/152948157-588cbe52-ea52-4654-9050-cd98db3a51ba.png">

# 코루틴 개요
[스레드와 코루틴](https://ppeper.github.io/cs/thread-coroutine/)을 알아보면서 코루틴에 대한 개념을 알아보았었다. 스레드와 코루틴은 둘다 __동시성을 보장__  관점에서 보면 비슷한 개념이라고 생각이 든다.

코투린은 실행되다가 __일시 정지(예: 일정 시간 대기)__ 할 때 코틀린 런타임에 의해 __자신이 실행되던 스레드가 다른 코루틴__ 에 할당된다. 이번 포스팅을 통해 __코루틴의 구현에 필요한 요소__ 들을 살펴보면서 대략적인 코루틴이란 무엇인지 알아보려고 한다👍

## 안드로이드 앱의 메인 스레드
안드로이드 앱이 처음 시작되면 런타인 시스템에서 하나의 스레드를 생성하며 이 스레드를 __메인 스레드(Main)__ 라고 한다. 모든 앱 컴포넌트들은 이 __메인 스레드__ 에서 실행이 되며, 주된 역할은 __사용자 인터페이스(UI)__ 를 처리하는 것이다.

__메인 스레드__ 를 사용해서 __시간이 오래걸리는 작업__ 을 수행을 하게되면 __해당 작업이 끝날 때까지 앱 전체가 멈춘 것__ 🚫 처럼 보이게 된다. 이 경우에 안드로이드에서는 __'ANR(애플리케이션 응답 없음)'__ 오류가 발생하게 된다. 이때는 별도의 __백그라운드 스레드__ 에서 수행되어야 메인 스레드가 방해를 받지 않고 다른 작업을 계속하게 해주어야 한다.

코루틴은 이러한 백그라운드 작업을 좀 더 효율적이고 간편하게 개발할 수 있도록 한다.

<img src="https://user-images.githubusercontent.com/63226023/152943012-23be57c1-e7ad-45ad-9d73-2d1d40522198.png" width="30%">

코루틴을 시작하기 전에 간단하게 __메인 스레드__ 가 방해받는 예시를 보고 들어가 보려고한다. 이 예시에서는 `Click을 통하여 textview를 1씩 증가`를 하고 `Download data를 통하여 Log 메시지를 20만번` 보여주게 하려고 한다.

```kotlin
class MainActivity : AppCompatActivity() {
    private var count = 0
    private lateinit var binding: ActivityMainBinding

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        binding = ActivityMainBinding.inflate(layoutInflater)
        setContentView(binding.root)

        binding.btnCount.setOnClickListener {
            binding.tvCount.text = count++.toString()
        }

        binding.btnDownload.setOnClickListener {
            downloadData()
        }
    }

    private fun downloadData() {
        for (i in 1..200000) {
            Log.i("Coroutines", "Download $i in ${Thread.currentThread().name}")
        }
    }
}
```
downloadData 함수에서 200000번 Log를 출력하고 btnCount는 클릭시 1씩 증가가 되어 사용자 인터페이스 UI에게 보여줄 것이다.

<img src="https://user-images.githubusercontent.com/63226023/152945466-305bb24c-f0f0-427d-b210-430ae3c63d4a.png">

Download Data 버튼을 클릭하면 Log창에 메시지가 뜨는것을 볼 수 있다. 

<img src="https://user-images.githubusercontent.com/63226023/152945657-ca178551-6316-4f55-8428-68546ed3cae9.png" width="31%"> <img src="https://user-images.githubusercontent.com/63226023/152945695-995e7004-bffe-499d-aec4-9aa35fd31840.png" width="30%">

이때 Click버튼을 계속 누르게 되면 __앱이 멈춘것처럼(Freezing)__ 있다가 __ANR__ 이 발생하여 앱이 종료되는 것을 볼 수 있다. 위에서 앱의 UI는 __메인 스레드__ 에서 처리한다고 하였는데 이때에 __메인 스레드에서 바쁘게 작업을 하고 있기 때문에__  UI를 처리하는데 문제가 생긴 것이다.

```kotlin
binding.btnDownload.setOnClickListener {
    CoroutineScope(Dispatchers.IO).launch {
        downloadData()
    }
}
```
해당 문제를 해결하기 위해 downloadData를 다음으로 감싼다. 코루틴 관련한 개념은 차차 알아가 보려고 한다.

해당 코드를 추가하고 앱을 실행해보면 Download Data를 클릭을 한 후 Click을 하여도 숫자가 잘 증가하는 것을 볼 수 있다😮

- - -

# 코루틴 사용하기
코루틴의 사용하기 위해서는 dependency를 추가해야 한다.
```xml
dependencies {
    .
    .
    .
    implementation 'org.jetbrains.kotlinx:kotlinx-coroutines-core:1.4.1'
    implementation 'org.jetbrains.kotlinx:kotlinx-coroutines-android:1.5.0'
}
```
코루틴을 사용할 준비가 되었다면 크게 다음과 같은 코루틴의 구성요소로 사용한다.
- 코루틴 범위(Scope)
- 코루틴 디스패처(Dispatchers)
- 코루틴 시작

## 코루틴 범위(Scope)
모든 코루틴은 ~~개별적으로 실행되지 않고~~ 그룹으로 관리될 수 있는 __범위__ 에서 실행이 되어야 한다. 범위에서 실행을 하여 액티비티나 프래그먼트가 소멸될때 이와 관련된 코루틴도 한번에 취소 또는 소멸을 하여 __메모리 누수__ 가 생기지 않게 해준다.

코틀린은 커스텀 범위의 생성뿐만 아니라 내장된 범위도 지원한다.

- __GlobalScope__ : __앱의 전체 생명주기와 같이 실행__ 이 되기 때문에 __앱이 시작될 때 생성되어 종료되어야만 메모리에서 해제__ 된다. 코루틴이 필요가 없어도(예: 액티비티가 종료) 계속 실행될 가능성이 있으므로 안드로이드 앱에서는 사용하지 않는 것을 권장한다.

- __ViewModelScope__ : __ViewModel 컴포넌트를 사용__ 할 때 ViewModel 인스턴스에서 사용하기 위해 제공된다. ViewModel 인스턴스가 소멸될 때 코틀린 런타임에 의해 자동으로 취소된다.

이 외에는 커스텀 범위가 사용된다. 이때에 사용되는 스코프는 __CoroutineScope__ 이다.

- __CoroutineScope__ : 코루틴 라이브러리에서 __기본적으로 제공__ 하는 스코프이다. 액티비티가 종료되거나 클래스의 인스턴스가 소멸되면 메모리에서 해제된다.

```kotlin
binding.btnDownload.setOnClickListener {
    CoroutineScope(Dispatchers.IO).launch {
        downloadData()
    }
}
```
위의 예시를 보면 `CoroutineScope` 를 사용하여 코루틴 실행에 사용될 `Dispatchers.Main` 디스패처를 갖는 __코루틴 범위__ 를 선언하였다. 

## 코루틴 디스패처(Dispatchers)
코틀린은 서로 다른 유형의 __비동기 작업 스레드__ 를 유지 관리한다. 코루틴을 시작할때 이러한 디스패처를 지정해 주어야 한다.
- __Dispatchers.Main__ : __메인 스레드__ 에서 코루틴을 실행한다. UI를 변경할 필요가 있는 코루틴에 적합하며, 경량의 작업을 사용하는데 보편적으로 사용된다.

- __Dispatchers.IO__ : __네트워크, 디스크, 데이터베이스 작업__ 을 수행하는 코루틴에 적합하다.

- __Dispatchers.Default__ : 안드로이드의 __기본 스레드풀을 사용__ 한다. __데이터의 정렬이나 복잡한 연산__ 과 같은 CPU를 많이 사용하는 코루틴에 적합하다.

## 코루틴 시작
코루틴의 범위와 디스패처를 정했으면 코루틴을 시작할 준비가 되었다. 코루틴을 동작하기 위해선 __launch__ 와 __async__ 로 시작을 하면된다.

- __launch__ 
    - 결과를 반환❌
    - Job 객체를 반환한다.
- __async__ 
    - 결과를 반환⭕
    - Deffered 객체를 반환한다.

- - -

# References 
- 안드로이드 스튜디오 Arctic Fox & 프로그래밍                                  