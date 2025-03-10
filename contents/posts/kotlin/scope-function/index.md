---
title: "코틀린의 Scope Function는 언제쓸까"
date: 2022-04-20
update: 2022-04-20
tags:
  - Kotlin
  - Scope Function
series: "Kotlin"
---
# Scope Function
코틀린에서는 기본적으로 __"표준 스코프 함수"__ 라는것을 제공한다.   
스코프 함수들을 사용하여 기존의 복잡한 코드를 단순화하고 효율적으로 만들 수 있다.

스코프 함수를 __람다식으로 사용을 하게되면 일시적인 Scope(범위)가 형성__ 되고, 이 범위 안에서는 __객체에 대해서 일일이 참조하지 않고 객체에 대한 접근__ 을 하여 사용할 수 있다.

코틀린에서 제공하는 스코프 함수들은 `let`, `also`, `with`, `run`, `apply`로 5가지로 이루어져 있다.   
각 키워드에 대해서 단어의 의미를 보면 어렴풋이 어떠한 기능을 할 것이다라는 감은 오지만 __역할이나 수행하는 기능들이 비슷하여__ 어떤 상황에 알맞은 스코프 함수를 써야하는지 __헷갈리게 된다.__(지금상태..😨)

최근에 안드로이드 프로젝트에서 스코프 함수를 사용하고 있지만 명확한 차이점을 알지못하고 제대로 사용하고 있나? 라는 의문이 생겨 이번 포스팅을 통하여 각각의 차이점, 특징들을 정리하려고 한다.

## let
> fun <T, R> T.let(block: (T) -> R): R

`let` 함수는 제네릭으로 매개변수화된 타입 T의 확장 함수이다. 따라서 __자기자신을 인수로 전달하고 수행된 결과(블록의 마지막 값)를 반환__ 하며, 인수로 전달한 객체의 참조는 __it__ 으로 참조를 한다.

```kotlin
data class Person(var name: String, var favorite: String)

fun main() {
    val person1 = Person("페퍼", "Dr pepper")
    person1.let {
        println("이름: ${it.name}, 좋아하는것: ${it.favorite}")
    }
    // 이름: 페퍼, 좋아하는것: Dr pepper

    val person2 = person1.let {
        it.name = "준후"
        it.favorite = "닥터페퍼"
        "이름: ${it.name}, 좋아하는것: ${it.favorite}"    // (T) -> R 부분에서의 반환값
    }
    println(person2)
    // 이름: 준후, 좋아하는것: 닥터페퍼
}
```
안드로이드에서 let은 `T?.let{ }`(safe call)의 형태로 let 블록안에 __not-null__ 만 들어올 수 있어 null 체크 시에 유용하게 쓰인다. 

```kotlin
val response = // 서버로부터 데이터 요청
        if (response.isSuccessful) {
            // body()가 null이 아니라면 실행
            // it -> result라는 이름의 인자로 바꾸어 사용가능
            response.body()?.let { result ->
                return // Something
            }
        }
```

## also
> fun <T> T.also(block: (T) -> Unit): T

`also`는 언듯보면 `let`과 역할이 거의 동일해 보인다. 하지만 반환값을 보면 __T__ 라는 것으로 조금 다른데, __let은 마지막으로 수행된 코드 블록의 결과를 반환__ 하고 __also는 블록 안의 코드와 상관없이 T인 객체를 반환__ 한다.

```kotlin
val person2 = person.let {
    it.name = "준후"
    it.favorite = "닥터페퍼"
    "이름: ${it.name}, 좋아하는것: ${it.favorite}"    // (T) -> R 부분에서의 반환값
}
println(person2)
// 이름: 준후, 좋아하는것: 닥터페퍼

val person3 = person.also {
    it.name = "준후"
    it.favorite = "닥터페퍼"
    "이름: ${it.name}, 좋아하는것: ${it.favorite}"    // 무시
}
println(person3)
// Person(name=준후, favorite=닥터페퍼) -> T인 person 객체 반환
```

## with
> fun <T, R> with(receiver: T, block: T.() -> R): R

`with`는 다른 스코프 함수들과 다르게 __일반 함수__ 이다. 따라서 __객체 receiver를 직접 입력__ 받고 `receiver`로 객체를 입력 받으면 , `this` 키워드 없이 __객체의 속성__ 을 변경할 수 있다.

`with`는 `not-null`객체를 이용하여 사용하고, 블록의 반환값이 필요하지 않을 때 사용한다.

주로 `with`는 __객체의 함수나 속성을 여러개 호출할 때 그룹화__ 하는 용도로 많이 활용한다.

```kotlin
binding.userId.text~
binding.userpwd.text~

// 주로 안드로이드에서 view binding을 사용할때 with사용
with(binding) {
    userId.text ~~
    userpwd.text ~
}
```

## run
`run`은 두 가지 형태로 선언이 되어 있는데, 첫 번째는 아래와 같다.
> fun <T, R> T.run(block: T.() -> R): R

`run`은 with와 유사하지만 T의 확장함수라는 점에서 차이가 있다. 또한 확장함수 이기때문에 `T?.`(safe call)을 사용하면 __null 객체가 들어와도 not-null을 체크하고 실행__ 이 가능하다. 

`run`은 __어떤 값을 계산할 필요__ 가 있거나, __여러 개의 지역변수 범위를 제한__ 하고자 할 때 사용한다.

```kotlin
val favorite = person.run {
    // person을 수신객체로 변환하여 favorite 값을 사용
    println("가장좋아하는것: $favorite")
    favorite.plus(" 존맛탱!!")     // run은 마지막 실행문의 결과를 반환
}
println("가장좋아하는것: $favorite")
// 가장좋아하는것: 닥터페퍼
// 가장좋아하는것: 닥터페퍼 존맛탱!!
}
```
두 번째 선언 방식은 아래와 같다.
> fun <R> run(block: () -> R): R

이 `run`은 __확장함수도 아니고, 블록에 대한 입력값도 없다.__ 따라서 객체를 전달 받고 속성을 이용할때 사용하는 함수가 아니고, __어떤 객체를 생성하기 위한 명령문들을 하나로 묶음으로써 가독성을 높이는 역할을 한다.__

```kotlin
val person = run {
    val name = "ppeper"
    val favorite = "닥터페퍼"
    Person(name, favorite) // Return
}
```
이렇게 사용한다면 Person() 객체 가 `person`에 담기게 된다.

## apply
> fun <T> T.apply(block: T.() -> Unit): T

`apply`는 T의 확장함수로 run과 유사하지만 반환값을 받지않고 객체 T를 반환한다는 점이 다르다. 따라서 apply는 이름에서 느낄수 있듯이, __새로운 인스턴스를 생성하고 특정 변수에 할당하기 전__ 에 __초기화 작업__ 을 하거나 변경할때 사용한다. 따라서 앞서 설명한 것처럼 `apply` 스코프 내에 모든 명령을 수행하여 적용된 새로운 인스턴스가 (T) 반환 한다는 특징이 있다.

```kotlin
val person = Person("", "")
person.apply {
    name = "ppeper"
    favorite = "닥터페퍼"
}
println("$person")
// Person(name=ppeper, favorite=닥터페퍼)
``` 
- - -

# References
- [https://blog.yena.io/studynote/2020/04/15/Kotlin-Scope-Functions.html](https://blog.yena.io/studynote/2020/04/15/Kotlin-Scope-Functions.html)