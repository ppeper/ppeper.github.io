---
title: "안드로이드 Compose 수명 주기"
date: 2023-08-22
update: 2023-08-22
tags:
  - Android
  - Compose
  - State
series: "Android"
---
<img src="https://github.com/ppeper/Kotlin_Algorithm/assets/63226023/3ee7a814-cc90-42ba-9411-6a20ff29c970">

# 컴포지션(Composition)
Jetpack Compose는 처음 컴포저블을 실행할때 생성되는 것으로 일종의 트리 구조로 되어 있다. 컴포지션에서 UI를 그리기 위해 호출한 `컴포저블을 추적`한다. 

컴포지션은 초기 컴포지션을 통해서만 생성 되고 리컴포지션을 통해서만 업데이트될 수 있다. 따라서 컴포지션을 수정하는 유일한 방법은 리컴포지션을 통하는 것이다.

> 📍 컴포저블의 수명주기는 __1. 컴포지션 시작__, __2. 0회 이상 리컴포지션__, __3. 컴포지션 종료 이벤트__ 로 정의된다.
안드로이드에서 Compose에서는 `State`를 통해서 관리를 한다.  

<img src="https://github.com/ppeper/Kotlin_Algorithm/assets/63226023/c237ff4e-ead6-49ae-a8e8-11f5c2c36b9e">

- - -

# State

리컴포지션은 일반적으로 `State<T>`가 변경되면 트리게 된다. 즉 Compose에서는 앱의 상태가 변경되면 Jetpack Compose는 `리컴포지션`을 예약하고 변경 되어야 할 UI를 반영하도록 컴포지션을 업데이트 한다.

```kotlin
@Preview
@Composable
fun Greeting() {
    var expanded = false
    Surface(
        modifier = Modifier.padding(vertical = 4.dp),
        color = MaterialTheme.colorScheme.secondary
    ) {
        Row(
            modifier = Modifier
                .padding(24.dp)
                .fillMaxWidth(),
            horizontalArrangement = Arrangement.SpaceBetween
        ) {
            Text(
                modifier = Modifier.align(Alignment.CenterVertically),
                text = "Compose State Expanded"
            )
            Button(
                onClick = { expanded = !expanded },
                modifier = Modifier.align(Alignment.CenterVertically)
            ) {
                Text(if (expanded) "Show Less" else "Show More")
            }
        }
    }
}
```

<img src="https://github.com/ppeper/Kotlin_Algorithm/assets/63226023/0504e579-ea4e-422f-a233-ec5eb5f59c40">

expanded 라는 변수를 선언해서 Text의 내용을 변경하지만, 동작하지 않는다.

상태가 바뀌면 compose가 UI를 업데이트 하지만, 이렇게 선언된 변수는 기본적으로 모니터링 하지 않는다.

> Compose에서는 `mutableStateOf()` 함수를 사용해서 내부 상태를 모니터링하게 할 수 있다.

## mutableStateOf, remember
Composable 내부에 `mutableStateOf()`를 할당하면 동작할 것 같으나, 이 값은 Composable이 다시 만들어질때 false값이 assign된다.

> 상태값을 유지하기 위해서는 `remember`를 사용해서 변경가능한 상태를 기억해야 한다.

```kotlin
@Preview
@Composable
fun Greeting() {
    var expanded = remember { mutableStateOf(false) }
    .
    .
    .
            Button(
                onClick = { expanded.value = !expanded.value },
                modifier = Modifier.align(Alignment.CenterVertically)
            ) {
                Text(if (expanded.value) "Show Less" else "Show More")
            }
}
```
- expanded의 data type은 MutableState<Boolean> 이다.
- 값을변경하려면 `.value` 로 속성에 접근해야 한다.

<img src="https://github.com/ppeper/Kotlin_Algorithm/assets/63226023/4b0a301b-2c34-4acd-8070-232ab3c81e12">

## by를 통한 위임
- immutable 을 위임하기 위해서는 (val)
    - getValue(thisRef: R, property: KProperty<*>)를 제공해야 한다.
- mutable을 위임하기 위해서는 (var)
    - setValue(thisRef: R, property: KProperty<*>, value: T)를 추가로 제공해야 한다.

```kotlin
@Preview
@Composable
fun Greeting() {
    var expanded by remember { mutableStateOf(false) }
    .
    .
    .
            Button(
                onClick = { expanded = !expanded },
                modifier = Modifier.align(Alignment.CenterVertically)
            ) {
                Text(if (expanded) "Show Less" else "Show More")
            }
}
```
## State 유지 - rememberSaveable
일반적인 rememeber 함수는 컴포지션 내에서 컴포저블 객체들이 유지될때만 동작한다. 따라서 화면이 회전과 같이 환경구성이 변경되거나 프로세스가 죽는 경우 Activity가 재시작 되기 때문에 모든 상태를 잃게 된다.

이러한 경우 remember대신에 `rememberSaveable`을 사용하면 상태를 보존할 수 있게 된다.
- - -
# References
- [상태 및 Jetpack Compose](https://developer.android.com/jetpack/compose/state?hl=ko#state-in-composables)