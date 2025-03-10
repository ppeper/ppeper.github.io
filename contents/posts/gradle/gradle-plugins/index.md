---
title: "Gradle Convention Plugin 생성 및 적용하기"
date: 2023-12-26
update: 2023-12-26
tags:
  - Version Catalog
series: "Gradle"
---
<img src="https://github.com/ppeper/Kotlin_Algorithm/assets/63226023/ecbbb8e9-4775-4847-8485-76ef893b3453">

Gradle에서 최근에 많이 사용하는 버전관리는 `version-catalog` 를 통하여 하고 있다. 결국 version-catalog, BuildSrc 등은 의존성 버전, 관리에 대한 재사용성을 높여주고 멀티 모듈을 구성하게 되면 공통적으로 사용되는 코드 작성을 줄여주게 된다.

이번에 Now in Android 예시 프로젝트를 보면서 build-logic이 적용되어 있는 모듈을 한번 구성해 보면서 기본 프로젝트 생성시에 Android, Compose에 대한 Convention Plugin을 생성해보려고 한다.
- - -

# Convention Plugin 생성하기
최근 안드로이드 스튜디오에서는 version-catalog가 적용된 프로젝트로 생성이 가능하다. 해당 프로젝트를 생성 후 `build-logic` 으로 plugin 들을 관리할 모듈을 추가한다.

`settings.gradle.kts(Project Settings)` 에서 기본적으로 모듈이 추가되었을 때 자동 include되는 `include(":build-logic")` 의 경우는 삭제해 주고 __build-logic__ 에 convention 모듈을 하나 더 생성하여 사용하기 때문에 `includeBuild("build-logic")` 을 추가해 준다.

```kotlin
pluginManagement {
    // 추가 되는 내용
    includeBuild("build-logic")
    repositories {
        google()
        mavenCentral()
        gradlePluginPortal()
    }
}
dependencyResolutionManagement {
    repositoriesMode.set(RepositoriesMode.FAIL_ON_PROJECT_REPOS)
    repositories {
        google()
        mavenCentral()
    }
}

rootProject.name = "ConventionPlugin"
include(":app")
// include("build-logic") 삭제
```

build-logic 모듈에 `convention` 모듈(java or kotlin library)을 추가해주고 root에 `settins.gradle.kts` 파일을 다음과 같이 추가해 준다. (convention 모듈이 추가되었을 때 `include(":build-logic:convention")` 는 동일하게 삭제해 준다.)

```kotlin
// settins.gradle.kts
dependencyResolutionManagement {
    repositories {
        google()
        mavenCentral()
    }
    versionCatalogs {
        create("libs") {
            from(files("../gradle/libs.versions.toml"))
        }
    }
}

rootProject.name = "build-logic"
include(":convention")
```

<img src="https://github.com/ppeper/Kotlin_Algorithm/assets/63226023/7bc87c15-08ef-4e9b-8a30-fea9581138e5">

lib.versions.toml 파일에 아래의 버전을 추가하고 이를 build-logic:convention의 build.gradle.kts에 아래와 같이 수정한다.

```toml
// libs.versions.toml
androidGradlePlugin = "8.1.4"
androidTools = "31.2.0"
kotlin = "1.9.10"
.
.
# Dependencies of the included build-logic
android-gradlePlugin = { group = "com.android.tools.build", name = "gradle", version.ref = "androidGradlePlugin" }
android-tools-common = { group = "com.android.tools", name = "common", version.ref = "androidTools" }
kotlin-gradlePlugin = { group = "org.jetbrains.kotlin", name = "kotlin-gradle-plugin", version.ref = "kotlin" }
```

```kotlin
// build.gradle.kts
@Suppress("DSL_SCOPE_VIOLATION") // TODO: Remove once KTIJ-19369 is fixed
plugins {
    `kotlin-dsl`
}

java {
    sourceCompatibility = JavaVersion.VERSION_17
    targetCompatibility = JavaVersion.VERSION_17
}

dependencies {
    compileOnly(libs.android.gradlePlugin)
    compileOnly(libs.android.tools.common)
    compileOnly(libs.kotlin.gradlePlugin)
}
```

다음으로 Android의 기본 세팅을 도와주는 Extension 파일인 `KotlinAndroid.kt` 파일과 Compose 세팅을 위한 `AndroidCompose.kt` 파일을 생성해 준다.

