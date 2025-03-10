---
title: "안드로이드 MediatorLiveData와 Transformations"
date: 2023-05-02
update: 2023-05-02
tags:
  - MediatorLiveData
  - Transformations
  - LiveData
series: "Android"
---
안드로이드에서 [LiveData](https://ppeper.github.io/android/android-livedata/)를 데이터를 저장하고 변화를 관찰 할 수 있는 객체이다.이번에는 LiveData를 조금 더 유연하게 사용하는 방법에 대해 몰랐던 내용을 알아보려고 한다.

# MediatorLiveData
지금까지 하나의 LiveData는 Observer 인터페이스를 구현하는 객체를 생성하여 LiveData 값이 변경될 때 호출되는 `onChange()` 메소드를 통해 관찰 할 수 있었다.

```kotlin
liveString.observe(this, object : Observer<String> {
    override fun onChanged(liveString: String?) {
        Log.d(TAG, "value : $liveString")
        text.text = liveString
    }
})
// 람다식으로 변환
liveString.observe(this) { liveString ->
    Log.d(TAG, "value : $liveString")
    text.text = liveString
}
```

`MediatorLiveData`는 여러 LiveData 객체들을 `observe`할 수 있도록 하는 LiveData의 하위 클래스이다. 즉 서로다른 Data Source들을 각각 관찰하는 것이 아닌 한번에 모두 관찰할 수 있다.

```kotlin
val emailLiveData: MutableLiveData<String> = MutableLiveData()
val pwdLiveData: MutableLiveData<String> = MutableLiveData()
val repwdLiveData: MutableLiveData<String> = MutableLiveData()
val nameLiveData: MutableLiveData<String> = MutableLiveData()
val isValidLiveData: MediatorLiveData<Boolean> = MediatorLiveData<Boolean>().apply {
    // 입력이 valid 하지않으면 버튼 disabled
    this.value = false

    addSource(emailLiveData) {
        this.value = validateForm()
    }

    addSource(pwdLiveData) {
        this.value = validateForm()
    }

    addSource(repwdLiveData) {
        this.value = validateForm()
    }

    addSource(nameLiveData) {
        this.value = validateForm()
    }

}
private fun validateForm(): Boolean {
    return !emailLiveData.value.isNullOrBlank() && !pwdLiveData.value.isNullOrBlank()
            && !repwdLiveData.value.isNullOrBlank() && !nameLiveData.value.isNullOrBlank()
}
```

`EditText`의 입력값으로 받는 LiveData들에 대하여 입력되지 않은 값이 없을때만 버튼을 활성화 하기 위해 `addSource` 메소드로 `LiveData`와 콜백을 설정해준다. 

해당하는 LiveData들이 변경될 때 `validateForm()` 메서드가 호출되어 하나의 MediatorLiveData만 observe 하더라도 다른 LiveData의 변경사항이 observe되어 모든 EditText가 입력되었을 때 `isValidLiveData` 의 상태가 true로 변경될 것이다.

# Transformations
이름과 같이 `LiveData` 를 변환하고자 할 때 사용된다.

## map()

```kotlin
@MainThread
public static <X, Y> LiveData<Y> map(
        @NonNull LiveData<X> source,
        @NonNull final Function<X, Y> mapFunction) {
    final MediatorLiveData<Y> result = new MediatorLiveData<>();
    result.addSource(source, new Observer<X>() {
        @Override
        public void onChanged(@Nullable X x) {
            result.setValue(mapFunction.apply(x));
        }
    });
    return result;
}
```
`map()` 함수는 `MediatorLiveData`를 내부적으로 사용하여 `addSource` 메소드를 통해 Observer를 등록해 주고 `onChanged()` 에서 `mapFunction` 를 통해 파라미터로 받은 X를 Y로 변환해 주어 result값을 set해준다.

예시로 LiveData인 count의 숫자 값에 따라 계산 결과를 보여주는 times 문자열 변수는 일반적으로 하면 값이 변화할 때마다 옵저빙 하기 어렵다. 따라서 count변수의 LiveData를 변경하여 새로운 LiveData형으로 만들어 줄 수 있다.

```kotlin
class NewActivityViewModel:ViewModel() {
    // 사용자의 클릭 수를 세는 변수
    private var _count = MutableLiveData<Int>().apply {
        value = 0
    }
    val count: LiveData<Int>
        get() = _count

    // 숫자를 계산 결과를 보여주는 문자열로 보여주기 위한 Transformations.map 사용
    val times = Transformations.map(count) {
        "$it x 2 = ${it * 2}"
    }
}
```

## switchMap()

```kotlin
@MainThread
public static <X, Y> LiveData<Y> switchMap(
        @NonNull LiveData<X> source,
        @NonNull final Function<X, LiveData<Y>> switchMapFunction) {
    final MediatorLiveData<Y> result = new MediatorLiveData<>();
    result.addSource(source, new Observer<X>() {
        LiveData<Y> mSource;

        @Override
        public void onChanged(@Nullable X x) {
            LiveData<Y> newLiveData = switchMapFunction.apply(x);
            if (mSource == newLiveData) {
                return;
            }
            if (mSource != null) {
                result.removeSource(mSource);
            }
            mSource = newLiveData;
            if (mSource != null) {
                result.addSource(mSource, new Observer<Y>() {
                    @Override
                    public void onChanged(@Nullable Y y) {
                        result.setValue(y);
                    }
                });
            }
        }
    });
    return result;
}
```

`switchMap()` 은 map과 비슷하지만 함수의 결과로 받은 `LiveData` 값이 같다면 아무 작업도 수행하지 않고, 다를 경우 map과 동일한 방식으로 추가해 준다.