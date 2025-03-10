---
title: "Kotlin 제네릭의 in, out 키워드?"
date: 2022-06-25
update: 2022-06-25
tags:
  - 제네릭
  - 공병성
  - 반공병성
  - 불변성
series: "Kotlin"
---
안드로이드와 코틀린을 공부 하면서 `out`, `in` 키워드를 많이 봤지만 의미를 정확히 알지 못하였던 개념에 대해서 차근차근 알아가보려고 한다 😅

# 제네릭(Generic)
프로그래밍 언어에서는 Int, Char, String등 기본(Primitive) 데이터 타입을 지원한다. __제네릭__ 은 __타입을 확실히 정하지 않고__ 동일한 코드를 사용할 수 있도록 지원해주는 유용한 기능이다. 이는 __`<T>`__ 로 많이 친숙하게 볼 수 있는 친구다. 
```kotlin
fun <T> generic(value: T) {
    println(value)
}

fun main() {
    generic("ppeper")
    generic("26")
}
```
# 불변성이란??
`제네릭`은 자바와 마찬가지로 __타입 불변성__ 을 가진다. 타입 불변성은 제네릭 타입을 사용하는 __클래스나 인터페이스__ 에서는 __일치하는 타입만 사용할 수 있다__ 는 것을 말한다. 즉 해당 타입의 부모,자식의 타입은 사용이 불가하다. 그러한 이유는 예시를 통하여 알아가보자.

```kotlin
open class Phone
class Apple: Phone()
class Samsung : Phone()

// 상속 관계로 부모에 자식 사용이 가능하다.
val iphone: Phone = Apple()

// 오케이 나도 해볼까
// Type Mismatch -> Required: Array<Phone> / Found: Array<Apple>
// ....
val iphones: Array<Apple> = arrayOf(Apple(), Apple())
val phones: Array<Phone> = iphones
```
> 🤔분명 Phone을 상속받은 Apple 클래스인데 Type Mismatch?

위와 같이 제네릭의 타입을 가지는 클래스, 인터페이스에서 __클래스의 상속관계가 형식인자의 상속관계와 같이 유지되지 않는다__ 즉, `A -> B` 일때 `Class<A> -> Class<B>` 를 만족하지 못한다. 이를 __Invariance(불변성)__ 이라고 한다.

__불변성(Invariance)__ 이 존재하는 이유는 다음과 같은 문제가 일어날 수 있기 때문이다!

```kotlin
fun myPhones(phones: Array<Phone>) {
    // ...???
    phones[0] = Apple()
}

fun main() {
    val galaxys: Array<Samsung> = arrayOf(Samsung())
    myPhones(galaxys)
}
```
Samsung폰에 대한 galaxys 변수의 Array를 모르고 myPhones[0]에 Apple()을 넣어준다면 타입이 맞지 않아 문제가 발생하게 된다.
- - -
## 불변성에 대한 한계점
__불변성__ 은 __컴파일 타임 에러를 잡아주고 런타임에 에러를 내지않는 안전한 방법__ 이다. 그러나 이는 가끔 안전하다고 보장된 상황에서도 컴파일 에러를내 개발자를 불편하게 할 수 있다. 

```kotlin
fun copy(from: Array<Phone>, to: Array<Phone>) {
    for (index in from.indices) {
        to[index] = from[index]
    }
}

fun main() {
    val phones: Array<Phone> = arrayOf(Phone())
    val galaxys: Array<Samsung> = arrayOf(Samsung())
    copy(galaxys, phones)
    // Type Mismatch -> Required: Array<Phone> / Found: Array<Samsung>
}
```
phones에 galaxys를 copy를 하는 함수는 문제가 없어보인다. 하지만 위에서 말한 __불변성__ 으로 인하여 __`A -> B` 일때 `Class<A> -> Class<B>`__ 를 만족하지 못하여 컴파일 에러로 판단한다.

이를 해결하기 위해서는 개발자가 __`A -> B` 일때 `Class<A> -> Class<B>`__ 를 상속 받게 바꿔 주어야한다. 이를 __공병성__ 이라고 하며 코틀린에서는 __`out`__ 키워드를 사용한다.
- - -
## 📍공병성, out 키워드
```kotlin
fun copy(from: Array<out Phone>, to: Array<Phone>) {
    for (index in from.indices) {
        to[index] = from[index]
    }
}

fun main() {
    val phones: Array<Phone> = arrayOf(Phone())
    val galaxys: Array<Samsung> = arrayOf(Samsung())
    copy(galaxys, phones)
}
```
> 위와 같이 __`out`__ 키워드를 통하여 __공변성__ 으로 변환을 통하여 불필요한 __불변성__ 문제를 해결할 수 있다.

