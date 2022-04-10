---
title: 안드로이드 Hilt에 대해 알아보기
categories: Android
tags: ['Android','Dagger']
header:
    teaser: /assets/teasers/android-hilt-image.png
last_modified_at: 2022-4-10T00:00:00+09:00
---
# Hilt
Hilt란 Google의 Dagger2를 기본으로 만든 의존성 주입 라이브러리이다. 따라서 Dagger2에 대한 Component, Scope를 기본적을 제공하며, 초기 프로젝트 환경 구축 비용을 크게 줄이는 것이 목적이 있다.


```kotlin
// Third Party Library라고 생각
class DataSource {
    fun getData() {
        Log.i("TAG", "데이터 다운로드..")
    }
}
```

Hilt에 대해 알아보기 전에 Dagger2를 사용하여 의존성 주입을 사용한 간단한 예시를 구성해 보면 다음과 같다.

```kotlin
@Module
class DataModule {

    @Provides
    fun providesDataSource(): DataSource {
        return DataSource()
    }
}

@Component(modules = [DataModule::class])
interface DataComponent {
    fun inject(mainActivity: MainActivity)
}

class App: Application() {
    lateinit var dataComponent: DataComponent
    
    override fun onCreate() {
        super.onCreate()
        dataComponent = DaggerDataComponent.builder()
            .build()
    }
}

class MainActivity : AppCompatActivity() {
    @Inject
    lateinit var dataSource: DataSource

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        (application as App).dataComponent
            .inject(this)

        // 사용하기
        dataSource.getData()
    }
}
```

```xml
<application
    android:name=".App"
    .
    .
</application>

```
Dagger2로 적용되어있는 간단한 예시를 Hilt를 알아보면서 바꿔보도록 한다.

# 종속 항목 추가
먼저 `hilt-android-gradle-plugin` 플러그인을 프로젝트의 루트 build.gradle 파일에 추가한다.

```gradle
buildscript {
    ...
    dependencies {
        ...
        classpath 'com.google.dagger:hilt-android-gradle-plugin:2.28-alpha'
    }
}
```

`app/build.gradle` 파일에 종속 항복을 추가한다.

```gradle
plugins {
    id 'kotlin-kapt'
    id 'dagger.hilt.android.plugin'
}

dependencies {
    // Dagger - Hilt
    implementation "com.google.dagger:hilt-android:2.41"
    kapt "com.google.dagger:hilt-android-compiler:2.41"
}
```

Hilt는 자바8 기능을 사용한다. 대부분 자바8을 사용하기 때문에 이미 되어있는 부분일 수 있다.

```gradle
compileOptions {
    sourceCompatibility JavaVersion.VERSION_1_8
    targetCompatibility JavaVersion.VERSION_1_8
}
```

## version 범블비
안드로이드 스튜디오 범블비 버전에선 `build.gradle`과 `settings.gradle`이 변화하였다.

`build.gradle`에 있던 이 부분이 `settings.gradle`로 이동하였다.

```gradle
repositories {
    gradlePluginPortal()
    google()
    mavenCentral()
}
```

`build.gradle` 의 classpath 설정하던 부분이 다음과 같이 변경되었다.
```gradle
plugins {
    id 'com.android.application' version '7.1.1' apply false
    id 'com.android.library' version '7.1.1' apply false
    id 'org.jetbrains.kotlin.android' version '1.6.10' apply false
}
```
Hilt의 2.41버전이 나오면서 classpath는 `build.gradle`에 다음과 같이 넣어주면 된다.
```gradle
plugins {
    id 'com.android.application' version '7.1.1' apply false
    id 'com.android.library' version '7.1.1' apply false
    id 'org.jetbrains.kotlin.android' version '1.6.10' apply false
    id 'com.google.dagger.hilt.android' version '2.41' apply false
}

```
# @HiltAndroidApp
```kotlin
class App: Application() {
    lateinit var dataComponent: DataComponent
    
    override fun onCreate() {
        super.onCreate()
        dataComponent = DaggerDataComponent.builder()
            .build()
    }
}
```
Hilt에서 __@HiltAndroidApp__ 어노테이션을 사용하면 컴파일 타임시 표준 Component 생성에 필요한 클래스들을 초기화한다. 따라서 Hilt를 프로젝트에 적용하기 위해서는 필수적으로 요구되는 과정이다. 

Dagger를 적용하였는 App 클래스를 Hilt로 변경하면 다음과 같이 코드가 줄어든다.
```kotlin
@HiltAndroidApp
class App: Application()
```
```xml
<application
    android:name=".App"
    .
    .
</application>

```
# Component hierachy
기존의 Dagger2는 개발자가 직접 필요한 Component들을 작성하고 상속 관계를 정의하여 사용하였지만, Hilt에서는 Android 화경에서 표준적으로 사용되는 Component들을 기본적으로 제공하고 있다.

또한 Hilt 내부적으로 제공하는 Component들의 전반적인 생명주기 또한 자동으로 관리를 해주기 때문에 초기 DI 적용을 위한 환경을 구축하는데 드는 비용을 줄일수 있다.

<img src="https://user-images.githubusercontent.com/63226023/162617386-fb24e8e4-e8dd-4730-9170-3430f0145c3e.png">

