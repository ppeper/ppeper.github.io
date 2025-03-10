---
title: "안드로이드 Dagger2 좀 더 알아가기"
date: 2022-04-08
update: 2022-04-08
tags:
  - Android
  - Dagger
series: "Android"
---
# Custom Application
저번 포스팅에서 Dagger의 Inject, Module, Component에 대해서 알아보고 SmartPhone 클래스에 적용시켜 보았다.

```kotlin
class MainActivity : AppCompatActivity() {
    // Field Injection
    @Inject
    lateinit var smartPhone: SmartPhone
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        DaggerSmartPhoneComponent.create()
            .inject(this)
            
        smartPhone.start()
    }
}
```
불변하는 데이터나 전역으로 사용되는 object 같은 경우 __모든 Component__ 에 공유가 되어 사용되야 할 것이다 (ex) Room Database, SharedPreference..)

위의 예시에서의 SmartPhone 컴포넌트가 필요한 Activity/Fragment가 10개라면 `DaggerSmartPhoneComponent.create().inject(this)` 의 같은 코드를 10번 써야할 것이다 😨

> 📍Application 서브 클래스를 생성하여 전역으로 사용할 수 있게 한다.

## 생성하기
Application을 상속 받는 커스텀 SmartPhoneApplication 클래스를 생성한다.

onCreate() 함수를 오버라이드 하여 어플리케이션 실행전에 Component를 build해준다.

```kotlin
class SmartPhoneApplication: Application() {
    lateinit var smartPhoneComponent: SmartPhoneComponent
    override fun onCreate() {
        super.onCreate()
        smartPhoneComponent = initDagger()
    }

    private fun initDagger(): SmartPhoneComponent =
        DaggerSmartPhoneComponent.builder()
            .build()

}
```
이후에 `AndroidManifest.xml`파일에 이름을 지정해 줘야한다.
```xml
    <application
        android:name=".SmartPhoneApplication"
            .
            .
    </application>
```
여기서 __android:name__ 의 역할은 구글 문서에서 다음과 같이 정의되어 있다.

> 📃: 어플리케이션에 대해 구현된 어플리케이션 서브클래스의 이름. 어플리케이션 프로세스(앱이 실행)가 시작될 때, 어플리케이션의 다른 어떤 컴포넌트보다 먼저 인스턴스화된다(객체화된다). 서브클래스는 없어도 된다(대부분의 어플리케이션은 서브클래스를 사용하지 않는다). 서브클래스가 없으면, 안드로이드는 기본 어플리케이션 클래스의 객체를 사용한다.

마지막으로 사용하고자하는 Activity/Fragment에서 사용하면 된다.

```kotlin
class MainActivity : AppCompatActivity() {
    // Field Injection
    @Inject
    lateinit var smartPhone: SmartPhone

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        (application as SmartPhoneApplication).smartPhoneComponent
            .inject(this)

        smartPhone.start()
    }
}
```
# Scope
특정 Component에 Scope를 지정하고 이와 같은 생명주기를 같이할 클래스에 Scope를 지정할 수 있다. Scope가 지정되면 해당 Component의 생명주기 동안은 항상 같은 객체를 반환 받는 것을 보장한다.

🧷Scoping rules
1. type에 Scope을 붙일 때는 같은 Scope이 붙은 Component에 의해서만 사용 가능하다.
2. Component에 Scope를 지정하면 Scope이 없는 type이나 동일한 Scope이 붙은 type만 제공할 수 있다.
3. SubComponent는 상위 Component에서 사용 중인 annotation을 사용할 수 없다.

Custom Scope를 생성하기 위해서는 다음과 같이 생성한다.
```kotlin
@Scope
@kotlin.annotation.Retention(AnnotationRetention.RUNTIME)
annotation class CustomScope
```

## @Singleton
Custom Application의 생성은 어플리케이션에서 공유되어 사용하기 위해서 만들어 준다고 했다. 지금까지 Dagger를 적용하여 SmartPhone을 실행하고 화면의 회전을 하게 되면 __SmartPhone이 다시 생성되어 실행되는 것을 볼 수 있다.__

<img src="https://user-images.githubusercontent.com/63226023/161263860-d0937a5d-aa2b-46df-8a57-b079fa97f6c4.png">

안드로이드 Room Database의 경우 room DB에 접근을 할때 Database객체를 받아와야 한다. 구글에선 한번 생성된 Database를 이용해서 사용하도록 권고 합니다.(싱글톤으로 만들어준다)

이때 @Singleton 어노테이션을 붙여서 한개의 컴포넌트만 생성할 수 있게 해준다. 

> 📍@Singleton은 javax.inject 패키지 하위에 있는 어노테이션으로 Dagger의 어노테이션 X.

```kotlin
@Singleton
class SmartPhone @Inject constructor(val battery: Battery, val simCard: SIMCard, val memoryCard: MemoryCard) {
    fun start() {
        Log.i("TAG", "스마트폰 동작!")
    }
}

@Singleton
@Component(modules = [MemoryCardModule::class])
interface SmartPhoneComponent {
    fun inject(mainActivity: MainActivity)
}
```

