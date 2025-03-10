---
title: "안드로이드 인텐트의 개념"
date: 2022-01-09
update: 2022-01-09
tags:
  - Android
  - Intent
series: "Android"
---
- - -
# Intent??
인텐트는 하나의 액티비티가 다른 액티비티를 시작할 수 있는 메시징 시스템이며, 이때 `android.content.Intent` 클래스 인스턴스를 사용한다. 액티비티가 안드로이드 런타임에 인텐트를 요청하면 해당 인텐트에 부합되는 액티비티를 안드로이드 런타임이 찾아서 시작한다.

# Intent 유형
인텐트의 유형에는 `명시적` 인텐트와 `암시적` 인텐트가 있다.
- __명시적 인텐트__ : 클래스 이름으로 액티비티를 참조하여 특정 액티비티의 시작을 안드로이드 런타임에 요청한다.
- __암시적 인텐트__ : 우리가 하기를 원하는 작업(액션 타입)을 안드로이드 런타임에 알려준다. 그러면 그런 작업을 할 수 있다고 자신을 등록한 액티비티를 안드로이드 런타임이 찾아서 시작한다.(이것을 `인텐트 레졸루션`이라고 한다). 하나 이상 찾을 경우는 사용자가 선택할 수 있게 해준다.

## 명시적 인텐트
명시적 인텐트는 수신하는 액티비티의 __컴포넌트 이름__(실제로는 클래스 이름)을 참조하여 특정 액티비티의 시작을 요청할 때 사용된다.

명시적 인텐트는 Intent 클래스의 새로운 인스턴스를 생성하여 요청한다. 그 이후 인텐트 객체를 `startActivity()`함수를 호출하여 사용한다.

```kotlin
val intent = Intent(this, ActivityB::class.java)
startActivity(intent)
```

startActivity() 함수를 호출하기 이전에 인텐트에 데이터를 추가 할 수 있다. 이때 인텐트 객체의 `putExtra()` 함수를 호출한다. 데이터는 Bundle 형태로 `키-값` 형태의 쌍으로 된 것이어야 한다.

```kotlin
val intent = Intent(this, ActivityB::class.java)
intent.putExtra("myString", "This is a message for ActivityB")
intent.putExtra("myInt", 100)
startActivity(intent)
```

수신 액티비티에서는 Bundle 객체로 데이터를 받는다. 이 객체는 `getIntent().getExtras()`를 호출하여 얻을 수 있다. Activity 클래스의 `getIntent()` 함수는 수신 액티비티를 시작했던 객체를 반환한다. 그리고 Intent 클래스의 `getExtras()` 함수는 데이터를 포함하는 해당 인텐트의 Bundle 객체를 반환한다.

```kotlin
val extras = intent.extras ?: return

val myString = extras.getString("myString")
val myInt = extras.getInt("myInt")
```

인텐트를 사용해서 같은 앱에 있는 다른 액티비티를 시작을 할때는 해당 액티비티가 앱의 manifest 파일에 정의 되어있어야한다. 

`AndroidManifest.xml`
```xml
.
.
<activity
    android:name="ActivityB"
    android:label="ActivityB" >
</activity>
```
## 암시적 인텐트
암시적 인텐트는 수행될 액션과 수신 액티비티에 의해 처리되는 데이터 타입을 지정하여 시작될 액티비티를 식별한다. 

예를 들어, URI 객체 형태로 웹 페이지의 URL을 동반하는 __ACTION_VIEW__ 액션 타입은 웹 브라우저의 능력을 갖는 액티비티를 찾아서 시작시키라고 안드로이드 시스템에 요청한다.

```kotlin
val intent = Intent(Intent.ACTION_VIEW,
        URi.parse("https://www.ebookfrenzy.com"))
startActivity(intent)
```
이 코드ㄴ의 `암시적 인텐트`가 액티비티에 요청되면 장치의 안드로이드 시스템에서 http 데이터의 __ACTION_VIEW__ 요청을 처리할 능력이 있다고 등록된 액티비티를 검색한다. 

- 일치하는 액티비티가 하나만 발견 -> 해당 액티비티 실행
- 두 개 이상 발견 -> 사용자가 액티비티를 선택하게 해준다

    > 이것을 __인텐트  레졸루션__ 이라고 한다.

- - -

## 인텐트 필터 사용하기
액티비티가 자신이 지원하는 액션과 데이터 처리 능력을 안드로이드 인텐트 레졸루션  프로세스에 알리는 메커니즘이 __인텐트 필터__ 이다.

위의 예시에서 요청된 인텐트에 의해 액티비티가 실행되려면 __ACTION_VIEW__ 타입의 지원을 나타내는 인텐트 필터를 `AndroidManifest.xml`에 포함해야한다. 이때 http 형식의 데이터, 즉 웹 페이지를 보여 줄 수 있다는 것도 같이 지정한다.

또한 MainActivity가 암시적 인텐트로 시작될 수 있도록 인텐트 필터도 추가되어 있어야 한다.

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.kyonggi.explicitintent">

    <uses-permission android:name="android.permission.INTERNET" />

    <application
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:roundIcon="@mipmap/ic_launcher_round"
        android:supportsRtl="true"
        android:theme="@style/Theme.ExplicitIntent">
        <activity
            android:name=".ActivityB"
            android:exported="false" />
        <activity
            android:name=".MainActivity"
            android:exported="true">
            <intent-filter>
                <action android:name="android.intent.action.VIEW" />

                <category android:name="android.intent.category.DEFAULT" />
                <data android:scheme="http" />
            </intent-filter>
        </activity>
    </application>

</manifest>
```

# References
- [안드로이드 인텐트 및 인텐트 필터]("https://developer.android.com/guide/components/intents-filters?hl=ko#kotlin")


