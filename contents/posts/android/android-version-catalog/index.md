---
title: "안드로이드 Version Catalog 도입하기"
date: 2023-10-30
update: 2023-10-30
tags:
  - Android
  - Version Catalog
series: "Android"
---
<img src="https://github.com/ppeper/Cellification/assets/63226023/acb3db44-6459-43b8-918c-3e49e7f6c204">

안드로이드에서는 gradle 파일을 통하여 버전관리를 하고 있다. 프로젝트를 진행하면서 같은 implement를 해야하는 라이브러리들을 관리를 하지 못하여 유지보수에 좋지 못하다는 점에 있어서 공통된 파일로 관리하면 좋겠다는 생각이 들었다.

이번에 Gradle 7.0에서 추가된 `Version Catalog`가 있다는 것을 알게되어 이를 통하여 간편하게 버전관리를 하는 방법을 공부하고 내용을 정리해 보려고 한다.

# Version Catalog
Version Catalog는 하나의 파일로 버전을 관리하여 여러 프로젝트나 모듈들의 의존성을 통합하여 관리할 수 있다는 장점이 있다.

> 최근에 업데이트된 안드로이드 스튜디오 Giraffe 버전에서는 __Kotlin DSL__ 을 기본 빌드 방식으로 Recommend하여 groovy가 아닌 Kotlin 언어로 빌드를 하도록 하고 있다.

Version Catalog는 파일형식으로 `TOML`를 사용하여 기본적으로 `키 = 값` 쌍, `[섹션 이름]` 과 `#`을 통한 주석을 관리한다. Version Catalog에서 사용하는 섹션 이름은 아래와 같다.

- `[versions]` : 라이브러리의 버전들에 대한 명시
- `[plugins]` : 플러그인에 대한 명시
- `[libraries]` : 각각의 라이브러리 의존성에 대한 명시
- `[bundles]` : 라이브러리를 묶어서 한 번에 선언이 가능하도록 하는 명시

## 생성하기
먼저 `gradle` 폴더에 `libs.versions.toml` 파일을 생성한다.

<img src="https://github.com/ppeper/ppeper/assets/63226023/666efcc3-b1eb-4d34-84a4-4370634c5909">

Version Catalog는 Gradle 7.4 버전부터 정식으로 지원되어, 이전 버전을 사용한다면 수동으로 toml 파일을 찾아서 지정을 해줘야한다.

```kotlin
// In settins.gradle.kts

enableFeaturePreview("VERSION_CATALOGS")

dependencyResolutionManagement {
    versionCatalogs {
        create("libs") {
            files(files("libs.versions.toml"))
        }
    }
}
```

> 파일을 gradle 폴더의 루트 디렉토리에 생성하면 경로를 명시하지 않아도 된다.

