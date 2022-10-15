---
title: 안드로이드 runOnUiThread?
categories: Android
tags: ['Android', 'Thread', 'runOnUiThread']
header:
    teaser: /assets/teasers/android-runOnUiThread-image.png
last_modified_at: 2022-10-16T00:00:00+09:00
---
# 안드로이드 Thread

안드로이드에서는 Main Thread와 Sub Thread가 동시에 자원에 접근하여 생기는 동기화 이슈를 발생 할 수 있기 때문에 __메인 스레드(UI 스레드)__ 에서만 뷰의 값을 바꿀 수 있는 __싱글 스레드로 동작__ 한다.

<p align="center"><img src="https://user-images.githubusercontent.com/63226023/195595735-1577e824-1979-410c-860c-7b0a01fc54da.png" width="70%"></p>

> -> 위와 같이 멀티 스레드 환경에서 동시에 스레드가 UI를 갱신을 하여고 한다면 `TextView`가 어떠한 값으로 변경될지 알 수 없다. 

## 싱글 스레드에서 중요한 점
싱글 스레드로 동작하는 안드로이드에서 중요한 점은 따라서 아래와 같이 조심해야 하는 포인트가 있다.
> __❗메인 스레드(UI 스레드) 를 오랜시간 블로킹하면 안된다__
> 
> -> 메인 스레드를 블로킹한다는 의미는 사용자가 보고 있는 UI가 멈춘다는 의미와 같다. 안드로이드에서는 메인 스레드를 블로킹하면 ANR(Application Not Responding)이 발생하여 강제 종료가 된다.

## 메인 스레드에서는 무거운 작업을 피하자
안드로이드에서 메인 스레드를 블로킹하지 않기 위해서 다른 스레드에게 사진을 서버에서 다운 받는 작업을 한다고 하자. 해당하는 사진은 결국 UI에서 보여주기 위해 __메인 스레드에서 처리를 해줘야 할 것이다.__

-> 이때 스레드간에 통신을 위해서는 __`Handler`와 `Looper`__ 를 통해서 처리 작업을 
바꿔주어야 한다.
- - -
# Handler & Looper

> 스레드가 시작 되면 스레드는 `Handler`, `Looper`, `MessagingQueue` 를 하나씩 가지고 있다. 각자의 기능은 스레드간의 통신을 위하여 사용된다.

## Looper
하나의 스레드는 단 하나의 `Looper`를 가지며, 해당 `Looper`는 오직 하나의 스레드를 담당한다.

`Looper`의 내부에는 `MessageQueue`라는 것이 존재한다. 이름과 같이 MessageQueue는 __스레드가 처리 해야할 일들이__  `Message`__의 형태로 들어가 있는 FIFO 구조로 동작하는 친구__ 이다. `Looper`는 MessageQueue가 비어있으면 아무 행동을 하지 않다가 Message가 들어오면 적절한 `Handler`에게 전달하는 역할을 한다.

> __Message?__
> - `Message` 객체는 스레드가 처리하는 작업이라고 생각할 수 있다. MessageQueue 에서 이러한 작업을 넣어주거나 꺼내어 적절한 __Handler__ 에게 전달한다.
> - Message 객체는 `Message`와 `Runnable` 두개로 나누어 진다.

종합적으로 `Looper`는 __MessageQueue에 있는 메시지를 꺼내어 해당하는 `Handler` 에게 전달하는 역할__ 을 한다.

## Handler
`Handler` 는 이름과 비슷하게 어떠한 것을 다루는 일을 한다. 구체적으로 하는 일은 `Looper` __라는 친구의 MessagingQueue에 어떠한 값을 넣거나__ `Looper`에서 __MessagingQueue의 특정 메시지를 주면 이를 처리__ 한다.

### 1. Looper에게 메시지 전달
Looper는 `Message`와 `Runnable` 객체를 담을 수 있다.
> __sendMessage()__
>   - MessageQueue에 `Message` 객체를 담을 수 있다.
>
> __post()__
>   - post 메소드를 통하여 `Runnable` 객체를 담을 수 있다.