<img src="https://github.com/ppeper/Kotlin_Algorithm/assets/63226023/3b3e5521-3677-4238-9dab-795971204841">

```kotlin
// KotlinAndroid.kt

internal fun Project.configureKotlinAndroid(
    commonExtension: CommonExtension<*, *, *, *, *>
) {
    commonExtension.apply {
        compileSdk = 34

        defaultConfig {
            minSdk = 26

            testInstrumentationRunner = "android.support.test.runner.AndroidJUnitRunner"
            vectorDrawables.useSupportLibrary = true
        }

        compileOptions {
            sourceCompatibility = JavaVersion.VERSION_17
            targetCompatibility = JavaVersion.VERSION_17
        }

        kotlinOptions {
            jvmTarget = JavaVersion.VERSION_17.toString()
        }
    }
}

fun CommonExtension<*, *, *, *, *>.kotlinOptions(block: KotlinJvmOptions.() -> Unit) {
    (this as ExtensionAware).extensions.configure("kotlinOptions", block)
}

// AndroidCompose.kt
// 기본적으로 프로젝트 생성시 적용되어있는 libs.versions.toml 파일에 있는 compose 관련 라이브러리 추가
internal fun Project.configureAndroidCompose(
    commonExtension: CommonExtension<*, *, *, *, *>
) {
    commonExtension.apply {
        val libs = extensions.getByType<VersionCatalogsExtension>().named("libs")

        buildFeatures.compose = true

        composeOptions {
            kotlinCompilerExtensionVersion =
                libs.findVersion("androidxComposeCompiler").get().toString()
        }

        dependencies {
            val bom = libs.findLibrary("compose-bom").get()
            add("implementation", platform(bom))
            add("androidTestImplementation", platform(bom))

            add("implementation", libs.findLibrary("activity-compose").get())
            add("implementation", libs.findLibrary("ui").get())
            add("implementation", libs.findLibrary("ui-graphics").get())
            add("implementation", libs.findLibrary("ui-tooling").get())
            add("implementation", libs.findLibrary("ui-tooling-preview").get())
            add("implementation", libs.findLibrary("material3").get())
            add("implementation", libs.findLibrary("ui-test-manifest").get())
        }
    }
}

// libs.versions.tomal
androidxComposeCompiler = "1.5.3"
.
.
[libraries]
.
.
compose-bom = { group = "androidx.compose", name = "compose-bom", version.ref = "compose-bom" }
```

## Convention Plugin 추가
이제 convention 모듈에서 build.gradle 설정 plugin인 `AndroidApplicationPlugin`, `AndroidApplicationComposePlugin` 파일을 추가해 준다.


<img src="https://github.com/ppeper/Kotlin_Algorithm/assets/63226023/524a0acb-2727-49b4-a832-842eb3349fc3">

```kotlin
// AndroidApplicationPlugin.kt
class AndroidApplicationPlugin: Plugin<Project> {
    override fun apply(target: Project) {
        with(target) {
            with(pluginManager) {
                apply("com.android.application")
                apply("org.jetbrains.kotlin.android")
            }

            extensions.configure<ApplicationExtension> {
                // 앞서 생성한 Android setting을 도와주는 extention
                configureKotlinAndroid(this)
                defaultConfig {
                    versionCode = 1
                    versionName = "1.0"
                    targetSdk = 34
                }
            }   
        }
    }
}

// AndroidApplicationComposePlugin.kt
class AndroidApplicationComposePlugin: Plugin<Project> {
    override fun apply(target: Project) {
        with(target) {
            pluginManager.apply("com.android.application")

            extensions.configure<ApplicationExtension> {
                // 앞서 생성한 Compose setting을 도와주는 extension
               configureAndroidCompose(this)
            }
        }
    }
}
```
이번 예제에서는 두 가지의 Plugin들만 추가하였지만 여러가지 Hilt, Room과 같은 라이브러리들을 관리해주도록 필요할 때마다 추가하여 관리하면 된다.

이제 만들어진 Plugin들을 build.gradle파일에 세팅을 해주면 사용준비가 완료된다!

