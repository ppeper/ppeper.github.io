---
title: "안드로이드 알림(Notification) 만들기"
date: 2022-01-27
update: 2022-01-27
tags:
  - Android
  - Notification
series: "Android"
---
<img src="https://user-images.githubusercontent.com/63226023/151293802-06329eaf-2910-4f76-ba0b-03f171184691.png" width="90%">

안드로이드의 첫 개발을 진행하면서 채팅기능을 만들어 보면서 푸시알림의 기능을 넣지 못했던것이 많이 생각났다.(공모전에 대한 회고도 조만간 작성해야겠다😢) 따라서 이번 포스팅은 알림에 대해서 정리해 보려고 한다.
- - -
# 알림(Notification)
알림에는 __로컬__ 알림과 __원격__ 알림(푸시 알림)이 있다. `로컬 알림`은 장치에서 실행 중인 앱에서 생성되며 `원격 알림`은 원격 서버에서 생성되어 장치에 전송된다.   
이번 알림은 안드로이드 앱에서 로컬 알림을 생성하는 방법과 기본적인 구조를 통하여 알림에 대해서 알아보려고 한다.   
알림에 대한 예시코드는 기본적으로 버튼에 `sendNotification()` 함수를 onclick으로 설정하여 보여 줄 것이다. [(알림 예시코드)](https://github.com/ppeper/Android_Arctic-Fox/tree/main/NotifyDemo)

<img src="https://user-images.githubusercontent.com/63226023/151316170-56467781-ab13-452d-8615-2f5230e7662c.png" width="20%">

알림을 생성하기 위해선 크게 다음과 같이 생성한다.

> 1. 알림 채널 생성 & 등록(__NotificationChannel__)
> 2. 알림 콘텐츠 설정(__NotificationCompat.Builder__)
> 3. 알림 표시(__NotificationManagerCompat.notify()__)
> 4. 알림 탭 작업 설정(__PendingIntent__)

## 1. 알림 채널 생성 & 등록
알림의 콘텐츠를 설정하기 위해 __NotificationCompat.Builder__ 객체의 경우 `채널 ID`를 제공해야 한다. 따라서 알림의 콘텐츠를 생성하기 이전에 `알림 채널`을 만들어야 한다.
> ❗Android 8.0(API 수준 26) 이상에서는 호환성을 유지하기 위해 필요하지만 이전 버전에서는 무시된다고 한다.


알림 채널 설정을 통하여 `소리`, `진동`, `색상`등을 설정 할 수 있다. 기본적으로 알림 채널을 만들기 위해서는 다음과 같다.
1. `고유한 채널 ID`, `사용자가 볼 수 있는 이름`, `중요도` 를 사용하여 __NotificationChannel__ 객체를 구성한다.
> NotificationChannel(String id, CharSequence name, int importance)
2. 선택적으로 `setDescription()` 를 사용하여 시스템 설정에서 사용자에게 보여주는 설명을 지정할 수 있다.
3. 생성된 NotificationChannel 객체를 __createNotificationChannel()__ 함수에 전달하여 등록한다.

```kotlin
private fun createNotificationChannel(
        id: String, name: String,
        description: String
    ) {
        val importance = NotificationManager.IMPORTANCE_LOW
        val channel = NotificationChannel(id, name, importance)

        channel.description = description
        channel.enableLights(true)
        channel.lightColor = Color.RED
        channel.enableVibration(true)
        channel.vibrationPattern =
            longArrayOf(100,200,300,400,500,400,300,200,400)

        notificationManager?.createNotificationChannel(channel)
    }

```
NotificationChannel 매개변수로 들어가는 중요도는 NotificationManager를 통하여 사용하여 가져올 수 있다. 

```kotlin
private var notificationManager: NotificationManager? = null
.
.
.
 notificationManager =
            getSystemService(
                Context.NOTIFICATION_SERVICE) as NotificationManager

// 체널 등록
createNotificationChannel(
            "com.kyonggi.notifydemo.news",
            "NotifyDemo News",
            "Example News Channel"
        )
```

중요도 설정은 채널에 있는 모든 알림의 중단 수준에 영향을 미치게 되고 `NotificationChannel` 생성자에서 지정해야 한다.

<img src="https://user-images.githubusercontent.com/63226023/151309445-b36b29ea-6043-428a-8352-84c64b586c97.png">

앱을 실행하여 설정을 가보면 "NotifyDemo News"로 설정한 이름의 채널을 볼 수 있다.

<img src="https://user-images.githubusercontent.com/63226023/151308576-9677ec3a-1eac-4dc5-b6ff-8049496e7414.png" width="30%">

앱에서 알림 채널을 삭제를 하려면 `NotificationManager`의 deleteNotificationChannel() 함수를 호출하면 된다. 매개변수는 삭제할 채널의 id를 받는다.
```kotlin
val channelID = "com.kyonggi.notifydemo.news"
notificationManager?.deleteNotificationChannel(channelID)
```

중요도 설정은 채널에 있는 모든 알림의 중단 수준에 영향을 미치게 되고 `NotificationChannel` 생성자에서 지정해야 한다.

<img src="https://user-images.githubusercontent.com/63226023/151309445-b36b29ea-6043-428a-8352-84c64b586c97.png">

## 2. 알림 콘텐츠 설정

> 안드로이드 developer 페이지에서 기본적인 알림에 대한 템플릿을 보여준다😎 

<img src="https://user-images.githubusercontent.com/63226023/151313708-87094c32-59e6-4b10-aebb-775de27ee731.png">

알림의 콘텐츠를 설정하기 위해서는 __NotificationCompat.Builder__ 객체를 사용하여 알림을 생성해야한다.

- `setSmallIcon()` : 알림 설정시 유일한 필수 콘텐츠.
- `setContentTitle()` : 알림의 제목
- `setContentText()` : 알림의 본문 내용.

    - `setPriority()` : 알림의 우선순위. 우선순위에 따라 Android 7.1 이하에서 알림이 얼마나 강제적인지 결정된다.

    > ❗Android 8.0 이상의 경우 알림 채널의 생성(NotificationChannel)에서 중요도를 대신 설정해야 한다.
- `setNumber()` : 앱 아이콘을 길게 누르면 나오는 팝업에서 두 개 이상의 알림이 대기 중일 때 알림 개수를 보여준다.

```kotlin
 val channelID = "com.kyonggi.notifydemo.news"

 val notification = NotificationCompat.Builder(
            this,
            channelID
        ).setContentTitle("Example Notification")
            .setContentText("This is an example notification.")
            .setSmallIcon(android.R.drawable.ic_dialog_info)
            .setChannelId(channelID)
            .setNumber(10)
            .build()
```
위에서 설정한 channelID를 setChannelId()로 등록을 하면 기본적인 알림의 생성이 완료된다.

## 3. 알림 표시
알림을 표시하기 위해서는 `NotificationManagerCompat.notify()`를 호출해야 한다.   
notify() 함수는 매개변수로 `알림 ID`와 `NotificationCompat.Builder`에 대한 `.build()`를 전달해야 한다.   
(예시에선 변수 notification)

```kotlin
.
.
val notificationID = 101

 val notification = NotificationCompat.Builder(
            this,
            channelID
        ).
         .
            .build()

notificationManager?.notify(notificationID, notification)
```
앱을 실행하여 버튼을 클릭하면 기본 알림이 오는것을 볼 수 있다.

<img src="https://user-images.githubusercontent.com/63226023/151315879-8a3a7949-df18-4d86-b8e2-824d128ae4eb.png" width="30%">

## 4. 알림 탭 작업 설정
위에서 알림을 만들었지만 클릭을 하더라도 어떤 형태의 액션도 수행할 수 없다. 대부분 알림을 처리하는 앱은 해당 앱의 액티비티를 시작시킨다. 이러한 작업을 추가하기 위해서는 해당 액티비티를 시작시키는 인텐트를 구성해야한다.

실행해야하는 액티비티가 ResultActivity로 지정하여 Intent를 생성한다.
```kotlin
val resultIntent = Intent(this, ResultActivity::class.java)
```

이후 이 인텐트를 `PendingIntent` 인스턴스에 포함해야 한다.[(PendingIntent 개념)](https://ppeper.github.io/android/pendingIntent/) PendingIntent 객체는 다른 앱에 인텐트를 전달할 수 있게 하며, 전달받은 앱은 향후에 인텐트를 수행할 수 있게 한다.
```kotlin
val pendingIntent = PendingIntent.getActivity(
    this, 0, resultIntent, PendingIntent.FLAG_UPDATE_CURRENT
)
```
위에서 설정한 notification 변수에 `setContentIntent()`를 호출하여 PendingIntent 객체를 지정하면 된다.
```kotlin
val notification = NotificationCompat.Builder(
            this,
            channelID
        ).
         .  
            .setContentIntent(pendingIntent)
            .build()
```

> 이제 상단바에서 알림을 클릭하면 ResultActivity로 가는것을 볼 수 있다👌

### 알림에 액션 추가하기
알림에서 액션을 추가하면 또 다른 방법으로 액티비티를 시작시킬 수 있다. 액션은 생성된 알림 메시지의 밑에 `버튼`으로 나타나며, 사용자가 탭을 하여 특정 인텐트를 시작할 수 있다.

알림 액션은 __NotificationCompat.Action.Builder__ 를 통해 action을 생성 할 수 있다.
```kotlin
 val action: NotificationCompat.Action =
            NotificationCompat.Action.Builder(android.R.drawable.ic_dialog_info, "Open", pendingIntent).build()
``` 
생성된 action을 notification에서 `addAction()`을 통하여 지정할 수 있다.
```kotlin
val notification = NotificationCompat.Builder(
            this,
            channelID
        ).
         .  
            .setContentIntent(pendingIntent)
            .addAction(action)
            .build()
```

또 다른 방법은 __addAction(int icon, CharSequence title, PendingIntent intent)__ 로 설정하는 방법이다.

```kotlin
val notification = NotificationCompat.Builder(
            this,
            channelID
        ).
         .  
            .setContentIntent(pendingIntent)
            .addAction(android.R.drawable.ic_dialog_info, "Open", pendingIntent)
            .build()
```

앱을 실행하여 알림창을 보면 설정한 `Open` 액션이 나타나게 된다.

<img src="https://user-images.githubusercontent.com/63226023/151322843-071287c7-50e7-49c8-9419-bcdd3271a374.png" width="30%">

# 알림 메시지 묶기
Android 7.0(API 수준 24)부터는 관련된 알림을 그룹으로 표시할 수 있다. 알림을 묶으려면 각 알림을 __같은 그룹__ 에 속하도록 지정해 한다. 알급 그룹을 위해 __고유 식별자__ 를 생성하여 `setGroup()`으로 호출하면 된다.
```kotlin
val GROUP_KEY_NOTIFY = "group_key_notify"

var builder = NotificationCompat.Builder(this, channelID)
    .
    .
    .
    .setGroup(GROUP_KEY_NOTIFY)
```

또한 그룹 요약을 설정하기 위해서는 `setGroupSummary(true)`를 호출해야 한다. (예시코드의 주석처리 해제후 sendNotification() 함수 내부의 제일 끝으로 볼 수 있다)

- - -

# 알림 직접 응답
Android 7.0(API 수준 24)에서 도입된 직접 응답 기능을 사용하면 알림에서 사용자가 텍스트를 입력하여 알림 관련 앱에 전달할 수 있다.   
예시는 기본적인 알림에서와 동일하지만 `직접 응답`은 사용자의 입력이 textView를 통하여 보여주는점이 다르다. [(직접 응답 알림 예시코드)](https://github.com/ppeper/Android_Arctic-Fox/tree/main/DirectReply)

<img src="https://user-images.githubusercontent.com/63226023/151329014-e1d0b614-194a-4f66-ad8b-d70c81eadf96.png" width="20%">

## RemoteInput 객체 생성
<img src="https://user-images.githubusercontent.com/63226023/151330082-32633069-2932-4ccf-85d5-2e7aaf07f1de.png">

알림에서 직접 응답 텍스트 입력을 가능하게 해주기 위해서는 __RemoteInput__ 객체를 생성해야 한다. __RemoteInput__ 객체 또한 기본 알림과 같이 `인텐트`와 `사용자가 입력한 텍스트`를 PendingIntent가 포함될 수 있게 해준다.

__RemoteInput.Builder()__ 함수를 사용하여 RemoteInput 객체를 생성하게 되고 여기서 필요한 것이 두가지이다.

> 1. 인텐트의 `사용자의 입력`을 추출하기 위한 __키 문자열__
> 2. 알림의 `텍스트 입력 필드`에 나타낼 __라벨 문자열__

```kotlin
 private val KEY_TEXT_REPLY = "key_text_reply"

 fun sendNotification(view: View) {
        val replyLabel = "Enter your reply here"
        val remoteInput = RemoteInput.Builder(KEY_TEXT_REPLY)
            .setLabel(replyLabel)
            .build()

        .
        .
        .
    }
```
이제 기본 알림과 마찬가지로 생성된 RemoteInput 객체를 `알림 액션`객체에 포함시켜야 한다.   
그 전에 PendingIntent 객체를 생성하여 사용자가 응답한 입력을 추출할 수 있게 생성해 준다.

```kotlin
val resultIntent = Intent(this, MainActivity::class.java)

        val resultPendingIntent = PendingIntent.getActivity(
            this, 0, resultIntent, PendingIntent.FLAG_UPDATE_CURRENT
        )
```

이후 알림 액션에 `addRemoteInput()`을 호출하여 생성한 __RemoteInput__ 객체를 연결해 준다.

```kotlin
val icon = Icon.createWithResource(
    this,
    android.R.drawable.ic_dialog_info
)

val replyAction = Notification.Action.Builder(
    icon, "Reply", resultPendingIntent)
    .addRemoteInput(remoteInput)
    .build()
```

끝으로 생성된 `알림`을 적용하고 표시한다.
```kotlin
val newMessageNotification = Notification.Builder(this, channelID)
        .setColor(ContextCompat.getColor(this,
        R.color.design_default_color_primary))
        .setSmallIcon(android.R.drawable.ic_dialog_info)
        .setContentTitle("My Notification")
        .setContentText("This is a test Message")
        .addAction(replyAction)
        .build()

val notificationManager = getSystemService(
    Context.NOTIFICATION_SERVICE) as NotificationManage
notificationManager.notify(
    notificationId,
    newMessageNotification
)
```
앱을 실행해 보면 생성된 `직접 응답 알림`을 확인해 볼 수 있다.

<img src="https://user-images.githubusercontent.com/63226023/151334603-3688781f-8352-4035-9816-a85b98c31f98.png" width="30%"> <img src="https://user-images.githubusercontent.com/63226023/151334622-e927ef45-28b6-4c2c-a283-77dd70ec5b91.png" width="32%">

## 직접 응답 입력 데이터 수신하기
사용자가 알림의 직접 입력을 하였으므로 앱에서 입력 데이터를 받아와야 할 것이다. 앞서 말했던 textView를 통하여 입력데이터의 수신을 확인해 본다.

사용자의 입력 데이터를 가져오기 위해서는 위에서 생성한 __RemoteInput.getResultsFromIntent()__ 를 통하여 Bundle 객체를 반환하여 가져온다.

```kotlin
 private fun handleIntent() {
        val intent = this.intent
        val remoteInput = RemoteInput.getResultsFromIntent(intent)
        if (remoteInput != null) {
            val inputString = remoteInput.getCharSequence(
                KEY_TEXT_REPLY).toString()
            binding.textView.text = inputString
        }
 }
```
여기서 전에 RemoteInput 객체 생성시 사용한 Key값`(KEY_TEXT_REPLY)`으로 Bundle 값을 가져온다.

알림에서 응답 텍스트를 입력하고 전송하게되면 전송된 응답 텍스트의 액티비티 수신 확인을 기다리는 것을 나타낸다. 알림 전송시 사용한 동일한 ID로 알림을 업데이트 하여 사용자에게 `답장이 올바르게 수신되고 처리`되었음을 확인하기 위해 필요하다.
```kotlin
private fun handleIntent() {
        .
        .
        val repliedNotification = Notification.Builder(this, channelID)
            .setSmallIcon(android.R.drawable.ic_dialog_info)
            .setContentText("Reply received")
            .build()

        notificationManager?.notify(notificationId, repliedNotification)
    }
}
```
앱을 실행하여 메시지를 보내게 되면 생성한 `repliedNotification`으로 알림이 바뀌는것을 볼 수 있다.   
-> (텍스트 "Hello"로 직접응답 메시지 전송)

<img src = "https://user-images.githubusercontent.com/63226023/151365468-0f00e889-943f-4b47-8072-c1eb231d4480.png" width="30%"> <img src = "https://user-images.githubusercontent.com/63226023/151365476-72cf2f63-e0a2-476d-900e-121ecfad7251.png">

- - -

# References
- [Android developer 알림 개요](https://developer.android.com/training/notify-user/build-notification?hl=ko)
