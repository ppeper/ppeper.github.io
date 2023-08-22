---
title: 안드로이드 Compose 상태 호이스팅
categories: Android
tags: ['Android', 'Compose', 'State Hoisting']
header:
    teaser: /assets/teasers/android-state-hoisting-image.png
last_modified_at: 2023-08-23T00:00:00+09:00
---
<img src="https://github.com/ppeper/Kotlin_Algorithm/assets/63226023/3ee7a814-cc90-42ba-9411-6a20ff29c970">

# Stateful vs Stateless
Compose에서는 `State`의 상태를 트리거하여 리컴포지션을 통해 화면을 갱신한다. 여기서 remember와 mutableStateOf를 써서 객체를 저장하는 컴포저블은 __내부 상태를 생성해서 컴포저블을 Stateful__ 로 만든다.

__Stateless Composable은 상태를 가지지 않는 컴포저블__ 이다. Stateless가 되기위한 쉬운 방법으로는 `상태 호이스팅`을 사용하는 것이다. 즉 상태를 상위 컴포넌트로 이동하여 내부적으로 상태를 가지지 않는 것이다. 상태 호이스팅을 이용하면 불필요한 상태가 중복되는것을 막을 수 있고, 버그 발생도 방지가 가능하다.

하지만 Caller가 컴포저블 함수의 상태를 알 필요가 없을 때는 상태 호이스팅 할 필요가 없다.

```kotlin
@Composable
fun ChatBubble(
    message: Message
) {
    var showDetails by rememberSaveable { mutableStateOf(false) } // Define the UI element expanded state

    ClickableText(
        text = message.content,
        onClick = { showDetails = !showDetails } // Apply simple UI logic
    )

    if (showDetails) {
        Text(message.timestamp)
    }
}
```

위에서 `showDetails`는 해당 컴포저블 UI 요소의 내부 상태이다. 이 변수는 이 컴포저블에서만 읽고 수정이 되기 때문에 이러한 경우에는 상태를 호이스팅해도 별 다른 이득이 없기 때문에 내부에 유지할 수 있다.

## 상태 호이스팅

```kotlin
@Preview
@Composable
fun ButtonEx() {
    var count by remember { mutableStateOf(0) }
    Button(
        onClick = { count += 1 },
        contentPadding = PaddingValues(16.dp)
    ) {
        Text(text = "Count: $count")
    }
}

```
<img src="https://github.com/ppeper/Kotlin_Algorithm/assets/63226023/199a8c1b-ff11-444c-bd31-73f86fc49517" width="30%">

버튼을 클릭하면 count값이 증가하여 Button에 보여주는 컴포저블 함수가 있다. 이 함수는 내부적으로 count를 가지고 있어 Stateful하게 되어있다.

상태를 끌어올리기 위해서는 상태를 두 변수로 나누는 방식으로 끌어올리게 된다.

- __value: T__ (값)
- __onValueChange: (T) -> Unit__ (변경하도록 요청하는 함수)

```kotlin
@Composable
fun ButtonEx(
    count: Int,
    onValueChange: (Int) -> Unit
) {
    Button(
        onClick = { onValueChange(count) },
        contentPadding = PaddingValues(16.dp)
    ) {
        Text(text = "Count: $count")
    }
}

@Preview
@Composable
fun ButtonStateless() {
    var count by remember { mutableStateOf(0) }
    ButtonEx(
        count = count,
        onValueChange = { count = it + 1}
    )
    // 마지막 매개변수에 들어간 함수는 람다로 가능
    ButtonEx(
        count = count,
    ) {
        count = it + 1
    }
}
```

<p align="center"><img src="https://github.com/ppeper/Kotlin_Algorithm/assets/63226023/03de1614-ea05-4198-b2c3-e327ba10c9b8" width="70%"></p>

위에서 내부에 있던 상태를 외부로 이동하고 ButtonEx 컴포저블 함수에서는 이벤트와 값만을 받아 Stateless하게 바뀐 것을 볼 수 있다.


- - -
# References
- [상태를 호이스팅할 대상 위치](https://developer.android.com/jetpack/compose/state-hoisting?hl=ko)