```kotlin
.
.
dependencies {
    compileOnly(libs.android.gradlePlugin)
    compileOnly(libs.android.tools.common)
    compileOnly(libs.kotlin.gradlePlugin)
}

gradlePlugin {
    plugins {
        register("AndroidApplicationPlugin") {
            id = "ppeper.example.application"
            implementationClass = "AndroidApplicationPlugin"
        }
    }
    plugins {
        register("AndroidApplicationComposePlugin") {
            id = "ppeper.example.compose"
            implementationClass = "AndroidApplicationComposePlugin"
        }
    }
}
```

여기서 id의 값은 플러그인 이름으로 사용하고 싶은 이름으로 지정하면 되고, implementationClass는 매칭되는 해당 Class의 이름이다.

## 사용하기
app 모듈에서 Android, Compose 세팅을 위해 생성한 Plugin을 도입하면 아래와 같이 사용할 수 있다.
```kotlin
// Before
@Suppress("DSL_SCOPE_VIOLATION") // TODO: Remove once KTIJ-19369 is fixed
plugins {
    alias(libs.plugins.com.android.application)
    alias(libs.plugins.org.jetbrains.kotlin.android)
}

android {
    namespace = "com.ppeper.conventionplugin"
    compileSdk = 34

    defaultConfig {
        applicationId = "com.ppeper.conventionplugin"
        minSdk = 24
        targetSdk = 34
        versionCode = 1
        versionName = "1.0"

        testInstrumentationRunner = "androidx.test.runner.AndroidJUnitRunner"
        vectorDrawables {
            useSupportLibrary = true
        }
    }

    buildTypes {
        release {
            isMinifyEnabled = false
            proguardFiles(
                getDefaultProguardFile("proguard-android-optimize.txt"),
                "proguard-rules.pro"
            )
        }
    }
    compileOptions {
        sourceCompatibility = JavaVersion.VERSION_17
        targetCompatibility = JavaVersion.VERSION_17
    }
    kotlinOptions {
        jvmTarget = "1.8"
    }
    buildFeatures {
        compose = true
    }
    composeOptions {
        kotlinCompilerExtensionVersion = "1.4.3"
    }
    packaging {
        resources {
            excludes += "/META-INF/{AL2.0,LGPL2.1}"
        }
    }
}

dependencies {

    implementation(libs.core.ktx)
    implementation(libs.lifecycle.runtime.ktx)
    implementation(libs.activity.compose)
    implementation(platform(libs.compose.bom))
    implementation(libs.ui)
    implementation(libs.ui.graphics)
    implementation(libs.ui.tooling.preview)
    implementation(libs.material3)
    testImplementation(libs.junit)
    androidTestImplementation(libs.androidx.test.ext.junit)
    androidTestImplementation(libs.espresso.core)
    androidTestImplementation(platform(libs.compose.bom))
    androidTestImplementation(libs.ui.test.junit4)
    debugImplementation(libs.ui.tooling)
    debugImplementation(libs.ui.test.manifest)
}

// plugin 적용 후
@Suppress("DSL_SCOPE_VIOLATION") // TODO: Remove once KTIJ-19369 is fixed
plugins {
    id("ppeper.example.application")
    id("ppeper.example.compose")
}

android {
    namespace = "com.ppeper.conventionplugin"

    buildTypes {
        release {
            isMinifyEnabled = false
            proguardFiles(
                getDefaultProguardFile("proguard-android-optimize.txt"),
                "proguard-rules.pro"
            )
        }
    }
    packaging {
        resources {
            excludes += "/META-INF/{AL2.0,LGPL2.1}"
        }
    }
}

dependencies {
    // 기존에 있는 compose 라이브러리들은 삭제 가능
    implementation(libs.core.ktx)
    implementation(libs.lifecycle.runtime.ktx)
    testImplementation(libs.junit)
    androidTestImplementation(libs.androidx.test.ext.junit)
    androidTestImplementation(libs.espresso.core)
    androidTestImplementation(libs.ui.test.junit4)
}
```

Convention Plugin들을 생성하고 적용해 보면서 멀티모듈을 구성할때 각 필요한 라이브러리를 plugin들로 관리하면 새로운 모듈이 추가될때 재사용성 있고 중복코드 없이 관리 할 수 있어 한번 적용을 해보면 좋겠다 생각이 들었다.

- - -
# References
- [https://brunch.co.kr/@purpledev/46](https://brunch.co.kr/@purpledev/46)
- [https://github.com/android/nowinandroid](https://github.com/android/nowinandroid)