❗하지만 여기서 from에 Samsung()을 Write하려고 한다면 에러가 발생한다. 
```kotlin
fun copy(from: Array<out Phone>, to: Array<Phone>) {
    for (index in from.indices) {
        to[index] = from[index]
    }
    // Type Mismatch -> Required: Nothing / Found: Samsung
    from[0] = Samsung()
}
```
from[0]에서는 Nothing 즉, 아무것도 입력받기를 원하지 않는다. 이유는 불변성과 비슷하다고 할 수 있다.

> __[Read]__
>
> from에서는 `Array<out Phone>`을 통하여 컴파일러가 from의 부모가 Phone인것과 sub Type인 Apple, Samasung 중 하나인것을 인지 하고 있다. 따라서 읽을때는 문제가 발생하지 않는다.
>
> __[Write]__
>
> 하지만 from에 값을 쓰려고 한다면 타입이 Phone, Apple, Samsung 중 하나라는 것만 인지하고 있을뿐 __실제 타입을 모르는 Array__ 에서 값을 쓸 수가 없는 것이다.

그렇다면 공병성과 반대되는 Read가 불가능하고 Write만 할 수 있는 것이 있지 않을까?

Read가 가능한 __`out`__ 키워드가 있다면 반대로 Write이 가능한 
__`in`__ 키워드가 있다. 이를 __반공병성(Contravariance)__ 라고 한다.
- - -
to 파라미터에 대해서 __`Array<Phone>`__ 에 대하여 __Phone의 super Type인 부모 클래스__ 를 전달하고 싶다면 어떻게 해야할까?

```kotlin
fun copy(from: Array<out Phone>, to: Array<Phone>) {
    for (index in from.indices) {
        to[index] = from[index]
    }
}

fun main() {
    val phones: Array<Any> = arrayOf(Any())
    val galaxys: Array<Samsung> = arrayOf(Samsung())
    copy(galaxys, phones)
    // Type Mismatch -> Required: Array<Phone> / Found: Array<Any>
}
```
위와 같이 __`Array<Any>`__ 로 선언하고 싶지만 위에서 설명한 __불변성으로 이는 안된다고 하였고__ 미리 컴파일러는 Type Mismatch를 통하여 에러를 알려 준다~~(고..마워😅)~~.

이에 대해서 명시적으로 부모 클래스를 넘겨 줄 수 있는 방법이 __`in`__ 키워드이다.

# 📍반공변성 in!
```kotlin
fun copy(from: Array<out Phone>, to: Array<in Phone>) {
    for (index in from.indices) {
        to[index] = from[index]
    }
}

fun main() {
    val phones: Array<Any> = arrayOf(Any())
    val galaxys: Array<Samsung> = arrayOf(Samsung())
    copy(galaxys, phones)
}
```
> 위와 같이 __`in`__ 키워드를 통하여 __반공변성__ 으로 변환을 통하여 불필요한 __불변성__ 문제를 해결할 수 있다.

`to[index] = from[index]`로 반공병성은 Write할때는 문제가 되지 않지만 반대로 Read를 하려고 하면 문제가 발생한다.

```kotlin
fun copy(from: Array<out Phone>, to: Array<in Phone>) {
    for (index in from.indices) {
        to[index] = from[index]
    }
    val phone: Phone = to[0]
    // Type Mismatch -> Required: Phone / Found: Any?
}
```
이에 대한 문제는 공병성과 반대로 컴파일러가 to 매개변수를 읽으려 할때 __반공병성__ 으로 __Phone과 그에 대한 조상 타입을 명시적으로 가능하게 하였으므로__ 읽을때는 실제 타입을 모르기 때문에 함부로 읽을 수 없는 것이다.

# 정리
공병성, 반공병성에 대해서 학습을 해보았지만 간단한 예시를 통하여 해보았기 때문에 많은 소스를 접해보고 사용해 봐야 할 것 같다.

> out: __꺼내와서(out 시킨다)__ 읽는다. -> Write은 불가능
>
>   - 슈퍼 클래스에 서브 클래스를 사용가능하게 해준다.
>
> in: __넣어준다(in 시킨다)__ 즉, Write 할 수 있다. -> Read는 불가능
>   - 서버 클래스에 슈퍼 클래스를 사용가능하게 해준다.

- - -
# References
- [https://medium.com/mj-studio/%EC%BD%94%ED%8B%80%EB%A6%B0-%EC%A0%9C%EB%84%A4%EB%A6%AD-in-out-3b809869610e](https://medium.com/mj-studio/%EC%BD%94%ED%8B%80%EB%A6%B0-%EC%A0%9C%EB%84%A4%EB%A6%AD-in-out-3b809869610e)
- [https://readystory.tistory.com/201](https://readystory.tistory.com/201)