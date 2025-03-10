---
title: "안드로이드 브로드캐스트(Broadcast)"
date: 2022-01-20
update: 2022-01-20
tags:
  - Android
  - Broadcast
series: "Android"
---
- - -
# 브로드캐스트(Broadcast)
앱의 액티비티를 시작시키는 메커니즘을 제공하는 것과 더불어 인텐트는 시스템의 다른 컴포넌트에 시스템 차원의 메시지를 전파하는 방법으로 사용될 수 있다. 예를 들어 안드로이드 시스템은 부팅이나 배터리가 부족할때와 같은 시스템 이벤트가 발생하면 __브로드캐스트__ 를 전송한다. 

브로드캐스트(방송)를 보낸다고 하면 어떠한 정의된 방법으로 `포장`을 하고 보내는 사람이 있으면 받는 `수신자`가 있을것이다. 안드로이드에서는 이를 __브로드캐스트 인텐트__, __브로드캐스트 수신자__ 를 사용한다.

## 브로드캐스트 인텐트 개요
브로드캐스트 인텐트는 Activity 클래스의 `sendBroadcast()` 또는 `sendStickyBroadcast()` 또는 `sendOrderedBroadcast()` 함수를 호출하여 전파되는 Intent 객체다. [(안드로이드 인텐트 개념)](https://ppeper.github.io/android/intent/)

브로드캐스트 인텐가 생성될 때는 __액션 문자열__ 을 포함해야 한다. 그리고 선택적인 데이터와 카테고리 문자열을 추가로 포함할 수 있다. 즉, 일반적인 인텐트와 마찬가지로 __putExtra(String, Bundle)__ 를 사용하여 데이터를 브로드캐스트 인텐트에 추가할 수 있다. 

```kotlin
val intent = Intent()
intent.action = "com.example.Broadcast"
intent.putExtras("MyData", 1000)
sendBroadcast(intent)
```

위의 예시는 안드로이드 3.0 이전 버전을 실행 중인 장치에서는 성공적으로 브로드캐스트 수신자를 시작시킬 수 있을 것이다. 그러나 3.0 이상 버전에서는 론칭(실행)을 제어하는 보안 조치가 도입되어 브로드캐스트 수신자에 의해 이 인텐트가 수신되지 않는다고 한다. -> __사용 정지된__ 앱의 컴포넌트가 인텐트를 통하여 시작되는 것을 막는 조치다.

앱이 방금 설치되어서 이전에 론칭(시작)된 적이 없거나 장치의 앱 매니저를 사용해서 사용자가 수동으로 정지시켰을 경우, 해당 앱은 사용 정지된 상태에 있다고 간주된다. 

```kotlin
val intent = Intent()
intent.action = "com.example.Broadcast"
intent.putExtras("MyData", 1000)
-> intent.flags = Intent.FLAG_INCLUDE_STOPPED_PACKAGES
sendBroadcast(intent)
```

이것을 해결하려면 인텐트에 __FLAG_INCLUDE_STOPPED_PACKAGES__ 플래그를 추가하여 사용 정지된 앱의 컴포넌트를 시작시키는 것이 인텐트에 허용된다는 것을 나타낼 수 있다.~~(Flags를 찾아보며 알아가야겠다..))~~


## 브로드캐스트 수신자 개요
브로드캐스트 인텐트가 생성이 되었다면 받을 수신자를 등록해야 한다. 브로드캐스트 수신자는 __BroadcastReceiver__ 클래스로 부터 상속받고 __onReceive()__ 함수를 오버라이딩하여 구현 한다.

브로드캐스트 인텐트와 받을 브로드캐스트 리시버가 있으면 수신자가 누군지 정해주어야 할 것이다. 안드로이드에서는 이를 __manifest 파일__ 및 __컨텍스트__(예: 액티비티 내부)에의 두가지 방식으로 수신자를 등록하여 브로드캐스트 인텐트를 수신할 수 있다. 

인텐트 필터에 특정 브로드캐스트 인텐트를 나타내는 __액션 문자열__ 을 정의하면 안드로이드에서 이 액션 문자열과 일치하는 브로드캐스트 인텐트가 감지되면 해당 브로드캐스트 수신자의 __onReceive()__ 함수가 호출된다. 

### Manifest.xml에 수신자 등록
브로드캐스트 수신자를 매니페스트 파일에 등록할 때는 __`<receiver>`__ 항목을 추가해야 한다.

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.kyonggi.sendbroadcast">

    <application
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:roundIcon="@mipmap/ic_launcher_round"
        android:supportsRtl="true"
        android:theme="@style/Theme.SendBroadcast">
        <receiver
            android:name=".MyReceiver"
            android:enabled="true"
            android:exported="true">
        </receiver>
                .
                .
                .
    </application>
</manifest>
```

안드로이드 8.0 이전 버전이 실행되는 경우에는 수신자와 관련된 인텐트 필터를 manifest 파일의 receiver에 둘 수 있다. 예시에서는 "com.exmaple.Broadcast"를 포함하는 브로드캐스트 인텐트를 리스닝한다.
```xml
<receiver 
    android:name=".MyReceiver">
        <intent-filter>
            <action android:name="com.example.Broadcast">
            </action>
        </intent-filter>