```toml
[versions]
activityCompose = "1.7.2"
androidGradlePlugin = "8.1.1"
kotlin = "1.8.10"
androidxActivity = "1.7.2"
androidxLifecycle = "2.6.1"
androidxHiltNavigationCompose = "1.0.0"
androidxNavigation = "2.7.1"
androidxAppCompat = "1.6.1"
androidxArchCore = "2.2.0"
androidxCoreSplashscreen = "1.0.0"
androixPagingCompose = "1.0.0-alpha18"
androidxComposeBom = "2023.09.00"
androidxComposeCompiler = "1.4.3"
androidXComposeUiGraphics = "1.5.1"
androidxComposeMaterial3 = "1.2.0-alpha07"
androidxComposeRuntimeTracing = "1.0.0-alpha01"
androidxCore = "1.9.0"
kapt = "1.8.10"
ksp = "1.8.10-1.0.9"
junit4 = "4.13.2"
androidxTestExtJunit = "1.1.5"
androidxTestCore = "1.4.0"
androidxTestExt = "1.1.5"
androidxEspresso = "3.5.1"
lifecycleRuntimeKtx = "2.6.1"
material = "1.8.0"
parcelize = "1.8.0"
sdp = "1.1.0"
hilt = "2.45"
hiltJavax = "1"
hiltCompiler = "2.45"
kotlinxCoroutines = "1.7.1"
okhttp = "4.10.0"
retrofit = "2.9.0"
room = "2.5.2"
coil = "2.4.0"
lottie = "6.1.0"

[libraries]

# Android
androidx-appcompat = { group = "androidx.appcompat", name = "appcompat", version.ref = "androidxAppCompat" }
androidx-core-ktx = { group = "androidx.core", name = "core-ktx", version.ref = "androidxCore" }
androidx-lifecycle-runtime-ktx = { group = "androidx.lifecycle", name = "lifecycle-runtime-ktx", version.ref = "androidxLifecycle" }
material = { group = "com.google.android.material", name = "material", version.ref = "material" }

# Compose
androidx-compose-material3 = { group = "androidx.compose.material3", name = "material3", version.ref = "androidxComposeMaterial3" }

androidx-navigation-compose = { group = "androidx.navigation", name = "navigation-compose", version.ref = "androidxNavigation" }
androidx-lifecycle-runtime-compose = { group = "androidx.lifecycle", name = "lifecycle-runtime-compose", version.ref = "androidxLifecycle" }

androidx-activity-compose = { group = "androidx.activity", name = "activity-compose", version.ref = "androidxActivity" }
androidx-compose-bom = { group = "androidx.compose", name = "compose-bom", version.ref = "androidxComposeBom" }
androidx-compose-ui = { group = "androidx.compose.ui", name = "ui" }
androidx-compose-ui-graphics = { group = "androidx.compose.ui", name = "ui-graphics", version.ref = "androidXComposeUiGraphics" }
androidx-compose-ui-tooling = { group = "androidx.compose.ui", name = "ui-tooling" }
androidx-compose-ui-tooling-preview = { group = "androidx.compose.ui", name = "ui-tooling-preview" }
androidx-compose-runtime = { group = "androidx.compose.runtime", name = "runtime" }
androidx-compose-runtime-livedata = { group = "androidx.compose.runtime", name = "runtime-livedata" }
androidx-lifecycle-viewmodel-compose = { group = "androidx.lifecycle", name = "lifecycle-viewmodel-compose", version.ref = "androidxLifecycle" }
androidx-paging-compose = { group = "androidx.paging", name = "paging-compose", version.ref = "androixPagingCompose" }

#OKHTTP
okhttp-bom = { group = "com.squareup.okhttp3", name = "okhttp-bom", version.ref = "okhttp" }
okhttp = { group = "com.squareup.okhttp3", name = "okhttp" }
okhttp-logging-interceptor = { group = "com.squareup.okhttp3", name = "logging-interceptor" }

#RETROFIT
retrofit = { group = "com.squareup.retrofit2", name = "retrofit", version.ref = "retrofit" }
retrofit-converter-gson = { group = "com.squareup.retrofit2", name = "converter-gson", version.ref = "retrofit" }

#ROOM
room-ktx = { group = "androidx.room", name = "room-ktx", version.ref = "room" }
room-compiler = { group = "androidx.room", name = "room-compiler", version.ref = "room" }
room-paging = { group = "androidx.room", name = "room-paging", version.ref = "room" }
room-runtime = { group = "androidx.room", name = "room-runtime", version.ref = "room" }

#TEST
androidx-arch-core-testing = { group = "androidx.arch.core", name = "core-testing", version.ref = "androidxArchCore" }
androidx-test-core = { group = "androidx.test", name = "core", version.ref = "androidxTestCore" }
androidx-test-espresso-core = { group = "androidx.test.espresso", name = "espresso-core", version.ref = "androidxEspresso" }
androidx-compose-ui-test = { group = "androidx.compose.ui", name = "ui-test-junit4" }
androidx-compose-ui-test-manifest = { group = "androidx.compose.ui", name = "ui-test-manifest" }
androidx-test-ext-junit = { group = "androidx.test.ext", name = "junit", version.ref = "androidxTestExt" }
junit4 = { group = "junit", name = "junit", version.ref = "junit4" }

# Hilt
hilt-android = { group = "com.google.dagger", name = "hilt-android", version.ref = "hilt" }
hilt-compiler = { group = "com.google.dagger", name = "hilt-android-compiler", version.ref = "hilt" }
hilt-navigation-compose = { group = "androidx.hilt", name = "hilt-navigation-compose", version.ref = "androidxHiltNavigationCompose" }
hilt-javax = { group= "javax.inject", name = "javax.inject", version.ref = "hiltJavax" }

#COROUTINE
kotlinx-coroutines-android = { group = "org.jetbrains.kotlinx", name = "kotlinx-coroutines-android", version.ref = "kotlinxCoroutines" }
kotlinx-coroutines-test = { group = "org.jetbrains.kotlinx", name = "kotlinx-coroutines-test", version.ref = "kotlinxCoroutines" }

#Sdp
sdp = { group = "com.github.Kaaveh", name = "sdp-compose", version.ref = "sdp" }

#Coil
coil = { group = "io.coil-kt", name = "coil-compose", version.ref = "coil" }
coil-gif = { group = "io.coil-kt", name = "coil-gif", version.ref = "coil" }

#Lottie
lottie = { group = "com.airbnb.android", name = "lottie-compose", version.ref = "lottie" }

[plugins]

android-application = { id = "com.android.application", version.ref = "androidGradlePlugin" }
android-library = { id = "com.android.library", version.ref = "androidGradlePlugin" }
android-test = { id = "com.android.test", version.ref = "androidGradlePlugin" }
kotlin-android = { id = "org.jetbrains.kotlin.android", version.ref = "kotlin" }
kapt = { id = "org.jetbrains.kotlin.kapt", version.ref = "kapt" }
ksp = { id = "com.google.devtools.ksp", version.ref = "ksp" }
kotlin-jvm = { id = "org.jetbrains.kotlin.jvm", version.ref = "kotlin" }
hilt = { id = "com.google.dagger.hilt.android", version.ref = "hilt" }
parcelize = { id = "org.jetbrains.kotlin.plugin.parcelize", version.ref = "parcelize" }

[bundles]
okhttp = ["okhttp", "okhttp-bom", "okhttp-logging-interceptor"]
retrofit = ["retrofit", "retrofit-converter-gson"]
coil = ["coil", "coil-gif"]
```

