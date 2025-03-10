---
title: "안드로이드 Dagger2 맛보기"
date: 2022-03-31
update: 2022-03-31
tags:
  - Android
  - Dagger
series: "Android"
---
# Dagger
Dagger는 [의존성 주입(DI)](https://ppeper.github.io/cs/dependency-Injection/)을 도와주는 프레임워크이다.   
의존성 주입(DI)에 대해서 전 포스팅에서 알아보았다. Dagger를 사용하여 의존성 주입을 자동화하도록 지원하여 코드 작성에 편의성을 제공한다. 전 포스팅에서 만들어 보았던 스마트폰 예시에 Dagger를 적용해 보려고 한다.

# 시작하기
## build.gradle
프로젝트에 Dagger를 사용하려면 `build.gradle`에 아래의 종속 항복을 추가해야 한다.
최신 버전의 Dagger는 [Github](https://github.com/google/dagger) 에서 확인 할 수 있다.
```kotlin
plugins {
    .
    .
    id 'kotlin-kapt'
}


dependencies {
    .
    .
    // Dagger2
    implementation "com.google.dagger:dagger:2.41"
    kapt "com.google.dagger:dagger-compiler:2.41"
}
```
## @Inject
__@Inject__ 를 사용하게 되면 Dagger가 해당 타입을 어떻게 생성하는지 알수 있게 한다. __@Inject__ 는 `필드`, `생성자`, `메서드`에 붙여 Component로 부터 의존성 객체를 주입 요청하는 어노테이션이다. 객체(인스턴스)의 생성이 클래스에 의존적이지 않고 Component가 생성해 주기 때문에 __보일러 플레이트 코드가 줄어들고 테스트하기 수월해 진다.__

__@Inject__ 로 의존성 주입을 요청하면 연결된 Component의 Module에서 의존 객체가 생성되어 반환된다.

- - -

각 Battery, MemoryCard, SIMCard, SmartPhone에 __@Inject__ 어노테이션을 추가한다. -> `생성자를 통한 의존성 주입`


```kotlin
class Battery @Inject constructor() {
    init {
        Log.i("TAG", "배터리 생성완료!")
    }
}

class MemoryCard @Inject constructor(){
    init {
        Log.i("TAG", "메모리 카드 생성완료!")
    }
}

class SIMCard @Inject constructor(){
    init {
        Log.i("TAG", "유심칩 생성완료!")
    }
}

class SmartPhone @Inject constructor(val battery: Battery, val simCard: SIMCard, val memoryCard: MemoryCard) {
    fun start() {
        Log.i("TAG", "스마트폰 동작!")
    }
}
```

## @Component
__@Component__ 는 `Interface` 또는 `abstract class`에만 사용이 가능하다.

> 📍 __컴파일 타임__ 에 접두어 `Dagger`와 `Component 클래스 이름`이 합쳐진 Dagger클래스 자동생성   
> ex) SmartPhoneComponent -> DaggerSmartPhoneComponent

연결된 Module로 부터 의존성 객체를 생성하고, Inject로 요청받은 인스턴스에 생성한 객체를 주입한다.

### Component Methods

__@Component__ 어노테이션이 가지는 인터페이스 또는 abstract class는 __하나 이상의 메소드__ 를 가지고 있어야 한다.

메소드의 방식에는 `Provision Method` 와 `Member-Injection` 이 있다.

#### 1. Provision Method
Provision Method의 경우 매개변수가 없고, Module이 제공하는 객체의 타입을 반환값으로 갖는다.

#### 2. Member-Injection
의존성 주입을 시킬 객체를 파라미터로 넘기는 방식으로, 해당 객체의 `클래스 내부`에서 `@Inject가 붙은 필드에 객체`를 주입한다.

```kotlin
@Component
interface SmartPhoneComponent {
    // Provision Method 방식
    fun getSmartPhone(): SmartPhone
}
```
## Component 객체 구현 - create() / build()
Dagger로 생성된 Component는 위에서 말한 형태로 `Dagger + Component 클래스 이름`으로 자동 생성된다. 
> 📍해당하는 클래스가 생성이 되지않았다면   
> 
> 1. Rebuild Project 실행   
> 2. 1번을 수행해도 안된다면 `dagger-complier` 라이브러리를 포함하는지 확인

Component의 인스턴스를 생성하는 방법은 `create(), build()`로 2가지 방법이 있다.

```kotlin
class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

//        val battery = Battery()
//        val simCard = SIMCard()
//        val memoryCard = MemoryCard()
//        val smartPhone = SmartPhone(battery, simCard, memoryCard)
//        smartPhone.start()

        // 1. create() 방식
        val smartPhone = DaggerSmartPhoneComponent.create()
            .getSmartPhone()
    

        // 2. build() 방식
        val smartPhone = DaggerSmartPhoneComponent.builder()
            .build()
            .getSmartPhone()
            
        smartPhone.start()
    }
}
```
생성된 스마트폰 Component에 있는 smartPhone.start()를 통하여 실행을 하게되면 동작을 하는것을 볼 수 있다.

<img src="https://user-images.githubusercontent.com/63226023/157629828-8ad84f5a-a74b-4866-ba14-a2f8ceea556e.png">

```java
@DaggerGenerated
@SuppressWarnings({
    "unchecked",
    "rawtypes"
})
public final class DaggerSmartPhoneComponent implements SmartPhoneComponent {
  private final DaggerSmartPhoneComponent smartPhoneComponent = this;

  private DaggerSmartPhoneComponent() {


  }

  public static Builder builder() {
    return new Builder();
  }

  public static SmartPhoneComponent create() {
    return new Builder().build();
  }

  @Override
  public SmartPhone getSmartPhone() {
    return new SmartPhone(new Battery(), new SIMCard(), new MemoryCard());
  }

  public static final class Builder {
    private Builder() {
    }

    public SmartPhoneComponent build() {
      return new DaggerSmartPhoneComponent();
    }
  }
}
```

자동으로 생성된 `DaggerSmartPhoneComponent` 클래스에서 `getSmartPhone() `을 상속받아 자동으로 객체들을 주입해준것을 볼 수 있다.

# Third Party Library는??
여기서 외부에 존재하는 `Third Party Library`라면 함수의 내부를 볼 수 없기 때문에 `@Inject`어노테이션을 사용할 수 없다. 예를 들어 Retrofit client의 경우 Third party Library 이기 때문에 생성자를 통한 의존성 주입이 불가능하다.

```kotlin
// Retrofit
class Retrofit @Inject constructor()??{
}
```
이러한 해당 객체를 의존성 주입을 사용하기 위해서는 __Module__ 과 __provider function__ 을 사용하여 주입해야한다.

## @Module / @Provides

Dagger에서 의존성 관계를 설정하는 클래스를 __Module__ 이라고 부르고 @Module 어노테이션을 붙여준다. 즉 Dagger에서 Module은 의존성 주입에 필요한 객체들을 관리해준다. 

__@Provide__ 어노테이션은 @Module클래스에서 선언된 메소드들에만 붙임으로서 Dagger가 메소드에서 해당 타입의 객체를 반환해 준다는것을 알 수 있게 해준다.

Module 클래스는 `클래스 이름 + Module`으로 생성하고, Provide 메소드는 `provide + 메소드 이름` 으로 만드는것이 일반적인 컨벤션이다.
- - -
위의 예시에서 MemoryCard가 Third Party Library라고 생각하고 Module을 생성해 보면 아래와 같다.
```kotlin
// third party 라이브러리하고 생각
class MemoryCard {
    init {
        Log.i("TAG", "메모리 카드 생성완료!")
    }
}
// Module 클래스 생성 
@Module
class MemoryCardModule {

    @Provides
    fun provideMemoryCard(): MemoryCard {
        return MemoryCard()
    }
}
```
Component가 생성된 Module을 알 수 있게 하기위해선 @Component에 아래와 같이 추가해 주면 된다.

```kotlin
@Component(modules = [MemoryCardModule::class])
interface SmartPhoneComponent {
    // Provision Method 방식
    fun getSmartPhone(): SmartPhone
}
```
- - -
# Field Injection
위의 Inject, Module, Component를 사용하여 의존성 주입을 해보았다. 실제로 의존성 주입을 사용하게되면 SmartPhone과 같은 Component 이외에 다은 의존성 객체가 많이 존재 할 수 있다. 이러한 Component들을 MainActivity에서 의존성 주입을 할때마다 생성해주어야 한다면 getSmartPhone()과 같은 getter 메소드들이 불필요하게 많이 추가가 될것이다.

Component에서 Method-Injection 방식을 사용하여 사용할 Activity / Fragment를 지정해 주고 사용할 View에서 Field Injection을 사용하면 불필요한 코드의 사용을 줄일 수 있다.

```kotlin
@Component
interface SmartPhoneComponent {
    // Provision Method 방식
    // fun getSmartPhone(): SmartPhone

    // Method-Injection 방식
    fun inject(mainActivity: MainActivity)
}
```

```kotlin
class MainActivity : AppCompatActivity() {
    // Field Injection
    @Inject
    lateinit var smartPhone: SmartPhone
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        // 1. create() 방식
        // val smartPhone = DaggerSmartPhoneComponent.create()
        //     .getSmartPhone()
    
        DaggerSmartPhoneComponent.create()
            .inject(this)
            
        smartPhone.start()
    }
}
```

- - -
# References
- [Android 앱에서 Dagger 사용](https://developer.android.com/training/dependency-injection/dagger-android?hl=ko)
- [https://jaejong.tistory.com/125](https://jaejong.tistory.com/125)