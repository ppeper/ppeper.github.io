---
title: 안드로이드 MVVM 패턴과 ACC 알아보기
categories: Android
tags: ['Android', 'MVVM 패턴', 'ACC']
header:
    teaser: /assets/teasers/android-jetpack-light.png
last_modified_at: 2022-1-29T00:00:00+09:00
---
<img src="https://user-images.githubusercontent.com/63226023/151594101-266890b7-079a-47c5-9daa-2afbc335ccb7.png">

안드로이드 앱 개발을 시작하면 __Activity에 거의 모든 동작하는 코드__ 를 직접 넣는다. 이렇게 모든 코드를 넣다보면 __구조화된 코드의 작성이 없게되고 추후에 유지 보수__ 가 어려워진다.

처음 안드로이드 프로젝트를 진행하면서 __기능을 바꾸거나 추가할 때 어느 부분을 수정해야하는지__ 굉장히 난해했던 경험이 있었고 자연스럽게 디자인 패턴을 공부를 하고 적용을 해봐야겠다는 생각이 들었다.😎 

<p align="center"><img src="https://user-images.githubusercontent.com/63226023/151658824-a9a66cb5-dadb-4931-b03a-eba217a1c345.png" width="70%"></p>


> 이렇게 모든 동작들을 하나의 Acitivity에 넣다보면 스파게티 코드가 된다...


다행이 구글에서 `Jetpack`을 출시하여 __안드로이드 아키텍쳐 컴포넌트(ACC)__ 를 포함하여 __MVVM 패턴__ 에 따른 앱 개발을 하도록 제공하고 있다.   
이번 포스팅은 __안드로이드 아키첵처 컴포넌트(ACC)__ 와 __MVVM 패턴__ 에 대해 정리를 해보려고 한다.

- - -

# 📍 MVC vs MVVM
MVVM 디자인 패턴을 알아보기 전에 이전에 사용하던 MVC 패턴을 알아보자!

MVC 패턴은 __Model-View-Controller__ 의 약자로 기능별로 나타내면 다음과 같다.

- Model : Data와 관련한 처리를 한다.
- View : Controller로 부터 알림을 받고, Model을 참조하여 사용자가 볼 화면을 구성한다.
- Controller : 사용자의 입력을 받는다.

<img src="https://user-images.githubusercontent.com/63226023/151660484-18f2e9c3-8733-46b2-ac8a-9e9c89a054d4.png">


> 1. __Controller__ 는 사용자의 입력을 받는다.   
> 2. Controller는 입력에 따라서 __Model의 갱신__ 을 요청한다.
> 3. __View__ 는 __Model을 참조__ 하여 __사용자가 볼 화면(UI)를 업데이트__ 한다.

안드로이드에서는 __Controller__ 의 역할과 __View__ 의 역할을 함께하게 된다(OnClickListener 등). 따라서 기존의 단순한 형태의 MVC 패턴에 따른 Activity에 모든 코드를 넣으면 이런 단점들이 발생하였다.

- 앱에 기능을 추가할때 마다 __Activity의 코드양이 많아짐__
- __View와 Model 간의 결합도__ 가 높아짐
- __코드 복잡도__ 가 높아질 수 있음
- __유지보수__ 가 어려워짐

따라서 MVC 패턴은 앱의 규모가 커지면 커질수록 유지보수가 굉장이 어려워 진다는 단점이 있다. 이러한 __MVC 패턴의 단점__ 을 보완하기 위하여 나온것이 __MVVM 패턴__ 이다.

__MVVM 패턴__ 은 기존 MVC에서 Model과 View는 동일하지만 __ViewModel__ 을 추가한 형태이다. 

<img src="https://user-images.githubusercontent.com/63226023/151661992-50183d81-a040-4d8d-91d1-8a89a3853b44.png" width="60%">


