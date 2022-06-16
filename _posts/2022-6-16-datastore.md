---
title: 새로운 동료 DataStore 알아보기
categories: Android
tags: ['Datastore', 'Local Data', 'jetpack']
header:
    teaser: /assets/teasers/android-datastore-image.png
last_modified_at: 2022-6-16T00:00:00+09:00
---
<img src="https://user-images.githubusercontent.com/63226023/174076432-24f77ccb-dcb7-45fe-a70d-5d65e4d08f19.png">

```kotlin
class 로컬 데이터 {
    DataStore: 이제 여기는 얼씬도 말라. 알았어?   
    SharedPreferences: ...
}
```

# 로컬 저장
지금까지 안드로이드 로컬에 간단한 데이터를 저장하기 위해 `SharedPreferences`를 사용하였다.   
구글에서는 __DataStore__ 의 사용을 적극 권장하고 있고 ~~(SharedPreferences는 구글 공식 문서에서도 사용가이드가 사라졌다..😨)~~

__Datastore를 사용하면 어떤 좋은점들이 있어서 사용을 이렇게 권고하는 것인가?__ Datastore를 하나씩 알아가 보자!

# 🚀 DataStore
__DataStore__ 는 `Key-Value` 타입으로 구성되어 있는 __Preferences DataStore__ 와 __사용자가 정의한 데이터를 저장__ 할 수 있는 __Proto DataStore__ 가 존재한다.

__Proto DataStore__ 을 사용하기 위해서는 '프로토콜 버퍼'를 이용하여 __스키마를 정의__ 해야한다. 이는 데이터의 타입을 보장과 더불어 SharedPreferences보다 빠르고 단순하다👍
## DataStore의 좋은점?
<img src="https://user-images.githubusercontent.com/63226023/174060195-e04e5a19-9058-4a05-8e1e-74e3ffc56496.png">

