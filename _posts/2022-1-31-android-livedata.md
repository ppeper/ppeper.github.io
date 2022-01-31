---
title: 안드로이드 Jetpack의 LiveData 적용하기
categories: Android
tags: ['Android', 'LiveData', 'ACC']
header:
    teaser: /assets/teasers/android-livedata-image.png
last_modified_at: 2022-1-31T00:00:00+09:00
---
<img src="https://user-images.githubusercontent.com/63226023/151594101-266890b7-079a-47c5-9daa-2afbc335ccb7.png">

이번 포스팅은 안드로이드 안드로이드 ACC 구성 요소중 하
나인 `LiveData`에 대해서 알게된 내용을 정리하려고 한다.😀
[ViewModel을 사용하여](https://ppeper.github.io/android/android-viewmodel/) 간단한 화폐 변환을 하는 예제에 __LiveData__ 를 추가하여 좀 더 개선해 보려고 한다.
- - - 
# LiveData 핵심 요약

__LiveData__ 는 관찰 가능한(obserable) __데이터 홀더 클래스__ 이다. __LiveData__ 의 인스턴스에 포함된 데이터는 __앱의 다른 객체__ 들이`(Activity나 Fragment와 같은 UI 컨트롤러)` 관찰할 수 있다. 또한 LiveData는 __활동 생명 주기 상태__ 를 인식하여 앱의 __메모리 누수__ 의 발생을 줄여준다.

`Observer` 클래스를 사용하여 __실시간으로 데이터가 변경__ 되는지 감시하고 있다가 UI 컨트롤러(Activity)에게 알려주고, 알림을 받은 UI 컨트롤러는 데이터를 이용하여 UI를 업데이트한다.😮

여기서 `Observer`가 관찰하는 데이터는 `LiveData라는 데이터 홀더 클래스`를 가지고 있는 데이터만 감시를 하게된다.

__LiveData__ 는 LifeCycleOwner가 기본적으로 구현되어 있는 Activity와 Fragment에서 __getLifecycle()__ 함수를 통하여 생명 주기를 가져와 __활성화된 상태__ 에서만 변경사항을 알림을 받아 앞서 언급한 __메모리 누수__ 의 발생을 줄여줄 수 있는것이다.👌
> `Observer` 클래스로 표현되는 __관찰자__ 의 수명주기가 `STARTED` 또는 `RESUMED` 상태일때 __활성화__(수신) ⭕

> 수명주기가 끝나는 순간 __관찰자__ 는 __비 활성화__(수신 거부)❌   
> `Lifecycle` 객체의 상태가 `DESTROYED`로 변경되면 __관찰자를 삭제__ 가 가능. 

## LiveData의 사용의 장점
- 데이터 상태와 UI의 일치 보장
    - `LiveData`는 데이터의 변경시 `Observer` 객체에 알린다. 따라서 `Observer`가 대신 UI를 업데이트하여 __데이터와 UI의 일치를 보장__ 한다.
- 메모리 누수 없음
    - `Observer`는 UI 컨트롤러(Activity/Fragment)의 수명 주기(Lifecycle) 객체에 결합이 되어있으므로 수명 주기가 끝나면 자동으로 삭제된다.
- 중지된 활동으로 인한 비정상 종료 없음
    - `Oberver`의 수명 주기가 __비활성 상태__ 에 있으면 어떤 `LiveData`의 이벤트를 수신하지 않기 때문에 비정상 종료가 없다.
- 수명 주기의 자동화
    - UI 구성 요소는 __데이터를 관찰__ 만 하기 때문에 관찰을 중지나 다시시작을 하지 않는다. 따라서 `LiveData`는 __데이터를 관찰하는 동안__ 관련 수명 주기 상태의 변경을 인식하여 자동으로 관리한다.
- 최신 데이터 유지
    - 수명 주기가 비활성화에서 활성화가 될때는 최신의 데이트를 수신한다.
- 적절한 구성 변경
    - 기기의 회전이 일어나면 기존에는 `savedInstanceState` 를 통하여 데이터를 저장하였다고 복원하는 방식이였지만 `LiveData`를 사용하면 최신의 데이터를 즉시 받을 수 있다.
- 리소스 공유
    - `LiveData` 객체가 한번 시스템 서비스에 연결되면 리소스가 필요한 모든 관찰자(Observer)가 `LiveData` 객체를 볼 수 있다.

- - -

# ViewModel에 LiveData 추가하기
기존의 MainViewModel 클래스에 결과로 보여질 값을 `LiveData`의 사용으로 바꿔준다.   
LiveData의 사용은 __캡슐화__ 를 통하여 접근할 수 있게 해준다.

> 캡슐화💊: 객체의 변수와 함수를 하나로 묶고 실제 구현 내용 일부를 내부에 감추어 외부에서 쉽게 사용하지 못하도록 한다.

## MutableLiveData vs LiveData
`MutableLiveData`는 변경이 가능하고 `LiveData`는 변경이 불가능 하다는 의미이다. 

`ViewModel`의 관점에서 `LiveData`를 보게되면
- ViewModel은 __변경이 불가능한 LiveData__ 객체만 외부 관찰자에게 보여줘야한다.
    - -> View(외부)에서는 LiveData 데이터를 변경하지 못해야 한다.

- `LiveData`를 사용하면 __실시간으로 데이터가 변경__ 을 관찰해야 하지만 변경이 불가능하면 이것 또한 모순이다😂


> 📍 따라서 __public__ 으로 `LiveData`을 만들고 __private__ 으로 `MutableLiveData`를 선언하여 외부에서는 관찰만 하고, 내부에서는 수정을 하는 방식으로 사용을 하게된다.

`LiveData`을 변경을 할때는 `MutableLiveData`의 사용 말고도 다음의 방법이 있다.
- __Room 등의 라이브러리를 사용한다면__ Room의 데이터가 바뀌면 해당 LiveData로 바뀐 Data를 즉시 수정해준다.
- Room 등의 사용이 없다면 `viewModel`에서 __MutableLiveData를 생성하여 LiveData와 연결__ 을 하고, 데이터의 변경은 __MutableLiveData를 사용__ 한다.


```kotlin
// Room 등의 라이브러리를 사용하여 데이터를 직접 할당
val livedata : LiveData<Float> = repository.함수이름

// MutableLiveData를 사용하여 직접 데이터를 가공하여 사용
private val _livedata : MutableLiveData<Float> = MutableLiveData()
val livedata: LiveData<Float> get() = _livedata
```

- - -

## 1. LiveData 추가
앞서 설명한 `LiveData`를 __MainViewModel__ 클래스에 선언해준다.
```kotlin
class MainViewModel: ViewModel() {
    private val usd_to_eu_rate = 0.74f
    private var dollarText = ""
    private var _result: MutableLiveData<Float> = MutableLiveData()
    val result: LiveData<Float> get() = _result

    fun setAmount(value: String) {
        this.dollarText = value
        _result.value = value.toFloat() * usd_to_eu_rate
    }
}
```
## 2. Observer 객체 생성
`LiveData`를 선언하였다면 `Observer` 인터페이스를 구현하는 객체를 생성하여 `LiveData`의 데이터 변경을 관찰할 수 있다.

일반적으로는 `UI 컨트롤러(Activity/Fragment)`에 `Observer` 객체를 만든다. `Observer` 인터페이스는 `LiveData`의 데이터 값이 변경될때 호출되는 __onChange()__ 함수를 구현하면 된다.

```kotlin
// 데이터 관찰을 위한 옵저버 설정 -> onChange() 함수의 구현
val resultObserver = Observer<Float> { result ->
    binding.resultText.text = result.toString()
}
```
## 3. observe() 함수 호출
__observe()__ 함수를 사용하여` LiveData` 객체와 `Observer` 객체를 연결합니다. observe() 메서드는 __LifecycleOwner__ 객체와 __Observer 객체__ 를 파라미터로 받는다. 
```kotlin
// viewModel를 Observer와 연결
viewModel.result.observe(this, resultObserver)
```
이렇게 하면 __ViewModel과 Observer 객체가 연결__ 이되고 __Observer 객체는 LiveData 객체를 (여기선 변수 result)를 구독__ 하여 데이터의 변경사항에 대한 알림을 받는다.

```kotlin
class MainActivity : AppCompatActivity() {
    private lateinit var binding: ActivityMainBinding
    private lateinit var viewModel: MainViewModel

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        binding = ActivityMainBinding.inflate(layoutInflater)
        setContentView(binding.root)
        viewModel = ViewModelProvider(this).get(MainViewModel::class.java)

        // 데이터 관찰을 위한 옵저버 설정
        val resultObserver = Observer<Float> { result ->
            binding.resultText.text = result.toString()
        }
        // viewModel를 Observer와 연결
        viewModel.result.observe(this, resultObserver)

        with(binding) {
            convertButton.setOnClickListener {
                if (dollarText.text.isNotEmpty()) {
                    viewModel.setAmount(dollarText.text.toString())
                } else {
                    resultText.text = "No Value"
                }
            }
        }
    }
}
```

`LiveData`를 사용하여 실시간으로 데이터의 변경을 감지하도록 개선해 보았다. 다음에는 `DataBinding`을 `LiveData`와 같이 사용해보면서 `DataBinding`을 알아보도록 하겠다.

> 📍[DataBinding + LiveData 적용하기](https://ppeper.github.io/android/android-databinding/)
- - -

# References
- [안드로이드 LiveData 개요](https://developer.android.com/topic/libraries/architecture/livedata?hl=ko)
- [LiveData 와 MutableLiveData](https://comoi.io/300)