---
title: 안드로이드 Jetpack의 ViewModel 사용해보기
categories: Android
tags: ['Android', 'ViewModel', 'ACC']
header:
    teaser: /assets/teasers/android-viewmodel-image.png
last_modified_at: 2022-1-30T00:00:00+09:00
---
<img src="https://user-images.githubusercontent.com/63226023/151594101-266890b7-079a-47c5-9daa-2afbc335ccb7.png">

이번 포스팅은 안드로이드 안드로이드 ACC 구성 요소중 하
나인 `ViewModel`에 대해서 알게된 내용을 정리하려고 한다.😀
- - - 
# ViewModel에 대해
__ViewModel__ 클래스는 UI 관련 데이터만을 저장하고 관리하도록 설계되어 있어 __View__(Activity/Fragment)의 UI 컨트롤러의 과도한 책임을 분담하여 __코드가 복잡해지고 거대해지는것__ 을 방지하고 화면 회전등을 하여도 __데이터를 유지__ 하게 해준다.

 > MVVM 패턴에서 말하던 ViewModel이 이런거구나~😀

❗조심할 점은 일반적으로 MVVM 패턴에서 말하는 __ViewModel__ 과 __ACC-ViewModel__ 은 다르다!!

[안드로이드 공식문서](https://developer.android.com/topic/libraries/architecture/viewmodel)에서도 ViewModel을 설명할때 MVVM 패턴에 대한 언급이 전혀 없다. 따라서 ViewModel과 구분하기 위하여 안드로이드에서는 __ACC-ViewModel__ 이라고 많이 부른다. __ACC-ViewModel__ 을 알아보기전에 둘의 차이점에 대해서 알아보자.

# ViewModel vs ACC-ViewModel
__[MVVM 패턴](https://ppeper.github.io/android/android-acc/)에서 ViewModel__ 을 보게되면 View로 부터 독립적이게 되어 View를 위한 데이터만을 가지고있도록 하는것으로 코드의 유지 보수를 좋게해준다는 장점이 있다. 

__ACC-ViewModel__ 은 안드로이드의 생명 주기를 고려하여 UI 관련 데이터를 저장하고 관리하도록 설계되었다.

<img src="https://user-images.githubusercontent.com/63226023/151693012-5e82e66a-68f0-466c-bb06-6af052697228.png" width="50%">

위의 그림을 보면 ViewModel은 Activity가 최초 생성되고 파괴되기 전까지 생명 주기를 유지하여 데이터의 보존을 하는것을 알 수 있다.

정리하면 __MVVM 패턴에서의 ViewModel은 View에서 필요한 데이터를 바인딩과 데이터의 가공을 처리__ 하기 위한 것이고 __안드로이드 ACC-ViewModel은 생명주기를 고려하여 데이터의 유연한 관리__ 라고 할 수 있을것 같다.

두 ViewModel의 개념을 정리해보니 서로 다른 개념이라는 것을 볼 수 있다. 결론적으로 __ACC-ViewModel__ 을 사용한다고 해서 MVVM 패턴이 되는것이 아니고 __ACC-ViewModel__ 은 안드로이드 앱 개발시 __MVVM 패턴을 쉽게~~(지않다..)~~ 개발할 때 도와주는 역할이다. 

> 📍정리: 안드로이드 개발 편의를 위한 이름만 ViewModel

추후에 공부해볼 ACC 구성요소의 `LiveData`, `dataBinding`등 활용하여 MVVM 패턴에서 말하는 ViewModel로 써 ACC-ViewModel을 사용이 가능하다.

# ACC-ViewModel 사용하기
ViewModel의 예시를 위해서 `activity_main.xml`을 다음과 같이 구상하자.   
(각 Id는 `EditText(Number) -> dollarText`, `TextView -> resultText`, `button -> convertButton`)

<img src="https://user-images.githubusercontent.com/63226023/151698396-82cbeecd-fd77-4eb4-bfbc-06198b2a9f61.png" width="30%">

ViewModel을 생성하여 환전을 하여 보여주는 함수를 만들어 준다.
```kotlin
class MainViewModel: ViewModel() {
    private val usd_to_eu_rate = 0.74f
    private var dollarText = ""
    private var result = 0f

    fun setAmount(value: String) {
        this.dollarText = value
        result = value.toFloat() * usd_to_eu_rate
    }

    fun getResult(): Float {
        return result
    }
}
```
데이터 변경을 관찰하기 위해서는 ViewModel의 참조를 얻어야한다. 이때 __ViewModelProvider 클래스__ 를 사용한다.
```kotlin
val viewModel = ViewModelProvider(this)
```
__ViewModelProvider__ 인스턴스가 생성되면 __get()__ 함수를 호출하여 위에서 만든 `MainViewModel` ViewModel 클래스를 인자로 전달한다.
```kotlin
val viewModel = ViewModelProvider(this).get(MainViewModel::class.java)
```
환전하여 보여줄 코드를 모두 작성한후 간단한 예시를 실행해 본다.
```kotlin
class MainActivity : AppCompatActivity() {
    private lateinit var binding: ActivityMainBinding
    private lateinit var viewModel: MainViewModel

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        binding = ActivityMainBinding.inflate(layoutInflater)
        setContentView(binding.root)
        viewModel = ViewModelProvider(this).get(MainViewModel::class.java)

        with(binding) {
            convertButton.setOnClickListener {
                if (dollarText.text.isNotEmpty()) {
                    viewModel.setAmount(dollarText.text.toString())
                    resultText.text = viewModel.getResult().toString()
                } else {
                    resultText.text = "No Value"
                }
            }

            resultText.text = viewModel.getResult().toString()
        }
    }
}
```
화면의 회전을 하더라도 데이터가 유지되는것을 확인할 수 있다.

<img src="https://user-images.githubusercontent.com/63226023/151699362-56c37016-7e8e-4de1-b21a-62ead07b4d31.png" width="30%"> <img src="https://user-images.githubusercontent.com/63226023/151699371-f4c84168-c820-4b22-aefe-951c7e2abd2d.png" width="60%">


ViewModel의 간단한 예시에서는 생성자의 아무런 인자가 없다. 그래서 별도의 의존성에 대한 고민을 할 필요가 없이 __ViewModelProvider__ 를 사용하여 __ViewModel__ 을 생성하였다.

실제로는 생성자에 아무런 매개변수를 쓰지않고 사용하는 경우가 거의 없을것이고, ViewModel에서 사용자에게 보여줄 UI에 표시할 데이터를 보여주기 위하여 여러 인스턴스가 있을것이다.   
앞으로 차근차근 공부해가며 진정한 __MVVM 패턴에 맞는 ViewModel__ 을 사용하기 위해 공부를 해야겠다.😁

> 📍[안드로이드 Jetpack의 LiveData 적용하기](https://ppeper.github.io/android/android-livedata/)

- - -
# References
- [안드로이드 ViewModel 개요](https://developer.android.com/topic/libraries/architecture/viewmodel?hl=ko)