</receiver>
```

### 컨텍스트에 수신자 등록

반면에 안드로이드 8.0 이상 버전에서는 코드에서 적합한 __IntentFilter__ 객체를 생성한 후 Activity 클래스의 __registerReceiver()__ 함수를 호출하여 등록해야 한다.

```kotlin
val filter = IntentFilter()
filter.addAction("com.example.Broadcast")
val receiver = MyReceiver()
registerReceiver(receiver, filter)
```

코드에서 등록된 브로드캐스트 수신자가 더 이상 필요하지 않을 때는 Activity 클래스의 __`unregisterReceiver(receiver: BroadcastReceiver!)`__ 함수를 호출하여 등록을 해지할 수 있다.

```kotlin
unregisterReceiver(receiver)
```
- - -

# 예제 코드

- activity_main.xml

 버튼과 onclick 속성으로 broadcastIntent()를 함수를 구현한다.

<img src="https://user-images.githubusercontent.com/63226023/150098676-4b56e257-eb45-4a40-be66-b8f2c751fc62.png" height="50%">

- MainActivity class

MainActivity에서 브로드캐스트 인텐트를 지정해주고 액션 문자열은 __com.kyonggi.sendbroadcast__ 로 지정한다.

그 다음은 "브로드캐스트 수신자"의 인텐트 필터에 위의 액션 문자과 일치하는 __`<action>`__ 요소를 정의할 것이다.


```kotlin
class MainActivity : AppCompatActivity() {
    .
    .
    // 브로드캐스트 인텐트를 전송
    fun broadcastIntent(view: View) {
        val intent = Intent()
        intent.action = "com.kyonggi.sendbroadcast"
        intent.flags = Intent.FLAG_INCLUDE_STOPPED_PACKAGES
        sendBroadcast(intent)
    }
}
```
- MyReceiver class

1. 브로드캐스트 리시버는 __BroadcastReceiver()__ 와 __onReceive()__ 를 구현해야 한다고 하였다.
2. 예시에서는 Toast메시지를 통하여 브로드캐스트 인텐트를 수신하였는지 확인할 것이다.

```kotlin
class MyReceiver : BroadcastReceiver() {

    override fun onReceive(context: Context, intent: Intent) {
        val message = "Broadcast intent detected " + intent.action
        Toast.makeText(context, message, Toast.LENGTH_SHORT).show()
    }
}
```

브로드캐스트 인텐트와 브로드캐스트 리시버가 완성되었기 때문에 이제 수신자를 등록해 주어야 한다.

수신자는 위에서 말한 인텐트 필터를 통하여 액션 문자열과 리시버를 포함해야한다. 

MyReceiver를 생성하였을 때 안드로이드 스튜디오에서 자동으로 __`<receiver>`__ 요소를 manifest 파일에 추가해 주었다. 지금 실행하는 안드로이드는 8.0 이상이기때문에 코드로 __IntentFilter__ 를 설정해 주어야한다.

```kotlin
class MainActivity : AppCompatActivity() {

    // 브로드캐스트 인텐트를 지정해야한다.
    var receiver: BroadcastReceiver? = null

    override fun onCreate(savedInstanceState: Bundle?) {
        .
        .
        configureReceiver()
    }

    private fun configureReceiver() {
        val filter = IntentFilter()
        filter.addAction("com.kyonggi.sendbroadcast")
        receiver = MyReceiver()
        registerReceiver(receiver, filter)
    }

    override fun onDestroy() {
        super.onDestroy()
        unregisterReceiver(receiver)
    }
    .
    .
}
```
onDestroy() 함수는 브로드캐스트 수신자가 더 이상 필요 없을 때 등록을 해지하기 위해 필요하다.

안드로이드 장치에서 Send Broadcast 버튼을 터치하면 처음 등록한 Toast 메시지가 나타나는걸 볼 수 있다.

<img src="https://user-images.githubusercontent.com/63226023/150101172-d94e1502-ab10-4e7a-9a7a-58f529acfa1c.png" height="50%">

- 추가: 장치의 외부 전원이 끊어졌을 때 전송되는 시스템 브로드캐스트 인텐트를 리스닝 하도록 브로드캐스트 수신자의 인텐트 필터를 변경해 보자. 이때 사용되는 액션은 __android.intent.action.ACTION_POWER_DISCONNECTED__ 이다.

```kotlin
    private fun configureReceiver() {
        val filter = IntentFilter()
        filter.addAction("android.intent.action.ACTION_POWER_DISCONNECTED")
        receiver = MyReceiver()
        registerReceiver(receiver, filter)
    }
```

이제 다시 앱을 실행하여 전원을 공급을 하지 않게되면 다음과 같은 Toast 메시지가 나올것이다. (에뮬레이터에서 실행 중이라면 확장 제어 대화상자 -> Battery선택 -> Charger connection을 AC charger 선택 했다가 None으로 변경)

``` > Broadcast intent detected android.intent.action.ACTION_POWER_DISCONNECTED```


[시스템 브로드캐스트 인텐트 수신 예시](https://github.com/ppeper/Android_Arctic-Fox/tree/main/SendBroadcast)
- - -

# References
- [안드로이드 브로드캐스트 개요](https://developer.android.com/guide/components/broadcasts?hl=ko)