싱글톤으로 생성되어야하는 SmartPhone 클래스와 Component 인터페이스에 @Singleton 어노테이션을 붙여주고 아까와 같이 실행후 화면을 돌려봐도 같은 객체를 사용하는 것을 볼 수 있다.

<img src="https://user-images.githubusercontent.com/63226023/161267195-f762a0ab-6f9e-4496-9a64-3c15b8b2fdb8.png">

# Component
## Builder
Component 인스턴스를 생성하기 위한 __Builder__ 용 어노테이션이다. __Module에 초기 설정__ 을 할때 사용한다. Component 내의 interface 또는 abstract 클래스에 붙여서 사용한다.

```kotlin
@Singleton
@Component(modules = [MemoryCardModule::class])
interface SmartPhoneComponent {
    fun inject(mainActivity: MainActivity)

    @Component.Builder
    interface Builder {
        fun getMemoryCardModule(memoryCardModule: MemoryCardModule): Builder
        fun build(): SmartPhoneComponent
    }
}

class SmartPhoneApplication: Application() {
    .
    .
    private fun initDagger(): SmartPhoneComponent =
        DaggerSmartPhoneComponent.builder()
            .getMemoryCardModule(MemoryCardModule())
            .build()

}
```
Builder는 __반드시 Component를 반환하는 메소드__ (위의 예시: build()), Builder를 반환값으로 가지면서 Component가 필요로하는 Module을 파라미터로 받는 메소드를 가지고 있어야한다.

Builder를 사용하지 않고 Dagger2를 생성하였을때도 이미 Component에는 자동으로 Builder가 생성되어 있다.

```kotlin
@Singleton
@Component(modules = [MemoryCardModule::class])
interface SmartPhoneComponent {
    fun inject(mainActivity: MainActivity)
}
```
```java
// DaggerSmartPhoneComponent.java
 public static final class Builder {
    private MemoryCardModule memoryCardModule;

    private Builder() {
    }

    public Builder memoryCardModule(MemoryCardModule memoryCardModule) {
      this.memoryCardModule = Preconditions.checkNotNull(memoryCardModule);
      return this;
    }

    public SmartPhoneComponent build() {
      if (memoryCardModule == null) {
        this.memoryCardModule = new MemoryCardModule();
      }
      return new DaggerSmartPhoneComponent(memoryCardModule);
    }
```
## Factory
`daager-android 2.22` 버전에서 추가된 Factory 어노테이션은 Builder와 의미는 같지만 사용법 다른 형태이다. Builder는 파라미터를 __1개__ 만 받으며 여러 Module을 설정해야 한다면 각각의 메소드를 선언하여 메소드 체이닝이 길어지는 점을 보완한 것이 Factory이다.

Factory는 __단 하나__ 의 메소드(create())만 선언되어야 하고 Component 인스턴스를 반환해야 한다.

```kotlin
@Singleton
@Component(modules = [MemoryCardModule::class, ModuleB::class])
interface SmartPhoneComponent {
    fun inject(mainActivity: MainActivity)

    @Component.Factory
    interface Factory {
        fun create(memoryCardModule: MemoryCardModule, moduleB: ModuleB): SmartPhoneComponent
    }
}
```
# @SubComponent 
SubComponent는 부모 Component가 있는 자식 Component라고 볼 수 있다. Component는 SubComponent로 계층관계를 만들 수 있으며, SubComponent는 Dagger의 그래프를 형성하는 역할이다.

Inject으로 의존성 주입을 요청받으면 SubComponent에서 먼저 의존성을 찾고, 없으면 상위 부모로 올라가면서 검색한다.

@Component를 사용할 때는 직접 해당하는 component의 인스턴스를 반환하는 Factory를 만들어 줄 필요가 없었지만 @SubComponent는 __반드시 부모 컴포넌트가 어떻게 서브 컴포넌트가 만들면 되는지 알도록 해야하므로 Factory를 구현해야한다.__


```kotlin
@CustomScope
@Subcomponent
interface PpeperSubComponent {
    fun inject(aActivity: A_Activity)

    @Subcomponent.Factory
    interface Factory {
        fun create(): PpeperSubComponent
    }
}
```

1. SubComponentModule을 만들고 서브컴포넌트 클래스를 subcomponents 속성값으로 넣어준다.

```kotlin
@Module(subcomponents = [PpeperSubComponent::class])
class SubComponentModule {}
```
2. 만든 모듈을 부모 Component에 추가한다.
3. 사용할 SubComponent를 외부로 노출시켜 다른곳에서 가져다 쓸수 있게 해준다.

```kotlin
@Singleton
@Component(modules = [ModuleA::class, ModuleB::class, SubComponentModule::class])
interface ParentComponent {
    fun pppeperSubComponent(): PpeperSubComponent.Factory  
}
```
- - -
# References
- [Android 개발자 > 문서 > 가이드](https://developer.android.com/guide/topics/manifest/application-element.html#nm)
- [https://jaejong.tistory.com/125](https://jaejong.tistory.com/125)