[출처] [https://android-developers.googleblog.com/2020/09/prefer-storing-data-with-jetpack.html](https://android-developers.googleblog.com/2020/09/prefer-storing-data-with-jetpack.html)

> 🤔 DataStore가 한눈에도 더 많은 것을 제공해 주는것을 볼 수 있다. 간단하게 요약하자면 다음과 같은 특징이 있다.
>
> - __코루틴 + Flow__ 를 사용하여 __Read/Write 에 대한 비동기 API를 제공__
> - __UI Thread(Main Thread)__ 를 호출해도 안전 __(Dispatcher.IO에서 동작한다)__
> - __RuntimeException 으로부터 안전__

## 직접 사용해 보기
### 라이브러리 추가
`build.gradle(app)`에 아래와 같이 라이브러리를 추가해 준다.
```gradle
// Datastore
implementation 'androidx.datastore:datastore-preferences:1.0.0'
```
### DataStore 생성
```kotlin
class AppDatastoreManager(private val context: Context) : AppDataStore {

    private val Context.datastore: DataStore<Preferences> by preferencesDataStore(
        name = "datastore_name"
    )

    // String 타입 저장 Key 값
    private val stringKey = stringPreferencesKey("key_name")
    // Int 타입 저장 Key 값
    private val intKey = intPreferencesKey("key_name")
    .
    .
    .
}
```
__DataStore__ 을 생성해 주기 위해서는 안드로이드 Context의 확장 프로퍼티로 선언해 주고 DataStore에서 사용할 키 값을 설정해 준다.

__PreferencesKey__ 는 아래와 같은 7가지 타입이 있다.

<img src="https://user-images.githubusercontent.com/63226023/174065589-5a4a0d80-1034-4561-ad8c-e94342fe504f.png">

### 데이터를 Read하는 Flow 생성
`DataStore`에서 데이터를 읽어올 때 __코루틴의 `Flow`__ 를 사용하여 데이터를 __`Flow`__ 객체로 전달한다.

```kotlin
val intValue :Flow<Int> =
    context.datastore.data
        .catch { exception ->
            if (exception is IOException) {
                emit(emptyPreferences())
            } else {
                throw exception
            }
        }.map { preference ->
            preference[intKey] ?: 0
        }
```
`map()` 함수를 사용하여 아까 생성한 __키 값(intKey)에 대응하는 Value__ 를 `Flow` 형태로 가져오도록 한다.

또한 `catch()` 를 사용하여, 데이터 읽어오기에 실패하는 경우 `IOException` 을 처리하여 __emptyPreferences()__ 로 비어있는 값을 전달해준다.

### 데이터를 Write하는 메소드 생성
`DataStore`에 값을 쓸 때는 `edit()`메소드를 이용한다. 또한 값을 쓸때는 __반드시 비동기__ 로 작업이 되야하므로 __`suspend`__ 키워드를 통해 해당 작업이 __코루틴 영역에서 동작__ 할 수 있도록 해준다.

```kotlin
suspend fun setInt(data: Int) {
    context.datastore
        .edit { preferences ->
            preferences[intKey] = data
        }
}
```
### 이제 사용하기
DataStore은 `Singleton`으로 관리되어야 한다. 따라서 Application에서 초기화해주고 사용해보도록 하자.

```kotlin
class App: Application() {
    companion object {
        lateinit var datastore: AppDataStoreManager
    }

    override fun onCreate() {
        super.onCreate()
        initDatastore()
    }

    private fun initDatastore() {
        datastore = AppDatastoreManager(applicationContext)
    }
}
```
<h2>1. 데이터 값 읽고 쓰기</h2>

`DataStore`에서 읽은 데이터를 사용하기 위해서는 __DataStore 클래스에서 선언해 놓은 변수에 접근한 후__ `Flow`객체를 반환 받고 __`collect`__ 함수를 이용하여 값을 읽어온다. `CoroutineScope`에서 수행되어야 한다.
```kotlin
CoroutineScope(Dispatchers.Main).launch {
    App.datastore.intValue.collect { it ->
    // 값을 사용하여 뷰에 적용
    }
}
```
딱 원하는 타이밍에 한 번만 값을 받아와서 사용하고 싶을때는 `first()` 함수를 이용할 수 있다.
```kotlin
CoroutineScope(Dispatchers.Main).launch {
    val intValue = App.datastore.intValue.first()
}
```

> 동기적인 동작이 꼭 필요한 경우 `runBlocking`을 사용할 수 있다.
> ```kotlin
> runBlocking {
>    val intValue = App.datastore.intValue.first()
> }  
> ```
> UI 스레드에서 동기 작업을 실행하면 __ANR 또는 UI 버벅거림__ 이 발생할 수 있기 때문에 __과도한 동작은 금해야한다😨__
> 

<h2>2. 데이터 저장</h2>

`DataStore`에 값을 저장하고 싶을때는 미리 작성해 놓은 함수(setInt())를 사용하면 된다. 이때에 `suspending`으로 지정되어 있기 때문에 __코루틴이나 RxJava를 통해__ 비동기적으로 호출해 준다.
```kotlin
CoroutineScope(Dispatchers.Main).launch {
    App.datastore.setInt(26)
}
```

지금까지 SharedPreferences를 대체하여 사용하기 위한 DataStore를 알아보았다. 자체적으로 코루틴의 Flow를 사용하여 비동기적이고 안전하게 사용이 가능하다는 점을 보고 DataStore를 꾸준히 사용해 보면서 점점 더 큰 장점들을 알아가보면 좋을것 같다.

# References
- [https://developer.android.com/topic/libraries/architecture/datastore?gclid=Cj0KCQjwqKuKBhCxARIsACf4XuHSV6c0dQKCbCAO0rH42Pc-MFbVKxhgf1YRYxu2qf_yPmkeU5m3WfoaAqfKEALw_wcB&gclsrc=aw.ds#kts](https://developer.android.com/topic/libraries/architecture/datastore?gclid=Cj0KCQjwqKuKBhCxARIsACf4XuHSV6c0dQKCbCAO0rH42Pc-MFbVKxhgf1YRYxu2qf_yPmkeU5m3WfoaAqfKEALw_wcB&gclsrc=aw.ds#kts)
- [https://kangmin1012.tistory.com/47](https://kangmin1012.tistory.com/47)