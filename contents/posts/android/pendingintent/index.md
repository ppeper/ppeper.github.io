---
title: "안드로이드 PendingIntent의 개념"
date: 2022-01-25
update: 2022-01-25
tags:
  - Android
  - PendingIntent
series: "Android"
---
- - -
안드로이드의 개념을 다시 한번 보면서 Notification(알림)을 처음으로 공부해 보았다. 알림 구현에서 `PendingIntent`를 사용하라고 하여 사용했지만 명확한 개념을 알려주지 않고 넘어가 `PendingIntent`와 `Intent`가 무엇이 다른지 궁금하여 정리해 보았다.

# PendingIntent
 > 안드로이드 developer에서 PendingIntent는 다른 응용 프로그램(프로세스)이 동일한 권한 및 ID를 사용하는 것처럼 지정한 작업을 수행할 수 있는 권한을 해당 응용 프로그램에 부여하게 된다고 설명한다.~~(이게 무슨의미지..?😅)~~

PendingIntent 객체는 이름에서 볼 수 있듯이 __Intent__ 를 가지고 있는 클래스이다. 앞의 `Pending`이 붙은것을 보면 PendingIntent의 특징을 어느정도 유추해 볼 수 있다.

<img src="https://user-images.githubusercontent.com/63226023/150944701-1b6b8e8c-bab9-428e-8f7a-9524222e2e34.png" >


__Pending__ 은 __보류__ 라는 의미를 가진 단어이다. 이를 합쳐보면 PendingIntent는 `보류중인 인텐트` 라고 할 수 있다.

보류중이라는 것은 나중에 어느 시점에 실행이 된다는 뜻이므로 PendingIntent는 __바로 실행을 하지 않고 어느 특정 시점에 실행을 하는 Intent__ 라는 의미로 파악할 수 있다. 어느 특정시점이라고 하면 이는 보통 __앱이 구동되지 않고__ 있을 시점이다. 

사용 예시로 Notification(알림)이 있다. 어플리케이션 안에서 __대용량 데이터를 다운__ 받을 시 보통 사용자는 다운로드 중에는 다른 앱을 사용하다가 __다운로드가 완료__ 되었을때 푸시알림을 통하여 앱으로 다시 돌아가는 경우를 본적이 있을것이다. 

푸시알림을 하기위한 `Notification`은 안드로이드 시스템의 __NotificationManager__ 을 통하여 `Intent`가 실행하게 된다. 이는 다른 프로세스에서 Intent를 실행하는 것으로 PendingIntent를 사용이 필수적이다.

> 🔔 PendingIntent를 통하여 `앱이 구동되지 않을때` 다른 프로세스(예: Notification)에게 권한을 허가하여 `Intent`(지정한 작업)를 수행 할 수 있게 한다.

- - -
## PendingIntent 생성하기
PendingIntent 객체는 각 컴포넌트마다 생성자의 호출방식이 다르다.

### - Activity
 ```PendingIntent.getActivity(Context, Int, Intent, Int)```

### - BroadcastReceiver
 ```PendingIntent.getBroadcast(Context, Int, Intent, Int)```

### - Service
 ```PendingIntent.getService(Context, Int, Intent, Int)```


각 컴포넌트마다 파라미터는 __Context, Int, Intent, Int__ 는 공통으로 구성되어 있다.

## 파라미터 정보
1. __Context__ : 해당 `context` 정보이다.
2. __Int__ : `requestCode`로 `PendingIntent`를 가져올때 구분을 하기위하여 사용한다.
3. __Intent__ : 실행할 `Intent`이다.
4. __Int__ : 플래그의 값 이다.
    - `FLAG_CANCEL_CURRENT` : 전에 생성한 `PendingIntent`를 취소 후 새로 사용
    - `FLAG_UPDATE_CURRENT` : 전에 생성한 `PendingIntent`가 있다면, `Extra Data` 를 모두 교체
    - `FLAG_NO_CREATE` : 전에 생성한 `PendingIntent`가 있다면 재사용 (없으면 `Null` 리턴)
    - `FLAG_ONE_SHOT` : `PendingIntent`를 일회성으로 사용

# References
- [PendingIntent - Android Developers](https://developer.android.com/reference/android/app/PendingIntent)