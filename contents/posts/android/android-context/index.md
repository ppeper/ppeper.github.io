---
title: "안드로이드 Context의 개념"
date: 2022-01-17
update: 2022-01-17
tags:
  - Android
  - Context
series: "Android"
---
- - -

# Context??
안드로이드 개발시 `Context`의 사용은 거의 모든 곳에서 사용되며 필자 또한 당연하게 생각하고 사용하였다. 

`Context`를 자세하게 알고 사용하지 못하면 안드로이드 어플리케이션 개발시 메모리 누수가 쉽게 발생할 수 있다고 하여 제대로 정리를 하려고 한다.

안드로이드에서 Context(문맥, 맥락)는 __현재 사용되고 있는 앱에 대한 전역 환경 정보__ 라고 한다.

- Application의 현재의 상태(맥락)을 나타낸다.
- Application, Activity의 정보를 얻기 위해 사용한다.
- Resources, DB, SharedPreferences에 접근할 때 사용된다.
- Context class는 Activity, Application class의 부모 클래스이다.


```kotlin
    Intent(applicationContext,ExampleActivity::class.java)
    Intent(baseContext, ExampleActivity::class.java)
    Intent(application, ExampleActivity::class.java)
    Intent(this@MainActivity, ExampleActivity::class.java)
```

안드로이드 많이쓰이는 `Intent 객체` 에서 파라미터로 Context를 넘겨주는데 이 예시에서도 이렇게 __applicationContext__, __baseContext__, __application__, __this@액티비티__ 가 있다. ~~(그래서 다들 뭐가 다른건데...ㅜㅜ)~~ -> Context에 대해서 제대로 알아보자!!

- - -

# 자주 사용하는 Context의 종류
- __Application Context__: Application에 관한 정보를 담은 Context
- __Activity Context__: Activity 관한 정보를 담은 Context, -> Activity마다 존재한다

## - Application Context
Application Context는 싱글톤 객체로 존재하고 kotlin에서는 `applicationContext`라는 함수로 접근이 가능하다. 이 Context는 어플리케이션 내 생명주기와 바인딩이 되어있다. 

Application Context는 라이프사이클이 현재 Context와 분리된 Context가 필요하거나 활동 범위를 넘어 Context를 전달할 때 사용할 수 있다. -> 이 객체에 Activity Context를 넘겨주면 Application이 실행되는 동안에는 Activity에서 참조가 계속 발생하여 가비지 콜렉팅이 되지않아 __메모리 누수__ 가 일어나게 된다!

한 예시로 Room을 사용하기위해 초기화할때 Context가 필요한대 이때 applicationContext를 사용한다. 그 이유는 Room은 어플리케이션이 실행되는동안 지속적으로 사용되는 __싱글톤__ 객체이기 때문에 `applicationContext`을 사용하는 것이다.

## - Activity Context
Activity Context는 반대로 Activity의 생명주기와 바인딩이 되어있다. 따라서 활동 범위내에서 Context를 넘겨주거나 현재 Context의 생명주기와 연결된 Context가 필요할때 사용된다.

한 예시로 안드로이드에서 Dialog를 사용한다고 하였을때 이는 activity내에서만 사용이 되는 경우일 것이다. 이 경우 Application Context를 사용하면 불필요한 메모리 누수가 일어날 것이다. 따라서 이경우 __Activity Context__ 를 사용하는 것이 좋을 것이다.


 > 정리: __어플리케이션내__ 에서 초기화를 해야하거나 __싱글톤__ 객체를 초기화할때는 `Application Context`를 Activity에서 사용되는 UI 컨트롤러 객체(예시: Toast)들은 Activity Context를 사용하는 것을 권장한다.

- - -

이제 Context에 대해서 알게되었으니 위의 예시에서의 __applicationContext__, __baseContext__, __application__, __this@액티비티__ 를  나누어 보자!!

- __Application Context__
    - applicationContext, application: 현재 활성화된 액티비티만이 아닌 Application 전체에 대한 Context가 필요할때 사용.
- __Activity Context__
    - baseContext: 다른 컨텍스트로부터 어떤 컨텍스트에 접근해야하는경우에 ContextWrapper를 쓴다. ContextWrapper 내부에서 참조 된 Context는 baseContext를 통해 액세스된다.
    - this@: View.getContext()와 같으며 현재 실행되고 있는 View의 Context를 return하는데 보통 현재의 Activity의 Context가 된다.

# Rule of Thumb
대부분의 경우 현재의 스코프에 맞는 Context를 사용하면 된다. 해당 Context 참조가 현재 생명주기에 넘어서지 않는 지만 판단해서 대입하면 된다. Activity, Service를 넘어서 다른 객체에서 Context를 참조해야하는 경우는 Application Context의 사용을 권장한다.

# References

 - [[번역] 안드로이드 Context(컨텍스트)이해하기](https://velog.io/@l2hyunwoo/Context-In-Android)
 - [안드로이드에서 Context란 무엇일까](https://rejrecords.wordpress.com/2015/07/23/%EC%95%88%EB%93%9C%EB%A1%9C%EC%9D%B4%EB%93%9C%EC%97%90%EC%84%9C-context%EB%9E%80-%EB%AC%B4%EC%97%87%EC%9D%BC%EA%B9%8C/)
 - [Understanding Context In Android Application](https://blog.mindorks.com/understanding-context-in-android-application-330913e32514)