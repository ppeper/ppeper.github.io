---
title: '안드로이드 Fragment??'
categories: Android
tags: ['Android', 'Fragment']
header:
    teaser: /assets/teasers/android-image.png
last_modified_at: 2022-1-1T00:00:00+09:00
---

- - -
# 프래그먼트란?
프래그먼트는 대표적으로 큰 화면은 가진 태블릿과 같은 기기에서 보다 역동적이고 유연한 UI 디자인을 지원하는 것이 목적이다. 

![image](https://user-images.githubusercontent.com/63226023/147904530-ac4f95b4-3ef3-4b52-a23d-c5ba3ae59414.png)

이러한 큰화면에서 하나의 액티비티를 통한 레이아웃을 구성하기엔 구현하기도 버겁고 유지보수에도 좋지 않을것이다. 이러한 버거움을 해결하고자 나온것이 프래그먼트이다.

프래그먼트는 액티비티 내부에서 독립적으로 앱의 UI(사용자 인터페이스)를 처리한다. 그리고 앱이 실행되는 런타임 시에는 UI를 동적으로 변경하기 위해 프래그먼트를 액티비티에 추가하거나 제거할 수 있다.

프래그먼트는 액티비티의 일부로만 사용될 수 있고, 혼자서 독립적으로 실행되는 앱 요소로는 생성될 수 없다.(다른 액티비티에 재사용할 수 있는 "하위 액티비티"와 같은 개념)

## 프래그먼트 사용법
프래그먼트를 사용하기 위해서는 다음과 같은 방법이 사용된다.
1. `onCreateView()` 사용
2. 액티비티에 프래그먼트 추가
3. 프래그먼트 관리

### 1. `onCreateView()` 사용
프래그먼트 클래스에서 레이아웃을 로드하기 위해서 사용해야한다. 
```kotlin
class FragmentOne : Fragment() {
    
    private var _binding: FragmentExampleBinding? = null
    private val binding get() = _binding!!

    override fun onCreateView(
        inflater: LayoutInflater, container: ViewGroup?,
        savedInstanceState: Bundle?
    ): View? {
        // Inflate the layout for this fragment
        // return inflater.inflate(R.layout.example_fragment, container, false)
        _binding = FragmentExampleBinding.inflate(inflater, container, false)
        return binding.root
    }
}
```
`onCreateView()`를 사용하여 레이아웃을 사용할 준비를 합니다.

- 예시에서 두개의 프래그먼트는 `FragmentTextBinding`, `FragmentToolbarBinding`을 사용.

### 2. 액티비티에 프래그먼트 추가
예시) `fragment_toolbar.xml`과 `fragment_text.xml`을 액티비티에 추가한다. 
```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">

    <androidx.fragment.app.FragmentContainerView
        android:id="@+id/fragmentContainerView"
        android:name="com.kyonggi.fragmentexample.ToolbarFragment"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:layout_marginTop="16dp"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent"
        tools:layout="@layout/fragment_toolbar" />

    <androidx.fragment.app.FragmentContainerView
        android:id="@+id/text_fragment"
        android:name="com.kyonggi.fragmentexample.TextFragment"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginTop="88dp"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toBottomOf="@+id/fragmentContainerView"
        tools:layout="@layout/fragment_text" />

</androidx.constraintlayout.widget.ConstraintLayout>
```
![image](https://user-images.githubusercontent.com/63226023/147905875-efb89ff4-9a3f-4143-bb9a-2991c7668f7c.png)

### 3. 프래그먼트 관리
액티비티 내에서 프래그먼트 트랜잭션(예: 프래그먼트의 추가, 제거, 교체)을 수행하기 위해서는 `FragmentTransaction`에서 가져온 API를 사용해야한다.

```kotlin
val fragmentManager = supportFragmentManager
val fragmentTransaction = fragmentManager.beginTransaction()
```

트랜잭션을 할 준비가 되었다면 `add()` 메서더를 사용하여 프래그먼트와 이를 삽입할 뷰를 지정한다.

`FragmentTransaction`을 변경하고나서 적용을 하기위해서는 `commit()`을 호출해야한다.

```kotlin
val fragment = ExampleFragment()
fragmentTransaction.add(R.id.fragment_container, fragment)
fragmentTransaction.commit()
```
---
[seekBar를 통하여 폰트크기 변경하는 프래그먼트 예시](https://github.com/ppeper/Android_Arctic-Fox/tree/main/FragmentExample)

# References
- [안드로이드 Fragment](https://developer.android.com/guide/components/fragments)





