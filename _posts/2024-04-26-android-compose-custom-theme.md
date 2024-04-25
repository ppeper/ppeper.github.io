---
title: Compose 나만의 Custom Theme 설정하기
categories: Android
tags: ['Android', 'Compose']
header:
    teaser: /assets/teasers/compose-custom-theme.png
last_modified_at: 2024-04-26T00:00:00+09:00
---
<img src="https://github.com/ppeper/Kotlin_Algorithm/assets/63226023/3ee7a814-cc90-42ba-9411-6a20ff29c970">

안드로이드에서는 Material Design을 사용하여 컬러를 적용하여 라이트, 다크모드를 지원하는 앱을 만들 수 있게 해주고 있습니다.

하지만 Material의 지정된 테마가 한정적이고, 디자이너가 만든 색상의 네임을 사용해야 하는 경우에는 사용자 지정 테마를 만들어야 합니다. 이번에 Compose에서 사용자 지정 테마를 적용하는 방법을 하나씩 살펴보도록 하겠습니다!

# 📍 Compose Theme
안드로이드 스튜디오에서 기본적으로 Compose 프로젝트를 만들게 되면 `Color`, `Theme`, `Type`의 기본 값들이 생성되어 있습니다.

## 1. Color
앱에서 사용할 라이트,다크모드에 사용할 색상들을 지정하고 있습니다.

```kotlin
val Purple80 = Color(0xFFD0BCFF)
val PurpleGrey80 = Color(0xFFCCC2DC)
val Pink80 = Color(0xFFEFB8C8)

val Purple40 = Color(0xFF6650a4)
val PurpleGrey40 = Color(0xFF625b71)
val Pink40 = Color(0xFF7D5260)
```

## 2. Typography

```kotlin
// Set of Material typography styles to start with
val Typography = Typography(
    bodyLarge = TextStyle(
        fontFamily = FontFamily.Default,
        fontWeight = FontWeight.Normal,
        fontSize = 16.sp,
        lineHeight = 24.sp,
        letterSpacing = 0.5.sp
    )
    ...
)
```

## 3. Theme
```kotlin
private val DarkColorScheme = darkColorScheme(
    primary = Purple80,
    secondary = PurpleGrey80,
    tertiary = Pink80
    ...
)

private val LightColorScheme = lightColorScheme(
    primary = Purple40,
    secondary = PurpleGrey40,
    tertiary = Pink40
    ...
)
```

`Theme.kt` 에서는 라이트, 다크모드에서 사용할 ColorScheme가 지정되어 있습니다. 이 메소드에서는 parameter를 따로 채워주지 않으면 기본 값이 설정되게 됩니다.

시스템의 모드에 따라 colors를 정의하고 `MaterialTheme`에 해당 인자들을 넘겨줘 사용하고 있다는 것을 볼 수 있습니다.

```kotlin
@Composable
fun CustomTheme(
    darkTheme: Boolean = isSystemInDarkTheme(),
    dynamicColor: Boolean = true,
    content: @Composable () -> Unit
) {
    val colorScheme = when {
        dynamicColor && Build.VERSION.SDK_INT >= Build.VERSION_CODES.S -> {
            val context = LocalContext.current
            if (darkTheme) dynamicDarkColorScheme(context) else dynamicLightColorScheme(context)
        }

        darkTheme -> DarkColorScheme
        else -> LightColorScheme
    }
    .
    .
    MaterialTheme(
        colorScheme = colorScheme,
        typography = Typography,
        content = content
    )
}
```

# Theme 만들어보기
위에서 봤던 컬러, 폰트등을 사용자 지정 테마로 만들어서 사용해 보도록 하겠습니다.

먼저 Material에서 `lightColorScheme`, `darkColorScheme`을 사용했던 것과 같이 커스텀 ColorScheme를 만들어야 합니다.

Color들이 시스템 모드에 맞게 변경된다면 Recomposition이 일어나야하므로 지정한 color들을 State로 감싸주는 작업이 필요합니다.

## 사용자 Color
```kotlin
val primary = Color(0xFFB4BDE9)
val primaryDark = Color(0xFF6E81E4)

val background = Color(0xFFE2E2E2)
val backgroundDark = Color(0xFF3D3D3D)


class CustomColorScheme(
    primary: Color,
    background: Color,
    .
    .
) {
    var primary by mutableStateOf(primary)
        private set
    var background by mutableStateOf(background)
        private set
    .
    .
}

val customLightColorScheme: RickColorScheme by lazy {
    RickColorScheme(
        primary = primary,
        background = background,
        .
        .
    )
}

val customDarkColorScheme: RickColorScheme by lazy {
    RickColorScheme(
        primary = primaryDark,
        background = backgroundDark,
        .
        .
    )
}
```
## 사용자 Typography

```kotlin
private val pretendard = FontFamily(
    Font(R.font.pretendard_regular, FontWeight.Normal),
    Font(R.font.pretendard_medium, FontWeight.Medium),
    Font(R.font.pretendard_semi_bold, FontWeight.SemiBold)
    .
    .
)

// FontFamily가 FontWeight에 따라 선택된다.
val title01 = TextStyle(
    fontSize = 18.sp,
    fontWeight = FontWeight.Bold,
    fontFamily = pretendard
)

val title02 = TextStyle(
    fontSize = 14.sp,
    fontWeight = FontWeight.SemiBold,
    fontFamily = pretendard
)
.
.
data class CustomTypography(
    val title01: TextStyle = title01,
    val title02: TextStyle = title02,
)
```
만들어준 ColorScheme 및 Typography들은 `CompositionLocalProvider` 를 통해 제공할 수 있습니다.

```kotlin
@Composable
fun CustomTheme(
    darkTheme: Boolean = isSystemInDarkTheme(),
    content: @Composable () -> Unit
) {

    val typography = CustomTypography()
    val currentColor = remember {
        if (darkTheme) {
            customDarkColorScheme
        } else {
            customLightColorScheme
        }
    }

    CompositionLocalProvider(
        LocalColorScheme provides currentColor
        LocalTypography provides typography
    ) {
        ProvideTextStyle(typography.title01, content = content)
    }
}

val LocalColorScheme = staticCompositionLocalOf { customLightColorScheme }
val LocalTypography = staticCompositionLocalOf { CustomTypography() }

```

## 테마 값 사용하기
__CompositionLocalProvider__ 로 주입된 color나 typography는 `LocalColorScheme.current` 를 통해 최신 값을 가져올 수 있습니다.

이를 `object` 로 감싸 더 쉽게 사용할 수 있도록 만들어 줍니다.

```kotlin
object CustomTheme {
    val colors: CustomColorScheme
        @Composable
        @ReadOnlyComposable
        get() = LocalColorScheme.current

    val typography: CustomTypography
        @Composable
        @ReadOnlyComposable
        get() = LocalTypography.current
}
```
> @ReadOnlyComposable : 읽기에 최적화되게 만들어주는 어노테이션

간단에게 테스트를 위해 color값을 설정해 보면 시스템 모드에 따라 색상이 설정되는 것을 볼 수 있습니다 👋🏼

```kotlin
Column(
    modifier = modifier
        .fillMaxSize()
        .background(color = CustomTheme.colors.background)
) {
    ...
}
```

<img src="https://github.com/ppeper/Kotlin_Algorithm/assets/63226023/09855cf3-7793-42a8-ab19-441ce43df544">

- - -
# References
- [https://betterprogramming.pub/jetpack-compose-custom-themes-b1836877981d](https://betterprogramming.pub/jetpack-compose-custom-themes-b1836877981d)