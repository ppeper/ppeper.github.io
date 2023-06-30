---
title: 안드로이드 Navigation 사용하기
categories: Android
tags: ['Android', 'Navigation', 'SafeArgs']
header:
    teaser: /assets/teasers/android-navigation-image.png
last_modified_at: 2023-07-01T00:00:00+09:00
---
# SAA (Single Activity Architecture)
SAA는 Google IO 2018에 소개된 개념으로 하나 혹은 적은 수의 Activity로 애플리케이션을 개발하는 것을 말한다.

Fragment에 비해 상대적으로 무거운 Activity의 사용을 자제하고, __Navigation__ 을 사용하여 한 눈에 앱의 흐름을 보여주고 유연한 화면 구성을 할 수 있다.

- Google IO 2018에 소개된 개념
- 하나 혹은 적은 수의 Activity로 어플리케이션을 개발
- Fragment에 비해서 무거운 Activity 사용을 자제하고, Navigation을 사용한 유연한 화면 구성

## 📌Navigation 구성 요소

- `Navigation Graph`
    - Navigation 정보들을 한 곳에 모은 XML 리소스이다. 앱에서 진행될 수 있는 모든 흐름을 보여주고 앱 내 Fragment를 한 눈에 확인하기 쉽다.

- `Navigation Controller`
    - Navigation Host에서 App Navigation을 관리해주는 객체이다. 앱에서 그래프대로 이동할 때 Navigation Host에서 컨텐츠 전환을 담당한다.

- `Navigation Host`
    - Navigation Graph에서 대상을 표시하는 빈 컨테이너이다. 대상 구성요소에는 Fragment 대상을 표시하는 기본 Navigation Host 구현인 NavHostFragment가 포함된다.


`Navigation Graph`에서 특정 경로를 따라 이동하거나 특정 대상으로 직접 이동하는 등 `Navigation Controller`에서 전달하고 Navigation Controller가 `Navigation Host`에 적절한 대상을 표시한다.

---

### 의존성 설정

```gradle
// Jetpack Navigation Kotlin
def nav_version = "latest_version"
implementation "androidx.navigation:navigation-fragment-ktx:$nav_version"
implementation "androidx.navigation:navigation-ui-ktx:$nav_version"
```

### 시작화면
SAA로 MainActivity의 FragmentContinerView를 가지는 xml코드 예시는 아래와 같다.


Navigation을 사용하기 위해 `app:navGraph` 속성에 navigation graph xml을 지정한다. 

```xml
<FrameLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:background="@android:color/background_light">

    <androidx.fragment.app.FragmentContainerView
            android:id="@+id/my_nav_host_fragment"
            android:name="androidx.navigation.fragment.NavHostFragment"
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            app:defaultNavHost="true"
            app:layout_constraintBottom_toBottomOf="parent"
            app:layout_constraintEnd_toEndOf="parent"
            app:layout_constraintStart_toStartOf="parent"
            app:layout_constraintTop_toTopOf="parent"
            app:navGraph="@navigation/navigation"/>
</FrameLayout>
```

### navigation.xml

위와 같이 안드로이드 스튜디오에서 새로운 Fragment들을 추가할 수 있다.

<img src="https://user-images.githubusercontent.com/63226023/235952947-f5387b1d-286b-416a-95ff-11f9dbef9061.png">

화면의 시작 위치를 시정하기 위해 `startDestination` 속성으로 처음위치를 지정한다.

```xml
<navigation xmlns:android="http://schemas.android.com/apk/res/android"
            xmlns:app="http://schemas.android.com/apk/res-auto"
            xmlns:tools="http://schemas.android.com/tools"
            app:startDestination="@+id/title_screen">
```

생성된 Fragment들은 Navigation Graph에서 drag 하여 관계를 설정할 수 있다.
<img src="https://user-images.githubusercontent.com/63226023/235955778-b56365ff-915e-44b3-b92a-27d007433e14.png">

xml에서 생성되는 코드를 보게되면 `action` 태그에 destination이 생성되는 것을 볼 수 있다.
```xml
<fragment
        android:id="@+id/title_screen"
        android:label="fragment_title_screen"
        tools:layout="@layout/fragment_title_screen">
    <action
        android:id="@+id/action_title_screen_to_leaderboard"
        app:destination="@id/leaderboard" />
</fragment>
...
```

## 화면이동
TitleScreenFragment에서 해당 action 명으로 `findNavController`로 navigationController를 찾아와서 navigate method를 사용한다.

```kotlin
class TitleScreenFragment : Fragment() {
    override fun onCreateView(
        inflater: LayoutInflater, container: ViewGroup?, savedInstanceState: Bundle?
    ): View {
        val binding = FragmentTitleScreenBinding.inflate(inflater, container, false)
        
        binding.leaderboardBtn.setOnClickListener {
            Navigation.findNavController(binding.root).navigate(R.id.action_title_screen_to_leaderboard)
        }
        return binding.root
    }
}
```

# Safe Args를 이용한 값 전달

- Safe Args: 값을 전달하고 탐색을 지원하는 Gradle plugin으로 기본의 bundle로 값을 전달하는 방식 대신에 사용이 가능하다.

## 프로젝트 레벨의 plugin 설정

```gradle
plugins {
    ...
    // safeArgs
    id 'androidx.navigation.safeargs' version '2.4.2' apply false
}
```

## 모듈레벨의 plugin 설정

```gradle
plugins {
    ...
    id 'androidx.navigation.safeargs.kotlin'
}
```


- 화면간 데이터를 전달할때는 많은 데이터를 전달하는 것보다는 최소한의 데이터를 전달하는 것이 좋다.
    - 객체를 전달하지 않고, key만 전달하는것이 효과적이다.


- navigation.xml에 argument 태그로 전달받을 값을 지정한다.


- 보내는 클래스 설정: Bundle방식과 SafeArgs 방식

```kotlin
// 1. Bundle로 담기
val bundle = bundleOf("userName" to myDataset[position])
Navigation.findNavController(holder.item)
      .navigate(R.id.action_leaderboard_to_userProfile, bundle)

// 받는 곳
val name = arguments?.getString("userName") ?: "Ali Connors" 
```

- Safe Args 를 사용 설정하면 송신/수신을 위한 대상 파일과 메서드들이 만들어진다.
- 생성된 클래스 이름에는 `Directions` 가 추가된다.

```kotlin
// 2. Safe Args.
val action = LeaderboardDirections.actionLeaderboardToUserProfile(myDataset[position])
Navigation.findNavController(holder.item).navigate(action)

// 받는 곳
val name = args.userName
```

## Arguments를 받는쪽

```kotlin
class UserProfile : Fragment() {
    // safe args 방식으로 받기
    val args: UserProfileArgs by navArgs()
    override fun onCreateView(
        inflater: LayoutInflater, container: ViewGroup?, savedInstanceState: Bundle?
    ): View {
        //safe args 방식
        val name = args.userName
    }
}
```

## Popup Behavior

- backstack에 쌓는 방식을 설정한다.
- `popUpTo`
    - 주어진 화면이 나올때까지 BackStack을 팝 하면서, 이동한 화면을 stack에 넣는다.
- `popUpToInclusive`: true
    - 주어진 화면이 나올때까지 BackStack을 팝 하면서, 현재 화면은 stack에 넣지 않는다.

<p align="center"><img src="https://github.com/ppeper/Cellification/assets/63226023/bf6454d5-161b-4f66-b7aa-c57213743a0b" width="50%"></p>

- - -
# References 
- [NavigationUI로 UI 구성요소 업데이트](https://developer.android.com/guide/navigation/navigation-ui?hl=ko)