앞서 말한 MVC 패턴은 Controller(Activity)가 View와 Model간 상호작용을 하였다.   
MVVM 패턴에서는 __View__ 는 __ViewModel의 데이터를 관찰__ 하여 UI를 갱신한다. 또한 __ViewModel__ 은 __Model__ 로부터 데이터의 갱신을 및 요청한 데이터를 받는다. 

결론적으로 __View에서 사용자에게 보여줄 데이터__ 는 __ViewModel__ 이 가지고 있고 이를 __관찰__ 하기 때문에 `MVC 패턴과 다르게` __View가 데이터를 가져오기위해 DB에 접근을 하지 않고 오직 UI 갱신에만 신경쓰게 된다.__

따라서 MVVM 패턴을 적용하게 된다면 안드로이드에서 __다음과 같은 장점을 가진다.__

- View의 UI 갱신이 간단하다.(ViewModel의 데이터를 관찰하고 있기 때문이다)
- ViewModel이 데이터를 가지고 있으므로 __메모리 누수__ 방지 (View와 Model간의 상호작용이 없어 생명주기에 의존하지 않는다)
- 기능별로 __모듈화__ 가 잘 되어 있기 때문에 유지 보수가 용이.

이러한 MVVM 패턴을 잘 활용한다면 좋은 앱을 개발할 수 있을것이다. 그렇지만 MVC 패턴은 쉽게 구현을 할 수 있었지만 MVVM 패턴은 __구조가 복잡하다는 단점__ 이 있다.

구글에서는 이러한 __MVVM 패턴__ 을 비교적 간편하게 적용할 수 있도록 __ACC__ 라는 것을 제공한다.


# 안드로이드 아키텍처 컴포넌트(ACC)
구글에서 제공하는 안드로이드 Jetpack의 구성요소중 하나인 ACC 는 아래의 그림과 같다.

<img src="https://user-images.githubusercontent.com/63226023/151663025-31f02dd5-30e2-4035-b924-12d9df521945.png">

## ViewModel
ViewModel은 앞서 설명한대로 UI를 갱신할 __데이터만을 가지고 있으며__ View와 분리되어 있어서 __생명주기에 영향__ 을 미치지 않는다.(예: 앱의 회전등이 일어날때)

## LiveData
LiveData는 데이터 홀더이며 저장된 데이터(ViewModel)을 관찰할 수 있게(Obserable) 해준다. 또한 Activity / Fragment의 생명주기를 인식하여 활동을 하고 있을 때만(Activity가 활성화) 동작을 하게되어 이는 __메모리 누수__ 의 발생을 줄여준다.

## RoomDatabase
RoomDatabase는 __SQLite 데이터베이스를 쉽게 사용__ 할 수 있도록 해주는 라이브러리이다. 별도의 쿼리문을 쓰지 않더라도 더욱 직관적이고 편하게 데이터베이스를 사용할 수 있다.

## Repository
ViewModel이 데이터베이스나 외부의 웹 서비스로부터 데이터를 주고받기 위해, 데이터 API를 들고 있는 클래스이다. 따라서 ViewModel은 직접 데이터베이스나 웹 서비스에 접근하지 않고 Repository에 접근하여 일관성 있게 데이터를 가져올 수 있다.

- - -
# 정리하며👍
구글에서 제공하는 __ACC__ 의 여러가지 구성 요소들을 활용하여 __MVVM 패턴__ 을 적용하고 사용할 수 있다. 
> ACC와 MVVM에 대해선 알겠는데 이를 적용시킬수 있을까..😅😅

MVVM 패턴이 __좋은것은 알겠지만 이를 어떻게 적용을 시킬지에 대한 의문__ 과 ACC 가 도움을 주는지는 알겠지만 __사용을 어떻게 해야할 지 아직까지 잘 생각이 들지 않는다.__

앞으로 ACC의 구성 요소와 많은 코드, 예제를 보면서 __패키지의 구조 및 코드의 구현 방식__ 을 많이 접해봐야겠다는 생각이 들었다. 

