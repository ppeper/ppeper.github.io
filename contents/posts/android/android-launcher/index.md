---
title: "안드로이드에서 Launcher를 통한 결과 전달"
date: 2023-02-27
update: 2023-02-27
tags:
  - Android
  - registForActivityResult
series: "Android"
---
# 기존의 startActivityForResult
안드로이드에서 일반적으로 Activity를 전환하기 위해서는 __startActivity()__ 를 사용하지만 화면 전환 이후에 해당 Activity에서의 결과값을 전달 받기 위해서는 __startActivityForResult()__ 과 __onActivityResult()__ 를 사용 하였다.

```kotlin
// 두 번째 인자인 requestCode는 어떤 Activity인지 식별해주는 값
startActivityForResult(Intent(this@FromActivity, ToActivity.class.java), 101)
```

<h4>setResult()</h4>
해당하는 ToActivity의 종료 시점에 작성하여 Activity가 종료되면 해당 메서드를 통해 호출한 Activity에 데이터를 되돌려 줄 수 있다.

```kotlin
// 호출당한 ToActivity
val intent = Intent()
intent.putExtra("result", "some data");
setResult(RESULT_OK, intent)
finish()
```

<h4>onActivityResult()</h4>
```kotlin

// 결과를 받는 FromActivity
override fun onActivityResult(requestCode: Int, resultCode: Int, data: Intent?) {
    super.onActivityResult(requestCode, resultCode, data)
    if (requestCode == 101) {
        if (resultCode == Activity.RESULT_OK) {
            val result = data?.getStringExtra("result")
            ...
        }
    }
}
```

# 새로운 API

<img src="https://user-images.githubusercontent.com/63226023/223721056-cb480e59-aa90-4b29-a149-633276f74475.png">

해당하는 startActivityForResult API는 2020년 5월에 deprecated 되었다고 한다. 구글에서는 새로운 AndroidX Activity와 Fragment에 도입된 __Activity Result API__ 사용을 적극 권장한다.

<h4>deprecated 된 이유와 새로운 API</h4>
안드로이드에서 메모리 부족으로 인해 프로세스와 Activity가 사라질 수 있다. 기존의 방식은 같은 곳에서 `startActivityForResult`와 `onActivityResult`에서 콜백을 처리해야 했다. 

만약 Activity가 종료되고 다시 만들어 진다면 Activity에게 Result를 기다린다고 다시 알려주어야 한다. 따라서 호출과 콜백을 따로 분리해서 만들어 주어야 하는데 이에 대한 역할을 해주는 것이 __ActivityResultLauncher__ 이다.

`registActivityForResult()` 메소드는 ActivityResultContract 및 ActivityResultCallback 을 가져와서 다른 활동을 실행하는 데 사용할 ActivityResultLauncher를 반환해 준다.

registActivityForResult()는 콜백을 등록해 주는 역할을 해주어 Activity가 종료되었다가 다시 생성이 되어도 결과를 기다리고 있다고 알려줄 수 있다.

> 기존의 방식 
> - A -> B를 호출하다 A가 종료된다.
> - B에서는 setResult()로 결과값을 반환한다.
> - 다시 돌아와서 보니 A가 종료 되었다가 다시 생성되어 결과값을 요청한지 모름
>
> 새로운 API
> - A -> B를 호출하다 A가 종료된다.
> - B에서는 setResult()로 결과값을 반환한다.
> - A가 다시 생성되도 registForActivityResult() 메소드가 다시 콜백을 등록해 주기 떄문에 결과값을 받아 올 수 있다.

```kotlin
// 호출하는 부분 기존
startActivityForResult(Intent(this@FromActivity, ToActivity.class.java), 101)
// 새로운 API -> launch() 로 시작
result.launch(Intent(this@FromActivity, ToActivity::class.java))

// Contract 및 결과를 받기 위한 Callback 등록 
private val result = registerForActivityResult(
    ActivityResultContracts.StartActivityForResult()) {
    if (it.resultCode == Activity.RESULT_OK) {
        val intent = it.data
        val data = intent?.getStringExtra("result")
        ...
    }
}
```

기존에서와 다르게 result라는 __ActivityResultLauncher__ 를 생성하여 콜백을 등록한다. 함수의 첫 번째 인자인 Contract는 현재 결과를 받을 Activity를 실행하는 함수인 `ActivityResultContracts.StartActivityForResult()`를 등록한다. launch()를 통하여 이동한 Activity에서는 동일하게 __setResult()__ 함수를 그대로 사용하면 된다.

>📍requestCode가 필요없는 이유
> - 원하는 Activity의 요청 마다 registForAcitivtyResult를 실행하여 콜백을 분리하였기 때문에 기존처럼 Acitivity를 구분할 필요가 없어졌다.

---
# References
- [https://developer.android.com/training/basics/intents/result?hl=ko](https://developer.android.com/training/basics/intents/result?hl=ko)