---
title: 안드로이드 Compose 알아보기
categories: Android
tags: ['Android', 'Compose']
header:
    teaser: /assets/teasers/android_compose_image.png
last_modified_at: 2023-08-21T00:00:00+09:00
---
<img src="https://github.com/ppeper/Kotlin_Algorithm/assets/63226023/16e97d30-b69e-4621-b022-94f1e8184b24">

최근에 Compose를 적용하여 프로젝트를 진행하면서 하나씩 만들어 보았는데 해당 내용들을 정리해 보려고 한다.
- - -
# Compose
Compose는 Kotlin언어를 사용하여 `선언형 프로그래밍` 방식으로 UI를 그리는 방식으로 기존의 xml의 방식과는 차이가 있다.

Compose를 사용하면 아래와 같은 장점이 있다.
- __코드 감소__
    - Kotlin, xml로 나눠서 개발하는 것이 아닌 Kotlin 언어만으로 개발이 가능하다.
- 선언적 API를 사용하여 __직관적__ 이다.
- __빠른 개발 과정__
    - Compose는 기존의 모든 코드와 호환되고 Compose에서 Views를, Views에서 Compose 코드를 호출할 수 있다.
- __강력한 성능__

## @Composable
- 구성가능한 함수는 @Composable annotation이 달린 일반함수
- Compose 함수는 대문자로 시작하도록 가이드 한다.

## @Preview
- 적용된 화면을 미리보기 가능

# Row, Column, Box

<img src="https://github.com/ppeper/Kotlin_Algorithm/assets/63226023/35e94a8a-b88e-4385-8984-219535d9b570">



Compose의 기본적인 Layout은 다음과 같이 세가지가 있다. `Row`와 `Column`은 xml에서 LinearLayout,   
`Box`는 FrameLayout으로 볼 수 있다.

## Row
- Row는 행으로 UI가 배치되는 Layout이다.

```kotlin
@Preview
@Composable
fun PreviewMyRow() {
    Row {
        Text(text = "Test1")
        Text(text = "Test2")
        Text(text = "Test3")
    }
}
```

<img src="https://github.com/ppeper/Kotlin_Algorithm/assets/63226023/f23f636d-dbd1-4d9e-87ae-dd504564f40b">

## Column
- Colunm는 열로 UI가 배치되는 Layout이다.

```kotlin
@Preview
@Composable
fun PreviewMyColumn() {
    Column {
        Text(text = "Test1")
        Text(text = "Test2")
        Text(text = "Test3")
    }
}
```

<img src="https://github.com/ppeper/Kotlin_Algorithm/assets/63226023/c524cd0b-8796-4f23-92fe-48f7023ae14a">

## Box
- Box는 위젯을 다른 화면 위에 배치시키는 Layout이다.

```kotlin
@Preview
@Composable
fun PreviewMyBox() {
    Box(
        modifier = Modifier.fillMaxWidth() // 화면 너비 꽉 채우기
            .height(200.dp) // 높이 200dp 설정
    ) {
        Text(modifier = Modifier.align(Alignment.TopStart), text = "Test1")
        Text(modifier = Modifier.align(Alignment.Center), text = "Test2")
        Text(modifier = Modifier.align(Alignment.BottomEnd), text = "Test3")
    }
}
```

<img src="https://github.com/ppeper/Kotlin_Algorithm/assets/63226023/65d9bbc1-5fa1-4e3d-81c4-e0c521326948"/>

> 🧐 위에서 Modifer와 align은 무엇일까?

## Modifier
- Surface, Text와 같은 대부분의 Compose UI Component는 modifier 매개변수를 선택적으로 허용한다.
- Modifier는 상위 Component Layout내에서 UI요소가 배치되고 표시괴는 동작 방식을 지정한다.

## 정렬 (Alignment)
- Column, Row에서의 Alignment
    - Column: `Start`, `CenterHorizontally`, `End`
    - Row: `Top`, `CenterVertically`, `Bottom`
- Box에서 Alignment
    - `TopStart`
    - `TopCenter`
    - `TopEnd`
    - `CenterStart`
    - `Center`
    - `CenterEnd`
    - `BottomStart`
    - `BottomCenter`
    - `BottomEnd`

## 배치 (Arrangement)

<img src="https://github.com/ppeper/Kotlin_Algorithm/assets/63226023/0c49feda-906c-491f-8276-c075b901445f">

📌 정렬과 배치를 사용하면 원하는대로 UI를 구성할 수 있다.

```kotlin
@Composable
fun MyRow() {
    Row(
        modifier = Modifier.size(200.dp),
        horizontalArrangement = Arrangement.SpaceAround,
        verticalAlignment = Alignment.Bottom
    ) {
        Text(text = "Test1")
        Text(text = "Test2")
        Text(text = "Test3")
    }
}
```
<img src="https://github.com/ppeper/Kotlin_Algorithm/assets/63226023/7dfc0090-c96d-4768-b9e0-d54f19cf2c29">

```kotlin
@Preview
@Composable
fun PreviewMyColumn() {
    Column(
        modifier = Modifier.size(100.dp),
        verticalArrangement = Arrangement.SpaceBetween,
        horizontalAlignment = Alignment.Start
    ) {
        Text(text = "Test1")
        Text(text = "Test2")
        Text(text = "Test3")
    }
}
```

<img src="https://github.com/ppeper/Kotlin_Algorithm/assets/63226023/aac7917f-ee2c-4bbe-b7ef-3e217bbdc055">

- - -
# References
- [Jetpack Compose로 더 빠르게 더 나은 Android 앱 빌드](https://developer.android.com/jetpack/compose?gclid=CjwKCAjwloynBhBbEiwAGY25dNfCR8uKXeCY7EMNwc06CxF06lWnhhGDpVKV0SLdpeEGcB9cCrxnkhoCKn8QAvD_BwE&gclsrc=aw.ds&hl=ko)
