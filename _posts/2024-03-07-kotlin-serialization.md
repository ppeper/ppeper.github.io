---
title: 코틑린을 위한 Kotlinx Serizalization
categories: Kotlin
tags: ['Android', 'Kotlin']
header:
    teaser: /assets/teasers/kotlinx-serialization.png
last_modified_at: 2024-03-07T00:00:00+09:00
---

서버에서 REST API를 구현하여 데이터를 다룰때는 Json 데이터 형태로 많이 다루게 된다. 안드로이드에서 서버의 데이터를 받아올때 편리하게 `Gson` 라이브러리를 활용하여 객체-Json 간의 Converter 작업을 하였었다. 이번에 구글 프로젝트를 보면서 새로운 직렬화/역직렬화를 다룰 수 있는 라이브러리를 보려고 한다.

# Kotlin의 Default Value
Gson으로도 데이터의 직렬화(역직렬화)를 할 수 있지만 코틀린의 데이터 클래스에서 Dafault Value를 사용하고 있다면 자바로 되어있는 Gson에서는 해당 기능을 제공하지 않기 때문에 NPE 가능성이 있다.

```kotlin
data class User(
    val name: String,
    val gender: String,
    val age: Int = 28,
    val hobby: String = "Work out"
)

val jsonString = """
    {
        "name" : "ppeper",
        "gender" : "Male"
    } 
""".trimIndent()

fun main() {
    val user = Gson().fromJson(jsonString, User::class.java)
    println(user)
}
```

> User(name=ppeper, gender=Male, age=0, hobby=null)

age의 값과 hoppy의 값에 Default value가 있지만 Primitive type의 경우 0, Reference type의 경우 null로 나오는 것을 볼 수 있다.   코틀린에서는 Null-Safety 를 통해 hobby가 Not-Null type이지만 null이 들어있어 개발시에 잘못한다면 NPE가 발생할 수 있다.

# Kotlinx Serialization
코틀린에서 공식적으로 제공하고 있는 __Kotlinx Serialization__ 라이브러리를 통해 직렬화/역직렬화를 사용할 수 있다. Kotlin에서 사용하면 위와 같은 Default value의 상황에서도 정상적으로 값을 보여줄 수 있으며 다른 Converter들과 다르게 Reflection을 사용하지 않고 KSerializer을 사용한다.

사용하는 방법은 변환하고자하는 Class에 `@Serializable` 어노테이션만 추가해 주면 된다.
이 어노테이션을 추가하면 플러그인이 자동으로 해당하는 클래스의 companion object에 `serializer()` 함수를 생성하여 사용할 수 있게 해준다. 이 함수는 __KSerializer__ 타입의 객체를 반환하여 직렬화할때 사용된다.

```kotlin
plugins {
    kotlin("plugin.serialization") version "1.9.22"
}

dependencies {
    implementation("org.jetbrains.kotlinx:kotlinx-serialization-json:$latest_version")
}

@Serializable
data class User(
    val name: String,
    val gender: String,
    val age: Int = 28,
    val hobby: String = "Work out"
)

val jsonString = """
    {
        "name" : "ppeper",
        "gender" : "Male"
    } 
""".trimIndent()

fun main() {
    val user = Json.decodeFromString<User>(jsonString)
    println(user)
}
```

> User(name=ppeper, gender=Male, age=28, hobby=Work out)

Default Value를 설정한 프로퍼티 값들이 성공적으로 출력되는 것을 볼 수 있다. kotlinx.serialization에서 Json 관련한 여러 옵션추가하여 입맛에 맞게 사용이 가능하다.

## ignoreKnownKeys
보통 Model로 다시 파싱을 할때 Json에 있는 Key값과 Model의 멤버 변수와 mapping이 되어햐 한다. 이때 Json에 있는 Key 값이 Model에 없을 때 이를 무시할 수 있는 `ignoreKnwonKeys` 옵션을 설정해 놓으면 Model에서 있는 멤버 변수의 값들만 파싱이 된다.

해당 옵션을 사용하지 않으면 Exception을 통해 알지 못하는 값을 출력해준다.
> Exception in thread "main" kotlinx.serialization.json.internal.JsonDecodingException: Unexpected JSON token at offset 53: Encountered an unknown key 'id' at path: $.gender
Use 'ignoreUnknownKeys = true' in 'Json {}' builder to ignore unknown keys.

```kotlin
val jsonString = """
    {
        "name" : "ppeper",
        "gender" : "Male",
        "id" : "1"
    } 
""".trimIndent()

private val json = Json { ignoreUnknownKeys = true }

fun main() {
    val decodeToString = json.decodeFromString<User>(jsonString)
    println(decodeToString)
}
```

> User(name=ppeper, gender=Male, age=28, hobby=Work out)

 이외에도 공식 [Kotlin Serialization](https://github.com/Kotlin/kotlinx.serialization/blob/master/docs/builtin-classes.md#enum-classes)을 보면 직렬화 대상 Model의 멤버 변수에 다양한 어노테이션을 활용할 수 있다. 또한 안드로이드에서 Retrofit을 사용하고 있다면 이를위한 [Converter](https://github.com/JakeWharton/retrofit2-kotlinx-serialization-converter)도 현재 제공하고 있다.

- - -

# References
- [https://kotlinlang.org/docs/serialization.html#what-s-next](https://kotlinlang.org/docs/serialization.html#what-s-next)
- [https://github.com/google/gson](https://github.com/google/gson)