위와 같은 Hilt에서 제공하는 Component들은 아래와 같은 함수 호출시점에 생성되고, Destroy가 된다.

|        생성된 구성요소       |       생성 위치         |        제거 위치        |
| :------------------------ | :--------------------- | :--------------------- |
|    SingletonComponent     | Application#onCreate() | Application#onDestroy()|
| ActivityRetainedComponent |   Activity#onCreate()  |  Activity#onDestroy()  |
|     ActivityComponent     |   Activity#onCreate()  |   Activity#onDestroy() |
|    FragmentComponent      | 	Fragment#onAttach()  |   Fragment#onDestroy() |
|       ViewComponent       |      View#super()      |        제거된 뷰        |
| ViewWithFragmentComponent |      View#super()      |        제거된 뷰        |
|     ServiceComponent      | 	 Service#onCreate()  |   Service#onDestroy()  |

각 컴포넌트들은 생성 시점부터 제거되기까지 `Member Injection`이 가능하고, 각 컴포넌트마다 아래와 같은 생명주기를 갖는다.

- SingletonComponent: Application 전체의 생명주기를 가진다.
- ActivityRetainComponent: `ApplicationComponent`의 하위 컴포넌트로써, Activity의 생명주기를 가진다.
   - Activity의 화면 회전과 같은 작업이 일어났을시 파괴되지 않고 유지된다.
- ActivityComponent: `ActivityRetainComponent`의 하위 컴포넌트로써, Activity의 생명주기를 가진다.
- FragmentComponent: `ActivityComponent`의 하위 컴포넌트로써, Fragment가 Activity에 붙는(onAttach()) 시점에 생성이 되고, Fragment가 파괴되는 시점에 제더된다.
- ViewComponent: `ActivityComponent`의 하위 컴포넌트로써, View의 생명주기를 가진다.
- ViewWithFragmentComponent: `FragmentComponent`의 하위 컴포넌트로써, Fragment의 view 생명주기를 가진다.
- ServiceComponent: `ApplicationComponent`의 하위 컴포넌트로써, Service의 생명주기를 가진다.

위와 같은 표준 Component/Scope들은 Hilt에서는 제공하고 있으며, 새로운 Component를 정의하고 싶다면 `@DefineComponent` 어노테이션을 사용하여 정의가 가능하다.

# @InstallIn
Dagger2에서는 새로운 module을 생성하면, 사용자가 정의한 Component에 module 클래스를 직접 넣어주는 방법을 사용하였다. 
```kotlin
@Component(modules = [DataModule::class])
interface DataComponent {
    fun inject(mainActivity: MainActivity)
}
```
반면에 Hilt에서는 표준적으로 제공하는 Component들이 존재하기 때문에 @InstallIn 어노테이션을 사용하여 표준 Component에 Module을 install 할 수 있다.

Dagger에서 사용하였던 DataComponent를 삭제하고 DataModule에 @InstallIn을 추가한다.

```kotlin
@Module
@InstallIn(SingletonComponent::class)
class DataModule {
    @Provides
    fun providesDataSource(): DataSource {
        return DataSource()
    }
}
```
# @AndroidEntryPoint
```kotlin
class MainActivity : AppCompatActivity() {
    @Inject
    lateinit var dataSource: DataSource

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        (application as App).dataComponent
            .inject(this)

        // 사용하기
        dataSource.getData()
    }
}
```
Dagger2에서는 의존성을 주입받을 대상을 전부 dependency graph에 지정을 해주었지만, Hilt에서는 객체를 주입할 대상에게 @AndroidEntryPoint 어노테이션을 추가만 해주면 된다. @AndroidEntryPoint를 추가할 수 있는 Android Component는 아래와 같다.
- Activity
- Fragment
- View
- Service
- BroadcastReceiver

```kotlin
@AndroidEntryPoint
class MainActivity : AppCompatActivity() {
    @Inject
    lateinit var dataSource: DataSource

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        // 사용하기
        dataSource.getData()
    }
}
```
<img src="https://user-images.githubusercontent.com/63226023/162624070-99438590-aae7-4ae4-b966-5e66119cbaa7.png">

- - -

# References 
- [Hilt를 사용한 종속 항목 삽입](https://developer.android.com/training/dependency-injection/hilt-android?hl=ko)
- [안드로이드-스튜디오-범블비202111-버전이-나오면서-buildgradle과-settingsgradle에-변화](https://open-support.tistory.com/entry/%EC%95%88%EB%93%9C%EB%A1%9C%EC%9D%B4%EB%93%9C-%EC%8A%A4%ED%8A%9C%EB%94%94%EC%98%A4-%EB%B2%94%EB%B8%94%EB%B9%84202111-%EB%B2%84%EC%A0%84%EC%9D%B4-%EB%82%98%EC%98%A4%EB%A9%B4%EC%84%9C-buildgradle%EA%B3%BC-settingsgradle%EC%97%90-%EB%B3%80%ED%99%94)
- [https://hyperconnect.github.io/2020/07/28/android-dagger-hilt.html](https://hyperconnect.github.io/2020/07/28/android-dagger-hilt.html)