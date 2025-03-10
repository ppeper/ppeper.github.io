---
title: "안드로이드 Permission 가져오기"
date: 2023-03-24
update: 2023-03-24
tags:
  - Android
  - Permission
series: "Android"
---
# Android Permission
안드로이드에서는 특정한 앱을 실행하기 위해서는 권한이 필요하다. 안드로이드 마시멜로 버전(API 23) 이전에는 앱 설치시 모든 권한이 요청되었지만 이후에는 __일반 권한__, __위험 권한__ 으로 나눠게 되었다.

따라서 API 23 이후 버전의 기기에서는 위험 권한은 기능이 동작할 때 사용자가 직접 권한을 허락하도록 __런타임에 권한을 설정__ 해주어야 한다.

<p align="center"><img src="https://user-images.githubusercontent.com/63226023/229967185-75cf1404-9f97-4728-bfca-1a10731d0bea.png"></p>

위에 해당하는 기능들은 런타임시 권한을 설정해 주어야 한다. 만약 권한이 없다면 앱을 종료시켜버린다.😅

## ✅권한 체크
__ContextCompat.checkSelfPermission(컨텍스트(), 퍼미션정보)__ 메서드를 사용하여 앱에 이미 권한을 부여 받았는지 확인 할 수 있다.

이 메서드는 앱에 권한 여부에 따라 `PERMISSION_GRANTED` 또는 `PERMISSION_DENIED`를 반환 한다.

## 👉권한 요청
__Activity.requestPermissions(컨텍스트(),퍼미션 리스트,요청값)__ 메서드를 사용하여 퍼미션 정보들과 요청을 보내게 되면 해당 요청 값이 반환된다.

## 📄요청 결과
요청 결과는 __onRequestPermissionsResult()__ 가 호출된다. 이 메서드는 요청 값을 확인하여 권한 상태에 따라 결과를 처리 한다.

AndroidManifest.xml에 위치 권한을 추가한다.

```xml
<!-- GPS 센서로 측정-->
<uses-permission android:name="android.permission.ACCESS_FINE_LOCATION"/>
<!-- 네트워크로 측정-->
<uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION" />
```

```kotlin
// 받으려는 권한들
val RUNTIME_PERMISSIONS = arrayOf(
    Manifest.permission.ACCESS_COARSE_LOCATION,
    Manifest.permission.ACCESS_FINE_LOCATION
)
// 권한 요청 코드
val PERMISSION_REQUEST_CODE = 8;

// 권한 체크
val coarseResult = ContextCompat.checkSelfPermission(this, Manifest.permission.ACCESS_COARSE_LOCATION)
val fineResult = ContextCompat.checkSelfPermission(this, Manifest.permission.ACCESS_FINE_LOCATION)

// 권한 획득했는지 체크
if (coarseResult == PackageManager.PERMISSION_GRANTED &&
        fineResult == PackageManager.PERMISSION_GRANTED ){
    // 권한이 확인된 이후 작업
} else {
    //권한이 없을때 권한 요청창 띄우기
    ActivityCompat.requestPermissions(this, RUNTIME_PERMISSIONS, PERMISSION_REQUEST_CODE);
}

// 요청 결과
override fun onRequestPermissionsResult(
    requestCode: Int,
    permissions: Array<String>,
    grantResults: IntArray
) {
    super.onRequestPermissionsResult(requestCode, permissions, grantResults)
    when (requestCode) {
        PERMISSION_REQUEST_CODE -> {
            var results = true
            Log.d(TAG, "onRequestPermissionsResult: 건수 : $grantResults")
            if (grantResults.size > 0) {
                for (temp in grantResults) {
                    if (temp == PackageManager.PERMISSION_DENIED) { //전체 권한 grant확인
                        results = false
                        break
                    }
                }
            }

            if (results) { // 전체 권한 획득된 경우.
                Toast.makeText(this, "모든 권한이 허가되었습니다.", Toast.LENGTH_SHORT).show()
            } else { // 거절된 경우
                Toast.makeText(this, "권한이 필요합니다.", Toast.LENGTH_SHORT).show()
            }
        }
    }
}
```
<img src="https://user-images.githubusercontent.com/63226023/230065499-72f269c8-c236-4e9e-8b8a-63dcff8f34f8.png">

안드로이드에서 `onRequestPermissionsResult()` 메소드는 deprecated 되어 registerForActivityResult를 사용하도록 권고한다.

권한 설정을 하기 위해선 ActivityResultConstracts의 `RequestPermission()` 으로 단일 권한을 요청할 수 있고, `RequestMultiplePermissions()` 으로 복수 권한을 요청할 수 있다.

단일 권한의 콜백으로는 `Boolean` 값으로 권한 허용 여부를 알 수 있고, 복수 권한의 경우 `Map<String, Boolean>` 으로 각 권한들의 key에 따라 구분 된다.

## ActivityResultConstracts 사용하기
```kotlin
// 단일 권한
val RUNTIME_PERMISSION = Manifest.permission.ACCESS_COARSE_LOCATION,

val requestPermission = registerForActivityResult(
    ActivityResultContracts.RequestPermission()) { isGrant ->
    when (isGrant) {
        true -> {
            Toast.makeText(this,"권한 허가",Toast.LENGTH_SHORT).show()
        }
        false -> {
            Toast.makeText(this,"권한 불가",Toast.LENGTH_SHORT).show()
        }
    }
}

// 복수 권한
val RUNTIME_PERMISSIONS = arrayOf(
    Manifest.permission.ACCESS_COARSE_LOCATION,
    Manifest.permission.ACCESS_FINE_LOCATION
)

val requestMultiplePermission = registerForActivityResult(
    ActivityResultContracts.RequestMultiplePermissions()) { result ->
    // result -> Map<String, Boolean>
    for ((permission, isGrant) in result) {
        Toast.makeText(this, "$permission = $isGrant", Toast.LENGTH_SHORT).show()
    }
}

// 권한 획득했는지 체크
if (coarseResult == PackageManager.PERMISSION_GRANTED &&
        fineResult == PackageManager.PERMISSION_GRANTED ){
    // 권한이 확인된 이후 작업
} else {
    //권한이 없을때 권한 요청창 띄우기
    // ActivityCompat.requestPermissions(this, RUNTIME_PERMISSIONS, PERMISSION_REQUEST_CODE);

    // 단일 권한
    requestPermission.launch(RUNTIME_PERMISSION)
    // 복수 권한
    requestMultiplePermission.launch(RUNTIME_PERMISSIONS)
}
```

Toast 메시지로 복수 권한에 대한 결과를 확인 해 볼 수 있다.

<img src="https://user-images.githubusercontent.com/63226023/230070042-b37cb500-777b-4d86-a378-c2c908aa3d77.png" width="30%"/> <img src="https://user-images.githubusercontent.com/63226023/230070044-3d9de587-2f16-46c7-8724-68044adbdf76.png" width="30%"/> <img src="https://user-images.githubusercontent.com/63226023/230070051-be83cf3d-445d-4877-85c1-e19caed21b08.png" width="30%"/>

- - -
# References 
- [https://developer.android.com/training/permissions/requesting?hl=ko](https://developer.android.com/training/permissions/requesting?hl=ko)