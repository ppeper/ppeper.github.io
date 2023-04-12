---
title: 안드로이드 Fragment 조각내기
categories: Android
tags: ['Fragment', 'FragmentTransaction', 'Android']
header:
    teaser: /assets/teasers/android-fragment-image.png
last_modified_at: 2023-04-12T00:00:00+09:00
---
안드로이드에서 UI 화면을 태블릿과 같은 큰 화면에서 역동적이고 유연한 디자인을 하기 위하여 Fragment가 나오게 되었다.

Fragment는 Activity와 마찬가지로 자체적인 생명주기를 가지지만 Activity의 생명주기에 영향을 받기 때문에 좀 더 신경쓰고 조심해야 하는 부분들이 많다. 처음 적용하고 공부하였을 때 신경쓰지 못하였던 Fragment의 내용을 보려고 한다. 
- - -
# Fragment LifeCycle
<p align="center"><img src="https://user-images.githubusercontent.com/63226023/231318855-b5603bfb-265c-4f08-bd2d-5eb5c868cc54.png" width="50%"/></p>

Fragment는 Activity와 다르게 View와 Fragment간의 LifeCycle이 다르다.

<h4>onAttach()</h4>

- Fragment가 Activity에 붙을(Attach) 때 호출된다.
- 아직 Fragment가 완벽하게 생성된 단계는 아니다.

<h4>onCreate()</h4>

- 본격적으로 Fragment가 Activity의 호출을 받아 생성되는 단계
- Fragment를 생성하면서 넘겨준 값들이 있다면, 이 단계에서 값을 꺼내 세팅한ㄷ.
- UI 초기화는 진행할 수 없다.

<h4>onCreateView()</h4>

- Layout을 inflate하는 단계. 뷰바인딩을 진행한다.
- View가 초기화되는 중이기 때문에, UI 초기화 작업을 진행하면 충돌이 일어날 수 있다.

<h4>onViewCreate()</h4>

- UI 초기화 진행
- onViewCreated()는 View 생성이 완료되었을 때 호출되는 메서드

<h4>onStart()</h4>

- Fragment가 사용자에게 보여 지기 직전에 호출되는 단계
- 해당 메서드가 호출되었을 때, Activity는 started 상태

<h4>onResume()</h4>

- 사용자와의 상호작용을 시작하는 단계
- 특정 이벤트가 발생하여 포커스가 떠날 때까지 onResume 단계에 머무름
- 프로그램이 일시정지되면 onPause()가 호출되고, 다시 재개되면 onResume()을 다시 호출함
- Resume 상태로 전환될 때마다 진행해야 되는 초기화 작업들을 세팅

<h4>onPause()</h4>

- 사용자가 Fragment를 떠나면 가장 먼저 onPause()를 호출
- Fragment가 사용자와의 상호작용을 중지하는 단계
- 사용자가 해당 Fragment로 돌아오지 않을 수도 있으므로, 지속되어야 하는 변경사항을 onPause에서 저장

<h4>onStop()</h4>

- 다른 Activity가 화면을 완전히 가리거나, 화면이 더이상 보여지지 않게 되는 상황에서 호출된다.
- 화면이 보이지 않을 때 실행할 필요가 없는 기능들을 정지시켜 줄 수 있다.

<h4>onDestroyView()</h4>

- Fragment와 연결된 View Layer가 제거되는 중일 때 호출되는 단계

<h4>onDestroy()</h4>

- Fragment가 제거되기 직전 단게
- Fragment가 생성될 때 onCreate → onCreateView 순으로 호출된 것과 달리, 파괴는 onDestroyView에서 View를 제거한 후 onDestroy가 호출된다.

## getViewLifecycleOwner() vs getLifecycleOwner()

<h4>LifecycleOwner</h4>
LifecycleOwner는 __Fragment의 생명주기__ 를 가지고 있는 클래스로 `onAttach()` ~ `onDestroy()`와 연결되어 있다.

<h4>viewLifecycleOwner</h4>
viewLifecycleOwner는 __Fragment View의 생명주기__ 를 가지고 있는 클래스로 `onCreateView()` ~ `onDestroyView()` 와 연결되어 있다.

> 📍viewLifecycle은 Fragment LifeCycle 보다 생명주기가 짧다!


이러한 서로 다른 생명주기에 따라 대표적으로 liveData를 사용할때 매개변수로 `this`로 넘겨주었다면 __경우에 따라 Fragment가 Destroy 되지 않았지만 새로운 observer가 등록되는 경우__ 생길 수 있다.

```kotlin
// onAttach() ~ onDestroy()
liveData.observe(this) { it -> 
  // this(fragment) -> Destroy 되지 않았을 때 다시 불릴 가능성
}

// onCreateView() ~ onDestroyView()
liveData.observe(viewLifecycleOwner) { it -> 
  ...
}
```

구글에서 기본으로 제공하는 Fragment 클래스에서도 View와 Fragment의 생명주기가 달라 메모리 누수가 발생할 수 있기 때문에 이에 코드를 볼 수 있다.

```kotlin
  private var _binding: ResultProfileBinding? = null
  // This property is only valid between onCreateView and
  // onDestroyView.
  private val binding get() = _binding!!

  override fun onCreateView(
      inflater: LayoutInflater,
      container: ViewGroup?,
      savedInstanceState: Bundle?
  ): View? {
      _binding = ResultProfileBinding.inflate(inflater, container, false)
      val view = binding.root
      return view
  }

  override fun onDestroyView() {
      super.onDestroyView()
      _binding = null
  }  
```

Fragment는 제거되지 않았지만 `onCreateView()`는 다시 사용할 수 있기 때문에 명시적으로 `onDestroyView()`에서 binding에 할당을 취소해 가비지 컬렉션이 되도록 해야한다. 

