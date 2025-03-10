---
title: "안드로이드 Local Service 알아보기"
date: 2023-04-10
update: 2023-04-10
tags:
  - Android
  - Service
series: "Android"
---
안드로이드의 4대 컴포넌트중에 하나인 [Service의 구성](https://ppeper.github.io/android/service/)은 과거에 한번 나누어 본적이 있다. 

안드로이드에서 Activity/Fragment는 많이 사용하지만 실제로 Service를 사용을 하지 않아 이번에 서비스에 대한 예시로 __Local  Service__ 의 동작 과정을 보려고 한다.

# Bind Service
안드로이드에서 바운드 서비스는 `startService()`와는 달리 하나 이상의 클라이언트 컴포넌트 간의 통신이 구현 가능하다. 
Service가 Background에서 실행되고 있을 때, Activity 및 Fragment에서 Service의 메서드를 호출/결과를 받아서 보여주는 경우에 사용할 수 있다.

## bindService 생명주기
<p align="center"><img src="https://user-images.githubusercontent.com/63226023/150674329-805df55b-0cde-4a26-915a-1cc42fb9343a.png" width="40%"></p>

- 서비스가 __bindService()__ 함수로 시작되면 `onCreate()` -> `onBind()` 함수가 호출되면서 Running 상태가 된다.
- `bindService()` 함수로 실행된 서비스는 `unbindService()` 함수로 종료하고, 종료하면서 `onUnbind()` -> `onDestroy()` 함수가 호출된다.

## bindService 호출 방법

- startService()를 통해 서비스를 실행시키고 클라이언트가 bindService()를 통해 이에 binding 할 수 도 있다.
    - 1 이때는 바인딩을 모두 해제해도 서비스가 소멸되지 않으므로 `stopSelf()` 또는 `stopService()` 를 호출 해 주어야함
    - 2 이런 서비스(서비스를 계속 실행하면서 바인딩도 제공해야하는 경우) `onStartCommand()` 와 `onBind()` 둘 다를 구현 해주어야 한다.

## IPC / Binder
<p align="center"><img src="https://user-images.githubusercontent.com/63226023/230884520-c4d718d9-696c-4d99-a7da-60a3ae834b87.png" width="50%"></p>

- IPC (Inter Process Communication)
    - 프로세스 통신, 리눅스 커널에서 Binder를 이용하여 프로세스간 메시지를 주고 받도록 구현되어 있다.
    - 안드로이드는 리눅스 커널을 기반으로 만들어져 있으므로, process는 kernel 내부의 일정 공간을 공유하여 함수를 호출하며 이를 Binder Driver가 수행한다.

- Binder는 데이터를 parcel 형태로 전달하기 때문에 안드로이드에서는 `parcelable` 클래스를 활용한다.

---

# Local Service
- Local에서 사용할 `BoundService.kt`를 작성한다.

```kotlin
class BoundService: Service() {
    private lateinit var myBinder: MyLocalBinder

    // Foreground Service가 아니면 Bind Serice에서 사용할 Binder를 리턴
    override fun onBind(intent: Intent?): IBinder {
        myBinder = MyLocalBinder()
        return myBinder
    }

    // 부를 함수를 생성
    fun getCurrentTime(): String {
        val dateFormat = SimpleDateFormat("HH:mm:ss MM/dd/yyyy", Locale.KOREA)
        dateFormat.timeZone = TimeZone.getTimeZone("Asia/Seoul")
        return dateFormat.format(Date())
    }

    // 외부면 Binder로 할 수 없음
    /**
     * Binder 는 IBinder 의 구현체로 onBind 를 통해 서비스 클라이언트에게 전달
     * 되며 클라이언트는 이 객체를 이용해 서비스에 선언된 기능을 호출
     */
    inner class MyLocalBinder: Binder() {
        // 외부 객체인 BoundService 객체를 반환하는 함수
        fun getService() = this@BoundService
    }
}
```

- service를 사용하기 위해 AndroidManifest.xml 파일에 service를 작성해 준다.

```xml
<service
    android:name="com.test.service.BoundService"
    android:enabled="true"
    android:exported="true" />
```

- 서비스를 바인딩하고자 하는 Activity 파일을 생성한다.

```kotlin
class BindActivity : AppCompatActivity() {

    private lateinit var myService: BoundService
    private var isBound = false

    /**
     * 서비스가 반환한 바인더 객체(service: IBinder)를 이용하여
     * 서비스에 접속하거나 접속을 종료하는 ServiceConnection 객체를 활용한다.
     */
    private val conn = object : ServiceConnection {
        override fun onServiceConnected(name: ComponentName?, service: IBinder?) {
            val binder = service as BoundService.MyLocalBinder
            myService = binder.getService()
            isBound = true
        }

        override fun onServiceDisconnected(name: ComponentName?) {
            isBound = false
        }
    }
}
```
서비스에서 반환할 `IBinder` 객체를 사용하기 위해 __ServiceConnection__ 의 서브 클래스를 구현하여 서비스의 연결 및 종료하는 코드를 작성하여 사용할 수 있다.

```kotlin
override fun onStart() {
    super.onStart()
    Intent(this, BoundService::class.java).also { intent ->
        bindService(intent, conn, Context.BIND_AUTO_CREATE)
    }
}

override fun onDestroy() {
    super.onDestroy()
    if (isBound) {
        unbindService(conn)
        isBound = false
    }
}

private lateinit var binding : ActivityBindBinding
override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    binding = ActivityBindBinding.inflate(layoutInflater)
    setContentView(binding.root)

    binding.button.setOnClickListener {
        // 서비스에 있는 함수인 getCurrentTime 가져오기
        if (isBound) {
            val result = myService.getCurrentTime()
            binding.textView.text = result
        }
    }
}
```
- onStart()에서 isBound가 아직 안되었자면 bindService()를 호출하여 생성한 BoundService를 연결 시켜 준다.
- onDestroy()에서 Activity가 종료되었다면 unbindService로 바인딩된 서비스 연결을 해제한다.

해당 xml의 버튼을 누르게 되면 서비스에서 getCurrentTime()을 호출하여 현재 시간을 보여주게 된다.

<img src="https://user-images.githubusercontent.com/63226023/230888917-1be3b7b0-1a65-46dc-9830-6e56ee68442c.png">