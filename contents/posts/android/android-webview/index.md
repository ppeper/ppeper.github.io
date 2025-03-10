---
title: "안드로이드 WebView로 웹과 놀기"
date: 2023-11-20
update: 2023-11-20
tags:
  - Android
  - Webview
series: "Android"
---
# WebView
모바일 앱을 사용하다 보면 웹으로 리다이렉트 하거나 웹 페이지를 보여주는 화면들을 많이 볼 수 있다. 앱에서 웹페이지를 보여줄 수 있는 웹뷰를 사용하기 위해서는 안드로이드에서 __WebView__ 를 사용하여 여러가지 옵션을 설정하고 웹 페이지와 상호작용 할 수 있다.

기본적으로 WebView를 사용하기 위해서는 안드로이드에서 Internet 퍼미션을 설정해야 한다. WebView는 간단하게 xml에서 WebView tag를 추가해 주면 된다.

```xml
<uses-permission android:name="android.permission.INTERNET" />

<WebView
    android:id="@+id/wvContent"
    android:layout_width="0dp"
    android:layout_height="0dp"
    app:layout_constraintBottom_toBottomOf="parent"
    app:layout_constraintEnd_toEndOf="parent"
    app:layout_constraintStart_toStartOf="parent"
    app:layout_constraintTop_toTopOf="parent" />
```

WebView를 사용하기 위한 준비는 간단하게 끝났다. 하지만 결국 WebView에서 모바일의 특정 화면을 보여주거나 다른 인터렉션들의 기능들이 필요할 것 이다. 안드로이드에서는 `WebViewClient`, `WebChromeClient`, `Settings` 을 통해 다양한 옵션들과 기능을 사용할 수 있다.

> WebViewClient: 웹페이지를 로딩할 때 생기는 콜백함수로 구성
> - onPageStarted: 페이지가 처음 로딩될 떄 호출된다.
> - onPageFinished: 페이지 로딩이 끝나면 호출된다.
> - shouldOverridingUrlLoading: 웹뷰에서 URL이 로딩이 될 때 호출된다. 해당 함수로 URL을 앱에서 제어 할수 있다.

> WebChromeClient: 웹페이지에서 일어나는 액션을 관리
> - onCreateWindow: 웹에서 새 창을 열때 호출된다.
> - onCloseWindow: 웹뷰가 창을 닫을 때 호출된다.
> - onProgressChanged: 페이지가 로딩되는 중간에 호출된다.

> Settings: 다양한 웹뷰에 관한 설정 가능 (ex: javaScriptEnabled)

지금까지 살펴본 웹뷰를 간단하게 웹에서 버튼을 눌러 안드로이드에서 Toast 메시지를 보여주는 예시를 통해 알아보자.

```kotlin
class MainActivity : AppCompatActivity() {

    private val binding by lazy {
        ActivityMainBinding.inflate(layoutInflater)
    }

    @SuppressLint("SetJavaScriptEnabled")
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(binding.root)
        binding.wvContent.apply {
            settings.javaScriptEnabled = true // 자바스크립트를 사용하기 위한 설정 값
        }
    }
    private fun toast() {
        Toast.makeText(this, "test", Toast.LENGTH_SHORT).show()
    }
    
}
```

## 🌉 브릿지
웹에서도 다양한 버튼과 이벤트들을 호출해주는 View가 있을 것이다. 해당하는 웹뷰와 웹 페이지 간에 상호 작용을 도와 주는 메커니즘을 __브릿지__ 라고 한다.

WebView에서는 네이티브 코드와 웹 페이지의 JavaScript 코드 간의 통신을 지원하기 위해 브릿지를 사용한다. 브릿지는 `@JavascriptInterface` 어노테이션을 함수에 붙여 생성하고 이에 대한 브릿지 함수를 `addJavascriptInterface` 함수를 통해 브릿지를 설정할 수 있다.

모바일과 웹뷰간에 인터렉션을 처리하기 위해서는  __@JavascriptInterface__ 어노테이션을 통하여 웹 페이지 간에 상호작용을 할 수 있다. 이를 통해 JavaScript 코드에서 안드로이드의 Kotlin 코드를 호출하거나, 반대릐 경우를 호출할 수 있다. 예시로 안드로이드에서 toast 메시지를 띄우기 위해 다음과 같이 코드를 작성해 준다.

```kotlin
class WebBridge(
    private val onClickMessage: () -> Unit
) {

    @JavascriptInterface
    fun showMessage() {
        onClickMessage.invoke()
    }
}
.
.
.
@SuppressLint("SetJavaScriptEnabled")
override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    setContentView(binding.root)
    binding.wvContent.apply {
        settings.javaScriptEnabled = true // 자바스크립트를 사용하기 위한 설정 값
        addJavascriptInterface(WebBride(::toast), "Android")
    }
}
```

해당 코드는 예시로 사용하기 위해 안드로이드의 `assets` 폴더에 `toast.html` 로 넣어 둔다.
```html
<!DOCTYPE html>
<html>
    <head>
        <meta name="viewport" content="initial-scale=1.0">
        <meta charset="utf-8">
    </head>
    <body>
        <input type="button" value="토스트 버튼" onClick="showAndroidToast()"/>
        <script type="text/javascript">
            function showAndroidToast() {
                Android.showMessage();
            }
        </script>
    </body>
</html>
```

위에서 addJavaScriptInterface 함수에서 지정하였던 "Android" 와 함수의 이름과 맞게 호출하면 된다.

<img src="https://github.com/ppeper/Kotlin_Algorithm/assets/63226023/a4bc6aaa-3a3c-4f2c-8c61-d6381b86cf03">

- - -
# References
- [WebView에서 웹 앱 개발](https://developer.android.com/develop/ui/views/layout/webapps/webview?hl=ko)