### 2. Looper에게서 메시지를 받음
Looper의 Message Queue에서 `(something)`을 꺼냈을때 아래와 같이 구분이 된다.
> __Message__
>    - 해당 메시지의 내부에 있는 `Handler`가 가지고 있는 `handleMessage()`를 호출하여 `Handler`가 메시지를 전달 받을 수 있다.
>
> __Runnable__
>    - `Runnable`의 `run()` 메소드를 통하여 작업을 실행할 수 있다.

- - -

# 정리해 보기
<p align="center"><img src="https://user-images.githubusercontent.com/63226023/195654013-d526bca7-1c79-495f-bc6e-8976ebc5363a.png" width="50%"></p>

> 1. 처음 스레드를 생성을 하여 `(something)`을 처리하고 UI를 갱신하기 위해서는 __메인 스레드(UI 스레드)__ 로 전달해야 한다.
> 2. 특정 스레드 `Handler`의 `sendMessage()` 또는 `post()` 메소드를 호출하여 메인 스레드의 `Looper`의 MessageQueue에 메시지를 전달한다.
> 3. 해당 스레드의 `Looper`가 MessageQueue에서 `Handler`에게 메시지를 하나씩 전달한다.
> 4. `Handler`는 `handleMessage()`를 통해 받은 메시지를 처리한다.

# 안드로이드에서 적접 보기
```kotlin
class MainActivity : AppCompatActivity() {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        // Other Thread
        val mThread = Thread { // 익명 객체 구현
            // 여기서 UI 작업을 수행하면 Exception 발생
        }
        mThread.start()
    }
}
```
안드로이드의 UI관련한 작업은 __메인 스레드__ 에서 동작한다고 하였다. 따라서 다른 스레드에서 UI 관련 작업을 하려고 하면 Exception이 발생한다.

앞서 말한 대로 `Handler`를 통하여 __메시지를 메인 스레드의 MessageQueue에 전달__ 해 주어야 한다.

안드로이드에서는 __메인 스레드__ 가 가지고 있는 `Looper`를 명시하는 `getMainLooper()`를 제공한다.

```kotlin
class MainActivity : AppCompatActivity() {
    private var mHandler: Handler? = null
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        // 명시적으로 메인 스레드의 Looper를 생성
        mHandler = Handler(Looper.getMainLooper())
        // Other Thread
        val mThread = Thread { // 익명 객체 구현
            // 여기서 UI 작업을 수행하면 Exception 발생
            mHandler!!.post {
                // UI 작업 가능.
            }
        }
        mThread.start()
    }
}
```
서브 스레드에서의 작업을 메인 스레드의 Looper의 `mHandler!!.post` 를 통하여 메인 스레드에서 UI 작업을 할 수 있다.

# runOnUiThread는 그래서?

<img src="https://user-images.githubusercontent.com/63226023/195632360-67bc0531-85bd-4182-943e-0889dc703619.png">

안드로이드 `runOnUiThread`는 `Activity` 클래스에서 제공되는 메소드이다. Android developer에서는 `Runnable` 객체를 메인 스레드에서 실행하도록 만드는 메소드로 __현재 스레드가 메인 스레드이면 `Runnable` 객체의 `run()` 메소드를 직접 실행__ 을 하고 아니라면 `Handler`에게 `post()` 메소드를 통하여 메인 스레드로 이벤트 큐를 발송한다.

```java
public final void runOnUiThread(Runnable action) {
    if (Thread.currentThread() != mUiThread) {
        mHandler.post(action);
    } else {
        action.run();
    }
}
```

따라서 `runOnUiThread()`를 사용하면 메인 스레드가 아닐때만 `Handler`는 `post()`를 통하여 이벤트를 발생시키기 때문에 좀 더 효율적이라고 할 수 있다.

```kotlin
class MainActivity : AppCompatActivity() {
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        // Other Thread
        val mThread = Thread { // 익명 객체 구현
            // 여기서 UI 작업을 수행하면 Exception 발생
            runOnUiThread {
                // UI 작업 가능.
                // UI 스레드가 아니라면 내부적으로 handler.post() 호출
            }
        }
        mThread.start()
    }
}
```
- - -
# References
- [https://hungseong.tistory.com/26](https://hungseong.tistory.com/26)