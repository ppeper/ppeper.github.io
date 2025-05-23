---
title: "Kotlin Coroutines Flow 맛보기"
date: 2022-07-16
update: 2022-07-16
tags:
  - Kotlin
  - Flow
  - Coroutines
series: "Kotlin"
---
`youtube: https://www.youtube.com/watch?v=fSB6_KE95bU`

`ReactiveX`은 반응형 프로그래밍으로 비동기 프로그래밍을 구현하기 위하여 많이 사용이 되어 한번 학습을 해봐야 겠다는 생각이 들었다. Coroutine을 학습하는 도중 ReacticeX와 같은 기능을 할 수 있는 __비동기 스트림__ `Flow`를 제공하고 있어 ReactiveX 이전에 먼저 Flow에 기본적인 개념에 대해 알아보려고 한다.
- - -
# Flow란
코루틴에서 Flow(흐름)은 단일 값만 반환하던 Suspend 함수와는 다르게 __여러 값을 순차적__ 으로 내보낼 수 있다. 또한 반응형 프로그래밍으로 실시간으로 업데이트 하는 상황에서 더욱 코드를 효과적으로 사용할 수 있다. 

<img src="https://user-images.githubusercontent.com/63226023/178661246-540be0cf-a252-4666-9204-ab953441fbfa.png">

코루틴에서 데이터 스트림을 구현하기 위해서의 구성요소는 다음과 같다.

- __Producer(생산자)__
- __Intermediary(중간 연산자) - 선택사항__
- __Consumer(소비자)__

## Producer(생산자)
먼저 생산자에서는 데이터를 발행하기 위하여 __`flow {}` 코루틴 블록(빌더)을 생성한 후 내부에서 `emit()`을 통하여 데이터를 생성한다.__ 또한 `flow {}` 블록은 suspend 함수이므로 delay를 호출할 수 있다.
```kotlin
fun flowSomething(): Flow<Int> = flow {
    repeat(10) {
        emit(it) // 0 1 2 3..9
        delay(100L) // 100ms
    }
}
```

### Flow 빌더
- 앞서 `flow {}`를 사용하는 것이 가장 기본적인 플로우 빌더이다.
- `emit()` 이외에 __`asFlow()`__ 를 통하여 Collection 및 Sequence를 Flow로 변환 할 수 있다.

```kotlin
// Int 배열    
(6..10).asFlow().collect { value ->
    println(value)
}
```
 
## Intermediary(중간 연산자)
생산자에서 데이터를 발행을 하였다면 중간 연잔자는 __생성된 데이터를 수정__ 할 수 있다.
대표적으로 코틀린의 컬렉션의 함수와 같이 대표적으로 __map(데이터를 원하는 형태로 변환), filter(데이터 필터링), onEach(데이터를 변경후 수행한 결과를 반환)__ 등이 있다.
```kotlin
// Filter 
flowSomethings().filter {
    // 짝수만 필터링
    it % 2 == 0
}
// map
flowSomethings().map {
    // 각 데이터를 2배씩
    it * 2
}
.
.
```
## Consumer(소비자)
마지막으로 생산자에서 데이터를 발행하고, 중간 연산자(선택)에서 데이터를 가공하였다면 소비자에서는 __`collect()`를 이용하여 전달된 데이터를 소비할 수 있다.__
```kotlin
fun main() = runBlocking {
    flowSomething().map {
        it * 2
    }.collect { value ->
        println(value)
    }
}
// 0 2 4 6 8 .. 18
```

# References
- [https://developer.android.com/kotlin/flow?hl=ko](https://developer.android.com/kotlin/flow?hl=ko)