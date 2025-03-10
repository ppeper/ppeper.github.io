---
title: "안드로이드 Repository 패턴은 무엇인가"
date: 2022-04-19
update: 2022-04-19
tags:
  - Android
  - MVVM
  - Repository Pattern
series: "Android"
---
<img src="https://user-images.githubusercontent.com/63226023/151594101-266890b7-079a-47c5-9daa-2afbc335ccb7.png">

# Repository
이전의 포스팅에서 안드로이드 [MVVM 디자인 패턴](https://ppeper.github.io/android/android-acc/)에 대해서 알아보면서 아주 잠깐 Repository의 개념 대해서 알아보았다. 이번에 새로운 프로젝트에 Repository를 적용하면서 공부하였던 내용들을 정리해 보려고 한다.

<img src="https://user-images.githubusercontent.com/63226023/163725815-0dc509ff-346b-4f7f-bc4f-cc407c2b2f8d.png">

단어의 의미보면 파악할 수 있듯이 Repository는 필요한 데이터들을 저장하고 있는 저장소라고 생각하면 된다. 그러면 안드로이드에서는 Repository가 어떻게 사용되는가??

## 안드로이드 아키첵처 컴포넌트
<img src="https://user-images.githubusercontent.com/63226023/163726014-6b6a0c25-1d70-47d2-ac4f-37f3972c5549.png">

위의 그림이 안드로이드에서 권장하는 아키텍처이다. 위 다이어그램을 보면 아키텍처의 Activity / Fragment 즉 UI 레이어에서 직접 Data에 접근하지 않고 __Repository__ 를 통하여 데이터를 가지고 오는것을 볼 수 있다.

Repository는 UI에서 사용할 데이터들을 가져올 수 있도록 접근하는 __LocalDataSource(앱 내부 데이터 ex)Room)__, __RemoteDataSource(서버데이터 ex)Retrofit)__ 을 캡슐화하여 사용하는 것이다.

> Repository는 데이터의 출처(local/remote)와 상관없이 동일한 인터페이스로 데이터에 접근할 수 있도록 만든것이다.

## Repository 패턴을 사용하는 이유?
이전에 안드로이드에서 MVC패턴으로 개발을 하게되면 View(Activity/Fragment)에 모든 코드를 작성을 하여 많은 단점들이 존재하였었다.

> View와 Model사이에 의존성이 발생하여 View의 UI 갱신을 위해 Model을 직/간접적으로 참조하여 Activity/Frament의 크기가 커지고 로직들이 복잡해 질 수록 유지보수가 힘들어진다.(처음 안드로이드 프로젝트를 진행하였을때 이러한 문제에 직면하였었다..😅)

그렇다면 Repository 패턴을 사용하게 되면 얻는 이점은 무었일까?

- 도메인과 연관된 모델을 가져오기 위해서 필요한 DataSource가 Presenter 계층에서는 알 필요가 없다(필요한 DataSource가 몇개든 사용될 data만 가져오면 된다)
   - 따라서 DataSource를 새롭게 추가하는 것도 부담이 없다.
- DataSource 의 변경이 되더라도 다른 계층에는 영향이 없다.
- Client는 Repository 인터페이스에 의존하기 때문에 테스트 하기 용이하다.

> 🔔 Repository는 결국 Presenter 계층과 Data 계층간의 Coupling을 느슨하게 만들어 주는것이다.

Repository 패턴을 사용한다는 것은 DataSource 즉, Data Layer를 `캡슐화`한다는 의미이다.

Repository를 추가하여 View에서 데이터를 참조하는 흐름을 보면 다음과 같다. 

> 1. __View -> ViewModel로만 데이터를 가져온다.__ 
>
> 2. __ViewModel -> Repository로 데이터 접근한다.__
>
> 3. __Repository -> DataSource(local/remote)로 부터 데이터를 요청한다.__

- - -

앱에서 회원가입을 구현 해야한다고 했을때 Repository 패턴을 적용하여 요청을 보내는 예시를 보면 다음과 같다.(Hilt 의존성 주입 사용)

> 먼저 UserRemoteDataSource로 부터 서버로 회원가입 요청을 보내는 함수를 만들어준다.

