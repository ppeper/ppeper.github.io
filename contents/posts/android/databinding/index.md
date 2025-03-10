---
title: "안드로이드 DataBinding + LiveData 적용하기"
date: 2022-02-02
update: 2022-02-02
tags:
  - Android
  - DataBinding
  - ACC
series: "Android"
---
<img src="https://user-images.githubusercontent.com/63226023/151594101-266890b7-079a-47c5-9daa-2afbc335ccb7.png">

이전의 포스팅은 `ViewModel에 LiveData`를 추가하였었다.😀
[안드로이드 Jetpack의 LiveData 적용하기](https://ppeper.github.io/android/android-livedata/) 이번에는 ACC 구성요소의 __DataBinding__ 을 LiveData와 함께 사용하여 프로젝트를 더 개선된 형태로 만들어 보려고 한다.
- - - 
# DataBinding(데이터 바인딩)
__데이터 바인딩__ 의 주목적은 __UI 레이아웃의 뷰(xml)와 앱 코드에 저장된 데이터(주요사용->ViewModel 인스턴스)와 연결__ 하는 __간단한 방법__ 을 제공하는 것이다.

> 📍 데이터 바인딩을 통하여 불필요한 코드를 줄여줄수 있다.

데이터 바인딩은 `LiveData 컴포넌트와 같이 사용`될 때 특히 좋다. [LiveData 컴포넌트](https://ppeper.github.io/android/android-livedata/)는 실시간으로 데이터의 변경을 감시하기 때문에 __데이터 바인딩__ 을 사용하여 UI와 앱 코드와 연결을 하게되면 데이터가 자동으로 LiveData 값을 변경할 수도 있다. 데이터 바인딩은 초기의 설정 외에 개발자가 추가로 작성할 코드가 필요 없다는 것도 큰 장점이다.

# 데이터 바인딩 사용하기
데이터 바인딩을 사용하기 위하 초기 작업은 다음과 같다.
- 프로젝트 빌드 구성
- 데이터 바인딩 레이아웃 파일 변환
- 레이아웃 파일 Data 요소 추가
- 바인딩 클래스 구성
- 바인딩 표현식 사용

간단한 예시를 데이터 바인딩을 사용하도록 바꾸면서 각각의 사용법을 알아갈 것이다.

## 1. 프로젝트 빌드 구성
데이터 바인딩을 사용하려면 우선 __build.gradle__(Module: 앱이름.app)에  다음을 추가해야 한다.
```gradle
android {
    .
    .
    .
    buildFeatures {
        dataBinding true
    }
}
```

전의 포스팅까지의 예시에 데이터 바인딩을 추가하기에 앞서 필요없진 코드를 삭제해 준다. 또한 사용자가 Convert 버튼을 클릭하였을때 ViewModel의 함수를 호출하여 환전된 값을 LiveData 변수에 넣어주도록 추가한다. 

```kotlin
class MainViewModel: ViewModel() {
    private val usd_to_eu_rate = 0.74f
    var dollarValue: MutableLiveData<String> = MutableLiveData()
    private var _result: MutableLiveData<Float> = MutableLiveData()
    val result: LiveData<Float> get() = _result

    fun convertValue() {
        dollarValue.let {
            if (!it.value.equals("")) {
                _result.value = it.value?.toFloat()?.times(usd_to_eu_rate)
            } else {
                _result.value = 0f
            }
        }
    }
}
```
## 2. 데이터 바인딩 레이아웃 파일 변환
앱을 구성하는 `UI는 XML 레이아웃 파일`에 포함된다. 데이터 바인딩을 사용하기 위해서는 __기존 XML 레이아웃 파일을 데이터 바인딩 레이아웃 파일로 변환__ 해야 한다. 

> 📍 루트 뷰를 __Layout 컴포넌트__ 로 교체

```xml
<?xml version="1.0" encoding="utf-8"?>
<layout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools">

    <androidx.constraintlayout.widget.ConstraintLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        tools:context=".MainActivity">


        <EditText
            android:id="@+id/dollarText"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_marginTop="100dp"
            android:ems="10"
            android:inputType="numberDecimal"
            app:layout_constraintEnd_toEndOf="parent"
            app:layout_constraintStart_toStartOf="parent"
            app:layout_constraintTop_toTopOf="parent" />
            .
            .
            .
    </androidx.constraintlayout.widget.ConstraintLayout>
</layout>
```

## 3. 레이아웃 파일 Data 요소 추가
데이터 바인딩 레이아웃 파일로 바꿨다면 이제 프로젝트의 UI 컨트롤러(ViewModel이나 Activity 및 Fragment)와 __바인딩__ 되어야 한다. 

이 __클래스의 이름__ 이 __데이터 바인딩 레이아웃 파일에 선언__ 되어야 하고, __클래스의 인스턴스를 참조하는 변수 이름__ 을 설정해야 한다. 이때 data 요소를 추가한다.

> 📍 __data__ 요소에는 __variable__, __import__ 가 포함된다.

```xml
<?xml version="1.0" encoding="utf-8"?>
<layout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools">

    <data>
        <variable
            name="myViewModel"
            type="com.kyonggi.viewmodelexample.MainViewModel" />
    </data>
    .
    .
    .
</layout>
```

예시의 코드를 보면 __MainViewModel 클래스__(이때 클래스는 전체 패키지 이름으로 지정해야한다.) 타입의 __변수이름 myViewModel__ 을 사용한다고 지정하였다고 해석하면 된다.

data 요소중 두 번째는 다른 클래스를 __import__ 할 수 있으며, 이 클래스는 __레이아웃 파일의 어디서든지 바인딩 표현식에서 참조가 가능하다.__ 

## 4. 바인딩 클래스 구성
위의 바인딩 레이아웃 파일의 data 요소에서 참조하는 각 클래스는 __레이아웃 파일의 CamelCase로 변환후 Binding이 붙은 형태__ 로 생성된다.(예시에서 MainViewModelBinding)

이 클래스는 __레이아웃 파일에 지정된 바인딩 정보__ 를 포함하며 이것을 __바인딩되는 객체의 변수와 함수__ 에 연결한다.

바인딩 클래스는 자동으로 생성되지만 __대응되는 데이터 바인딩 레이아웃 파일에 대해 바인딩 클래스의 인스턴스를 생성__ 하는 코드는 생성해 줘야한다. 이때 __DataBindingUtil 클래스__ 를 사용한다.

```kotlin
class MainActivity : AppCompatActivity() {
    private lateinit var binding: ActivityMainBinding
    private lateinit var viewModel: MainViewModel

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        binding = DataBindingUtil.setContentView(this, R.layout.activity_main)
        binding.lifecycleOwner = this
        viewModel = ViewModelProvider(this).get(MainViewModel::class.java)
        // viewModel과 data 요소와 결합
        binding.myViewModel = viewModel

    }
}
```

__DataBindingUtil.setContentView(this, R.layout.activity_main)__ 을 통하여 바인딩 클래스를 생성해준다. ViewModel과 LiveData를 함께 사용하기 위해 __binding.lifecycleOwner = this__ 를 통하여 바인딩 객체가 Activity가 존재하는 동안에 메모리에 남아있고 소멸될때 바인딩 객체도 소멸되도록 생명주기 소유자(lifecycleOwner)를 현재 Activity로 설정해 준다. 마지막으로 __binding.myViewModel = viewModel__ 을 통하여 레이아웃 data 요소의 변수 myViewModel을 ViewModel과 바인딩해준다.

## 5. 바인딩 표현식 사용
바인딩 표현식은 __단방향__ 과 __양방향__ 이 있다. 또한 바인딩의 표현식은 __산술 표현식, 함수 호출, 문자열 결합, 배열 요소 사용, 비교 연산__ 등을 수행 할 수 있다.

단방향 표현식은 __@로 사작하며 중괄호({}) 안에 표현식을 넣는다.__ 예를 들어 data 요소의 변수이름이 viewmodel이고 result 변수를 포함하는 ViewModel 인스턴스라면 다음과 같이 바인딩 표현식을 사용할 수 있다.

```xml
android:text="@{viewmodel.result}"
```
단방향 바인딩은 대응 되는 데이터 모델 값이 변경되면 레이아웃 뷰의 값도 변경되지만 그 반대의 경우는 안된다. 

양방향 표현식은  __@=__ 으로 선언하면 된다. 예시로 EditText 뷰 같은경우는 사용자가 다른 값을 입력하면 이것과 대응되는 데이터 모델의 값이 입력 값으로 변경된다.

> 단향향 바인딩:  __@로 사작하며 중괄호({}) 안에 표현식을 넣는다.__   
> 양방향 바인딩: __@=로 사작하며 중괄호({}) 안에 표현식을 넣는다.__ 

- - -

이제 다시 예시로 가서 바인딩 표현식을 레이아웃 파일에 바인딩 해준다. 첫 번째로 TextView의 ViewModel 의 result 값을 바인딩 해준다.

```xml
        <TextView
            android:id="@+id/resultText"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text='@{safeUnbox(myViewModel.result) == 0.0 ? "Enter value" : String.valueOf(safeUnbox(myViewModel.result))}'
            app:layout_constraintBottom_toBottomOf="parent"
            app:layout_constraintEnd_toEndOf="parent"
            app:layout_constraintStart_toStartOf="parent"
            app:layout_constraintTop_toTopOf="parent" />
```

바인딩 표현식은 result값이 0이면 값을 입력하라고 "Enter value"로 알려주고 0이 아니면 환전된 값을 변환하여 사용자에게 보여준다.

여기서 값이 변경되는것은 __ViewModel의 값은 변경될 필요가 없기 때문에 단방향 바인딩__ 을 사용하였다. 반면에 사용자가 입력을하는 EditText 값은경우 __사용자가 직전에 입력한 값으로 ViewModel의 값이 변경될 수 있도록(장치의 회전이 일어났을때 현재 값을 보여줘야한다.) 양방향 바인딩을 사용한다.__

```xml
        <EditText
            android:id="@+id/dollarText"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_marginTop="100dp"
            android:ems="10"
            android:text="@={myViewModel.dollarValue}"
            android:inputType="numberDecimal"
            app:layout_constraintEnd_toEndOf="parent"
            app:layout_constraintStart_toStartOf="parent"
            app:layout_constraintTop_toTopOf="parent" />
```

마지막으로 Convert 버튼을 누르면 처음에 추가한 __convertValue() 함수__ 를 호출하기 위해 바인딩 해준다.

```xml
        <Button
            android:id="@+id/convertButton"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_marginTop="50dp"
            android:text="Convert"
            android:onClick="@{() -> myViewModel.convertValue()}"
            app:layout_constraintEnd_toEndOf="parent"
            app:layout_constraintStart_toStartOf="parent"
            app:layout_constraintTop_toBottomOf="@+id/resultText" />
```

앱을 실행해보면 "Enter value" 메시지와 올바르게 환전이 되는지를 확인할 수 있다.

<img src="https://user-images.githubusercontent.com/63226023/152167733-87c2d584-9059-46ae-82b4-849834117724.png" width="21.5%"> <img src="https://user-images.githubusercontent.com/63226023/152167779-94bd6551-938e-4a1b-9cf8-34537bcac9fe.png" width="21%"> <img src="https://user-images.githubusercontent.com/63226023/152167810-57480cd8-4dca-482a-b427-82d49c40f82f.png" width="56%">
- - -

# References
- [안드로이드 데이터 결합 라이브러리](https://developer.android.com/topic/libraries/data-binding?hl=ko)