- toml에서 각 versions, libraries, bundles들을 관리할 수 있다.

```groovy
dependencies {

    implementation(libs.androidx.core.ktx)
    implementation(libs.androidx.lifecycle.runtime.ktx)
    implementation(libs.androidx.activity.compose)
    implementation(libs.androidx.navigation.compose)
    implementation(platform(libs.androidx.compose.bom))
    implementation(libs.androidx.compose.ui)
    implementation(libs.androidx.compose.ui.graphics)
    implementation(libs.androidx.compose.ui.tooling.preview)
    implementation(libs.androidx.compose.material3)
    testImplementation(libs.junit4)
    androidTestImplementation(libs.androidx.test.ext.junit)
    androidTestImplementation(libs.androidx.test.espresso.core)
    androidTestImplementation(platform(libs.androidx.compose.bom))
    androidTestImplementation(libs.androidx.compose.ui.test)
    debugImplementation(libs.androidx.compose.ui.test.manifest)
    debugImplementation(libs.androidx.compose.ui.tooling)
    // Hilt
    implementation(libs.hilt.android)
    implementation(libs.hilt.navigation.compose)
    kapt(libs.hilt.compiler)
    // ViewModel
    implementation(libs.androidx.lifecycle.viewmodel.compose)
    // collectWithLifeCycle
    implementation(libs.androidx.lifecycle.runtime.compose)
    // Compose Paging
    implementation(libs.androidx.paging.compose)
    // Coil
    implementation(libs.bundles.coil)
    // Sdp
    implementation(libs.sdp)
    // lottie
    implementation(libs.lottie)
}
```

> 위의 적용한 부분을 보게 되면 version에 대해서는 통합하여 관리를 할 수 있지만 여전히 gradle 파일 내에서만 라이브러리 의존성을 정의해야 한다는 것을 볼 수 있다.

# References
- [https://docs.gradle.org/current/userguide/platforms.html](https://docs.gradle.org/current/userguide/platforms.html)
- [https://ko.wikipedia.org/wiki/TOML](https://ko.wikipedia.org/wiki/TOML)