- - -
# FragmentManager

<p align="center"><img src="https://user-images.githubusercontent.com/63226023/231444667-211d675f-5ce3-4b11-8b55-f745849df216.png" width="70%"/></p>

- FragmentManager 속성은 호출된 Fragment 계층 구조에서 어디에 있는지에 따라 어떤 FragmentManager에 액세스하는지 다르다.
- FragmentManager는 Activity와 Fragment의 중간에서 서로를 이어주는 역할을 하여 Fragment Transaction 수행해 준다.
- FragmentManager은 백 스택(Back Stack) 에 Fragment 추가/교체/삭제 작업에 의한 변경 사항을 push 및 pop 하는 작업을 담당한다.

> - `findFragmentById()` : Fragment 컨테이너 내의 현재 Fragment 참조
>    - UI를 제공하는 Fragment를 참조할 때 사용

```kotlin
val fragment = supportFragmentManager.findFragmentById(binding.fragmentContainer.id)
```

> - `findFragmentByTag()` : Fragment에 고유한 태그를 부여한 후 태그를 이용해 Fragment 참조

```kotlin
val fragment = supportFragmentManager.findFragmentByTag("my_fragment")
```
- - -
# Fragment Transaction

Fragment Transaction의 인스턴스는 FragmentManager의 beginTransaction() 메서드를 통해 얻을 수 있다

```kotlin
val manager = supportFragmentManager.beginTransaction()
```
Android Jetpack Fragment 라이브러리에서 제공하는 클래스로 Transaction 클래스는 Fragment 추가/교체/삭제 작업을 제공한다.

- Fragment Transaction은 FragmentManager의 단일 수행 단위
- 하나의 Fragment Transaction 단위 내에 Fragment Transaction 클래스가 제공하는 Fragment 추가/교체/삭제 작업 등을 명시하면 된다.
- 하나의 Fragment Transaction 단위 내에 작성된 Fragment 조작 관련 작업들은 해당 Fragment Transaction 수행될 때 모두 실행된다.

## Transaction 메소드
Fragment의 Transaction 메소드들은 `add()`, `replace()`, `commit()`, `commitAllowingStateloss()`, `remove()`, `addToBackStack()` 등이 있다.

> __add()__
> - 기존에 있던 Fragment를 지우는 것이 아니라 그 위에 다시 추가한다.
>
> __replace()__
> - 기본에 있던 Fragment를 제거한 후 새로운 Fragment를 추가한다.
>
> __remove()__
> - 기존에 있던 Fragment를 삭제한다.  
  > - fragment를 찾기 위해 `findFragmentById()`, `findFragmentByTag()` 메서드로 제거할 fragment를 찾는다
>
> __commit()__
> - FragmentTransaction을 생성하고 Fragment의 추가/교체/삭제 작업을 명시한 후 __반드시 마지막에 commit__ 을 해줘야 한다.
>
> __addToBackStack()__
> - 해당 메소드를 해준 후 commit()을 하게 되면 하기 전까지의 모든 변경 내용이 하나의 Transaction으로 backStack에 들어간다.

## 📌 commit() vs commitNow()

commit()에는 `Now` 가 붙은 메소드들이 있다. commit() 메서드는 __비동기로 처리되는 함수__ 이기 때문에 commit 함수 호출 시점에 바로 Transaction이 즉시 수행되는 것이 아니라 Main Thread에 예약된다.

반면에 commitNow() 메서드는 호출 시점에 즉시 해당 Fragment Transaction이 __동기적으로 수행__ 된다.

<p align="center"><img src="https://user-images.githubusercontent.com/63226023/231384985-1a4605eb-ce73-454a-8a91-f1069211ce45.png" width="50%"/></p>

> commitNow()는 동기적으로 실행하기 때문에 `addToBackStack()` 를 통하여 백스택에 Transaction을 추가하려는 경우 프레임워크는 명령에 대한 보증을 할 수 없기 때문에 사용할 수 없다.

## 📌 commit() vs commitAllowingStateLoss()
Fragment를 사용을 해보았다면 `onSaveInstanceState()` 이후 commit()을 수행할 수 없다는 `IllegalStateException` 을 봤을 것이다.

<p align="center"><img src="https://user-images.githubusercontent.com/63226023/231441130-7a39f286-91f8-4994-bd48-2bcd813fcf59.png"/></p>

- Fragment Transaction은 호스트 Activity가 자신의 상태(RESUMED, STOPPED 등)를 저장하기 전에 생성되고 commit 되어야 한다.
- 만약 호스트 Activity가 `onSaveInstanceState()` 메소드를 호출한 후에 Fragment Transaction이 commit 된다면 에러가 발생한다.
    - Activity의 상태가 저장될 때 자신에서 __호스팅되어 있는 Fragment의 상태도 저장__ 하게 되는데, __이러한 상태 저장 후 Fragment Transaction에 의해 Fragment 추가/교체/삭제 작업이 일어나면 Activity가 저장한 Fragment 상태와 달라__ 지기 때문이다.

> 📍`commitAllowingStateLoss()` 는 commit()과 거의 동일 하지만 상태와 달라질 수 있는(상태 손실)을 허용한다.

---

# References 
- [https://developer.android.com/guide/fragments?hl=ko](https://developer.android.com/guide/fragments?hl=ko)
- [https://medium.com/hongbeomi-dev/%EB%B2%88%EC%97%AD-%EB%8B%A4%EC%96%91%ED%95%9C-%EC%A2%85%EB%A5%98%EC%9D%98-commit-8f646697559f](https://medium.com/hongbeomi-dev/%EB%B2%88%EC%97%AD-%EB%8B%A4%EC%96%91%ED%95%9C-%EC%A2%85%EB%A5%98%EC%9D%98-commit-8f646697559f)