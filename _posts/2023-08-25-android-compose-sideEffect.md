---
title: 안드로이드 Side Effect란?
categories: Android
tags: ['Android', 'Compose', 'Side Effect']
header:
    teaser: /assets/teasers/android-compose-side-effect-image.png
last_modified_at: 2023-08-25T00:00:00+09:00
---
<img src="https://github.com/ppeper/Kotlin_Algorithm/assets/63226023/3ee7a814-cc90-42ba-9411-6a20ff29c970">

# Side Effect
Sife Effect(부수 효과)는 컴포지블 외부에서 발생하는 앱의 상태 변경사항을 말한다. Compose에서는 Side Effect를 처리하기 위해 아래의 동작을 실행할 수 있는 `Effect API`를 사용할 수 있다.

## LaunchedEffect

```kotlin
@Composable
fun LaunchedEffect(
	key1 : Any?,
	block : suspend CorountineScope.() -> Unit
)
```

- __LaunchedEffect__: 컴포저블에서 suspend fun을 실행하기 위해 사용된다.
    - 컴포저블에서 시작하면 매개변수로 전달된 블록이 코루틴으로 시작된다.
    - 컴포지션을 __종료하면 실행중인 코루틴이 취소__ 된다.
    - key값이 다른 키로 리컴포지션되면 __기존 코루틴은 취소되고 새 코루틴에서 suspend 함수가 실행__ 된다.

이외에도 LaunchedEffect에서는 여러개의 key값을 매개변수로 받아 재실행할 수 있다.

```kotlin
@Composable
fun LaunchedEffect(
	key1 : Any?,
	key2 : Any?,
	block : suspend CorountineScope.() -> Unit
)

@Composable
fun LaunchedEffect(
	vararg : Any?,
	block : suspend CorountineScope.() -> Unit
)
```

## DisposableEffect

```kotlin
@Composable
fun DisposableEffect(
	key1 : Any?,
	effect : DisposableEffectScope.() -> DisposableEffectResult
) {
	remember(key1) { DisposableEffectImpl(effect) }
}
```

- __DisposableEffect__: 컴포저블이 Dispose될 때 정리되어야 할 Side Effect를 정의하기 위해 사용된다. 

실제로 사용시 재수행 되는것을 결정하는 key값과 함께 `onDispose{}` 블록에서 초기화 로직을 호출한다.

```kotlin
@Composable
fun HomeScreen(
    lifecycleOwner: LifecycleOwner = LocalLifecycleOwner.current,
    onStart: () -> Unit, // Send the 'started' analytics event
    onStop: () -> Unit // Send the 'stopped' analytics event
) {
    // Safely update the current lambdas when a new one is provided
    val currentOnStart by rememberUpdatedState(onStart)
    val currentOnStop by rememberUpdatedState(onStop)

    // If `lifecycleOwner` changes, dispose and reset the effect
    DisposableEffect(lifecycleOwner) {
        // Create an observer that triggers our remembered callbacks
        // for sending analytics events
        val observer = LifecycleEventObserver { _, event ->
            if (event == Lifecycle.Event.ON_START) {
                currentOnStart()
            } else if (event == Lifecycle.Event.ON_STOP) {
                currentOnStop()
            }
        }

        // Add the observer to the lifecycle
        lifecycleOwner.lifecycle.addObserver(observer)

        // When the effect leaves the Composition, remove the observer
        onDispose {
            lifecycleOwner.lifecycle.removeObserver(observer)
        }
    }

    /* Home screen content */
}
```

위의 예시에서 onStart에서 시작하여 onStop에서 정리되어야 하는 경우에 `rememberUpdateState` 를 사용하여 각 매개변수의 onStart, onStop에 대한 state를 저장하고 lifecycle이 바뀔 때마다 새로운 observer가 lifecycle에 붙어 구독하고 `onDispose` 블록을 통하여 컴포저블이 제거될때 observer를 삭제할 수 있다.

## SifeEffect

- __SideEffect__: 컴포저블 State를 Compose에서 관리하지 않는 객체와 공유하기 위해 사용된다.

```kotlin
@Composable
fun BackHandler(
    backDispatcher: OnBackPressedDispatcher,
    enabled: Boolean = true, // Whether back events should be intercepted or not
    onBack: () -> Unit
) {
    /* ... */
    val backCallback = remember { /* ... */ }

    // On every successful composition, update the callback with the `enabled` value
    // to tell `backCallback` whether back events should be intercepted or not
    SideEffect {
        backCallback.isEnabled = enabled
    }

    /* Rest of the code */
}
```

위의 예시에서 BackHandler의 callback을 enable 할지 말지 통신을 하기위해 SideEffect를 사용해서 값을 업데이트한다.

이외에 위의 3가지와 함께 사용할 수 있는 CoroutineScope와 State관련 함수를 제공한다.

- `rememberCoroutineScope`: 컴포저블의 코루틴 스코프를 참조하여 외부에서 실행할 수 있도록 한다.
- `rememberUpdatedState`: LaunchedEffect에서 State 변경시 재실행되지 않아도 되는 State를 정의하기 위해 사용한다.
- `produceState`: Compose State가 아닌 것을 Compose State로 변환하기 위해 사용한다.
- `deriveStateOf`: State를 다른 State로 변환하기 위해 사용한다.
- `snapshotFlow`: 컴포저블의 State를 Flow형태로 변환한다.

- - -
# 정리
Compose에서 새로운 내용들이 많아 적용하고 한번 정리해 볼 시간이 필요하였는데 몰랐던 내용들을 많이 알고 가는 것 같다. 개념들을 알아보면서 실제로 서비스에 적용될 때 파악하고 쓰는 것이 중요할 것 같다.

Compose를 처음 프로젝트에 적용하고 개발할 때 공부하는데 어려움이 많았는데 개념들을 정리해 보면서 꾸준히 공부해 나가야겠다! 🥲

- - -
# References
- [Compose의 부수 효과](https://developer.android.com/jetpack/compose/side-effects?hl=ko)