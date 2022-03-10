---
title: 의존성 주입(DI) 알아보기
categories: CS
tags: ['CS','DI']
header:
    teaser: /assets/teasers/android-di-image.png
last_modified_at: 2022-3-10T00:00:00+09:00
---
<p align="center"><img src="https://user-images.githubusercontent.com/63226023/157641205-b344edbb-e4e3-4965-a366-5ee1401325b5.png"></p>

의존성 주입을 최근에 공부하게 되면서 의존성 주입에 대한 개념을 알아보려고 한다🤔

- - -

# 의존성 주입💉

의존성 주입(Dependency Injection)은 객체지향 프로그래밍을 하게 되면 한번씩 들어볼 수 있는 용어이다.

일반적으로 객체생성을 하게되면 사용할 클래스 내에서 객체를 생성하여 사용하지만, DI는 __외부에서 생성된 객체를 주입__ 을 받는다. 
> 📍의존성 주입: 외부에서 생성된 객체를 주입한다.

<img src="https://user-images.githubusercontent.com/63226023/157621746-f6102708-c8c2-4698-ad6b-4b6aff63ca9b.png">

## 일반적인 예시
스마트폰이라는 객체가 있고 스마트폰에는 배터리, 유심칩, 메모리카드가 있다고 하자. 이를 일반적인 객체생성을 하게 되면 아래와 같을 것이다.
```kotlin
class SmartPhone {

    init {
        val battery = Battery()
        val simCard = SIMCard()
        val memoryCard = MemoryCard()
    }

    fun start() {
        Log.i("TAG", "스마트폰 동작!")
    }
}

class MainActivity : AppCompatActivity() {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        val smartPhone = SmartPhone()
        smartPhone.start()
    }
}

```
이렇게 해서 스마트폰을 생성하여 사용하더라도 문제없이 동작을 한다.

<img src="https://user-images.githubusercontent.com/63226023/157629828-8ad84f5a-a74b-4866-ba14-a2f8ceea556e.png">

여기서 만약 배터리를 다른 용량의 배터리로 교체를 하려고 한다면 __스마트폰 객체의 코드에서 다른 배터리 객체__ 로 바꿔야할 것이다. 

위의 간단한 예시에서는 한줄의 코드만 바꾸면 되지만 의존 관계가 복잡해 진다면 바꿔야 할 코드가 굉장히 많아지고 또한 어느 패키지에 있는 클래스를 바꿔야할지 등 코드 수정이 굉장히 힘들어질것이다😨

## 생성자 주입(constructor Injection) 사용
> 📍생성자를 통해 의존하는 객체를 주입하는 방법

스마트폰과 부품들간의 결합도를 느슨하게 하기위하여 예시의 코드를 변경해보자.

```kotlin
class SmartPhone(val battery: Battery, val simCard: SIMCard, val memoryCard: MemoryCard) {

    fun start() {
        Log.i("TAG", "스마트폰 동작!")
    }
}

class MainActivity : AppCompatActivity() {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        val battery = Battery()
        val simCard = SIMCard()
        val memoryCard = MemoryCard()
        val smartPhone = SmartPhone(battery, simCard, memoryCard)
        smartPhone.start()
    }
}
```

이렇게 스마트폰을 생성을 하게되면 __다른 용량이 큰 배터리로 교체를 하더라도 새로운 배터리 객체를 생성하여 스마트폰에 주입__ 을 할 수 있다. 이렇게 생성자를 통하여 의존성 주입을 하는 방법을 __생성자 주입__ 이라이고 한다.

## 필드 주입(Field Injection) 사용
> 📍객체를 초기화한 후 의존하는 객체로 주입하는 방법

```kotlin
class SmartPhone {
    lateinit var battery: Battery
    lateinit var simCard: SIMCard
    lateinit var memoryCard: MemoryCard

    fun start() {
        Log.i("TAG", "스마트폰 동작!")
    }
}

class MainActivity : AppCompatActivity() {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        val battery = Battery()
        val simCard = SIMCard()
        val memoryCard = MemoryCard()

        val smartPhone = SmartPhone()
        smartPhone.battery = battery
        smartPhone.simCard = simCard
        smartPhone.memoryCard = memoryCard
        smartPhone.start()
    }
}
```

위의 예시처럼 배터리, 유심카드, 메모리 카드를 생성하여 외부에서 해당 멤버변수로 직접 값을 넣어주는 방법을 __필드 주입__ 이라고 한다.

## 의존성 주입의 장점
앞서 살펴본 의존성 주입을 사용한다면 클래스간의 결합도가 약해지는것을 볼 수 있었다. 이러한 클래스간의 결합도가 약해지면 얻어지는 장점들은 다음과 같다.
- __코드의 리펙토링이 쉬워진다.__
- __코드의 재사용성이 좋아진다.__
- __코드의 가독성이 좋아진다.__
- __결합도가 약해져서 Unit testing에 용이하다.__

- - -

# 마무리
안드로이드에서 의존성 주입은 Dagger라는 프레임워크와 kotlin 언어를 사용하면 Koin을 많이 사용한다고 한다. 최근에 안드로이드 의존성 주입에 대해서 공부를 하면서 굉장히 러닝커브가 높다는걸 느꼈다.😅~~(프로젝트에 적용시킬 수 있을까..)~~ 


