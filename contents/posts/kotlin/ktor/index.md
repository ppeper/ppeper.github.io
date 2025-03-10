---
title: "KMP를 지원하는 Ktor Client를 사용해보자"
date: 2024-12-04
update: 2024-12-04
tags:
  - Android
  - Kotlin Multiplatform
series: "Kotlin"
---
안드로이드에서는 Rest Api 통신을 편리하게 해주는 `Retrofit2` , `Okhttp` 라이브러리를 사용을 현재까지도 많이 사용하고 있다. 최근 Kotlin Multiplatform을 사용하게 되면서 Retrofit을 사용하지 못하게 되어 말로만 듣던 Ktor를 살펴보고자 한다.

# Ktor?

Ktor는 Kotlin으로만 작성된 경량 웹 프레임워크이다.  따라서 플랫폼에 종속적이지 않고 Multiplatform 환경에서 사용이 가능하다.

Ktor는 Server와 Client 두 모듈을 모두 제공을 하고 있으며 Android 및 Multiplatform에서 사용하기 위해 Client 부분을 알아보려고 한다.

1. 먼저 ktor를 사용하기 위해 gradle에 등록해준다.

```groovy
ktor = <latest_version>

implementation("io.ktor:ktor-client-core:$ktor")
implementation("io.ktor:ktor-client-cio:$ktor")
```

Ktor에서 지원하는 여러 옵션들이 있지만 공식문서에서 말하는  라이브러리를 살펴보면 다음과 같다.

- `core` :  클라이언트 함수들을 제공하는 모듈
- `CIO` : 플랫폼 별로 요청을 처리하는 엔진에 관한 모듈

| Engine | Platforms |
| --- | --- |
| Apache | JVM |
| Java | JVM |
| Jetty | JVM |
| Android | JVM, Android |
| OkHttp | JVM, Android |
| Darwin | Native |
| WinHttp | Native |
| Curl | Native |
| CIO | JVM, Android, Native |
| Js | JavaScript |

간단한 예시로 네트워크 통신에서 필요한 직렬화는 Ktor에서 직렬화/역직렬화를 위해서 사용되는 `negotiation` 라이브러리와 `Kotlin-serialization` 을 사용하여 요청/응답 값을 파싱할 수 있고 이는 Ktor는 HttpClient 생성할때 `install` 함수를 사용하면 된다.

[(Ktor 공식 문서](https://ktor.io/docs/client-create-new-application.html)에는 다양한 옵션, 함수들을 제공하고 있어 필요한 내용을 학습하고 적용하면 될것 같다!)

> 💡 많이 사용하는 Retrofit2 라이브러리와 같이 어노테이션 기반으로 사용할 수 있는 [Ktorfit 라이브러리](https://foso.github.io/Ktorfit/) 도 지원하고 있다!
> 

```groovy
implementation("io.ktor:ktor-client-content-negotiation:$ktor")
implementation("io.ktor:ktor-serialization-kotlinx-json:$ktor")
```

```kotlin
val client = HttpClient {
	install(ContentNegotiation) {
    json(Json {
        isLenient = true
        ignoreUnknownKeys = true
    })
	}
	install(HttpTimeout) {
    connectTimeoutMillis = 5_000L
    socketTimeoutMillis = 5_000L
    requestTimeoutMillis = 5_000L
	}
}
```

이렇게 생성된 client를 통해 HttpRequest들을 보낼 수 있다. 공식문서의 get 요청을 하나 만들어 보면 다음과 같다.

```kotlin
// 많은 함수들이 존재 (ex. request, post, delete...)
public suspend inline fun HttpClient.get(
    urlString: String,
    block: HttpRequestBuilder.() -> Unit = {}
): HttpResponse = get { url(urlString); block() }

// ktor docs를 가져오도록 하자
suspend fun greet(): String {
    val response = client.get("https://ktor.io/docs")
    return "[Response]\n${response.bodyAsText()}"
}
```

이제 Kotlin Multiplatform Wizard로 생성한 프로젝트에 Desktop에서 해당 요청을 사용해 보자!

```kotlin
@Composable
@Preview
fun App() {
    MaterialTheme {
        var showContent by remember { mutableStateOf(false) }
        Column(Modifier.fillMaxWidth(), horizontalAlignment = Alignment.CenterHorizontally) {
            Button(onClick = { showContent = !showContent }) {
                Text("Ktor Click me!")
            }
            AnimatedVisibility(showContent) {
                val scope = rememberCoroutineScope()
                var text by remember { mutableStateOf("Loading") }
                Column(
                    Modifier.fillMaxWidth(),
                    horizontalAlignment = Alignment.CenterHorizontally
                ) {
                    LaunchedEffect(true) {
                        scope.launch {
                            text = try {
		                            // Get 요청 
                                KtorExample().greet()
                            } catch (e: Exception) {
                                e.message ?: "UnKnown Error"
                            }
                        }
                    }
                    Text(text)
                }
            }
        }
    }
}
```

<img src="https://github.com/user-attachments/assets/48b1d777-3e20-4560-8537-5d251023b820">

- - -
# References

- [https://ktor.io/docs/welcome.html](https://ktor.io/docs/welcome.html)
- [https://foso.github.io/Ktorfit/](https://foso.github.io/Ktorfit/)