```kotlin
// 서버로 부터 데이터 요청하는 class 인터페이스화
interface UserRemoteDataSource {
    suspend fun signInUser(user: User): Response<ResponseUser>
    .
    .
}

// 인터페이스 UserRemoteDataSource를 구체화할 Impl 클래스
class UserRemoteDataSourceImpl(
    private val userService: UserService
): UserRemoteDataSource {
    override suspend fun signInUser(user: User): Response<ResponseUser> {
        return userService.signInUser(user)
    }
    .
    .
}
```

> UserRepository는 UserRemoteDataSource를 참조하여 서버로 회원가입 요청을 보낸다.

```kotlin
interface UserRepository {
    suspend fun signInUser(user: User): APIResponse<ResponseUser>
    .
    .
}

class UserRepositoryImpl(
    private val userRemoteDataSource: UserRemoteDataSource
): UserRepository {
    // 회원가입 요청이 성공하면 Success에 데이터를 실어서 ,실패하면 Error에 message 리턴
    override suspend fun signInUser(user: User): APIResponse<ResponseUser> {
        val response = userRemoteDataSource.signInUser(user)
        if (response.isSuccessful) {
            response.body()?.let { result ->
                return APIResponse.Success(result)
            }
        }
        return APIResponse.Error(response.message())
    }
    .
    .
}
```

 Retrofit으로 서버의 요청에 대한 응답을 받으면 return값을 `APIResponse` 를 통하여 State와 data를 매핑해 주었다.

```kotlin
sealed class APIResponse<T>(
    val data: T? = null,
    val message: String? = null
) {
    class Success<T>(data: T? = null): APIResponse<T>(data)
    class Loading<T>(data: T? = null): APIResponse<T>(data)
    class Error<T>(message: String, data: T? = null): APIResponse<T>(data, message)
}
```

> UserViewModel은 UserRepository만을 이용해서 데이터에 접근하고 LiveData를 관찰하는 observer에게 값을 넘겨준다.

```kotlin
@HiltViewModel
class UserViewModel @Inject constructor(
    private val repository: UserRepository
): ViewModel() {

    // request state
    val state: MutableLiveData<APIResponse<ResponseUser>> = MutableLiveData()

   fun signInUser(user: User) {
       // 서버의 요청에대한 response가 오기전에는 Loading 상태
       state.value = APIResponse.Loading()
       viewModelScope.launch(Dispatchers.IO)  {
           val response = repository.signInUser(user)
           try {
               if (response.data != null) {
                   state.postValue(response)
               } else {
                   state.postValue(APIResponse.Error(response.message.toString()))
               }
           } catch (e: Exception) {
               state.postValue(APIResponse.Error(e.message.toString()))
           }
       }
   }
   .
   .
}
```

> 사용할 Activity에서 직접 서버로 요청을 하지 않고 UserViewModel을 사용하여 회원가입을 요청(requestSignIn)하고 LiveData를 observe한다.

```kotlin
@AndroidEntryPoint
class LogInActivity : AppCompatActivity() {
    private val userViewModel: UserViewModel by viewModels()
    private lateinit var binding: ActivityLogInBinding

    override fun onCreate(savedInstanceState: Bundle?) {
        binding = ActivityLogInBinding.inflate(layoutInflater)
        super.onCreate(savedInstanceState)
        setContentView(binding.root)


        with(binding) {
            // 로그인 요청
            buttonLogin.setOnClickListener {
                // user의 input 값이 들어갔다고 가정
                    val userLogin =
                        UserLogin(editTextLoginId.text.toString(), editTextLoginPwd.text.toString())
                    requestLogin(userLogin)
                }
            }
        }
    }

    private fun requestLogin(userLogin: UserLogin) {
        userViewModel.getTokenRequest(userLogin)
        userViewModel.isLogin.observe(this@LogInActivity, Observer { response ->
            when (response) {
                is APIResponse.Success -> {
                    // success code
                }
                is APIResponse.Error -> {
                    // error code
                }
                is APIResponse.Loading -> {
                    // loading code
                }
            }
        })
    }
}
```

- - -
# References 
- [https://vagabond95.me/posts/android-repository-pattern/](https://vagabond95.me/posts/android-repository-pattern/)