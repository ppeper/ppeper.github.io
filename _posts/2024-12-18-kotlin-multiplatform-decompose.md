---
title: Kotlin Multiplatform - Decompose
categories: Kotlin Multiplatform
tags: ['Kotlin Multiplatform']
header:
    teaser: /assets/teasers/multiplatform-decompose-image.png
last_modified_at: 2024-12-18T00:00:00+09:00
---

### 개요

> 최근에 Kotlin Multiplatform을 활용하여 안드로이드 태블릿과 Desktop에서 사용할 서비스들을 개발하고 있습니다.
> 
> 
> 안드로이드에서는 화면간에 Jetpack Compose Navigation을 사용하고 있지만 현재 Android에서만 사용이 가능하여 대체할 여러 라이브러리를 찾아보던 도중 Decompose를 알게되어 한번 공유해보고자 합니다.
> 

## Decompose??

Decompose 는 멀티 플랫폼에서 사용할 수 있는 라이브러리로 공식 문서에서는 Lifecycle-aware한 비즈니스 컴포넌트(Business Logic Component, BLoC)로 코드를 분해해 준다고 설명하고 있다.


> 💡 [BLoC](https://github.com/felangel/bloc) 이란 키워드를 처음 접하였는데 이는 Flutter에서 쓰이는 상태관리 패턴중 하나로
UI와 비즈니스 로직의 분리를 강화하는 디자인 패턴이라고 합니다. 


공식 문서의 Decompose의 주요 장점으로는 Navigation의 역할 (argument 전달, 뒤로가기), 딥링크도 지원을 하며, State Holder로써의 역할을 해줄 수 있다.

또한 가장 중요한 Lifecycle aware Component로써의 역할을 할 수 있는데, 이는 Android 내에서 Activity와 같이 수명 주기 상태 변경에 따라 작업을 실행 할 수 있다.

## Decompose의 구성 요소

코드를 살펴보기 전에 Decompose에 사용된 주요 구성 요소를 살펴보겠습니다.

- ComponentContext
    - 수명 주기를 관리하여 수명주기 이벤트를 관찰할 수 있도록 한다.
- Value
    - 상태 홀더 역할을 하며 현재 상태를 반환한다. (Android의 stateflow나 LiveData와 비슷한 느낌)
- ChildStack
    - 요소간의 탐색을 관리한다. (push, pop, replace)
- StackNavigation
    - 현재 표시되는 구성을 유지하므로 탐색 관리가 용이해진다.

---

Decompose를 보면서 Android에서 Lifecycle, 상태, Navigation의 핵심 기능을 Multiplatform 환경에서 사용할 수 있도록 제공하고 있구나 생각이 들었습니다. 

Item 리스트와 Detail 화면을 이동하는 간단한 화면을 만들어보면서 구성요소들 및 사용법을 한번 살펴보고자 합니다.

```kotlin
@Serializable
data class Item(
    val title: String
)
```

## libs.versions.toml

```toml
[versions]
...
decompose = "<latest_version>"
kotlin = "<latest_version>"

[libraries]
...
decompose = { module = "com.arkivanov.decompose:decompose", version.ref = "decompose" }
decompose-extensions-compose = { module = "com.arkivanov.decompose:extensions-compose", version.ref = "decompose" }

[plugins]
kotlinSerialization = { id = "org.jetbrains.kotlin.plugin.serialization", version.ref = "kotlin"}
```

shared 모듈에 있는 에 있는 `build.gradle.kts` 에 의존성을 추가해준다.

```kotlin
plugins {
    ...
    alias(libs.plugins.kotlinSerialization)
}

kotlin {
    ...
    sourceSets {
        commonMain.dependencies {
        ...
        implementation(libs.decompose)
        implementation(libs.decompose.extensions.compose)
        }
    }
}
```

> 💡 kotlin-serialization의 경우 Navigation 통해 데이터를 다른 화면으로 전달 시 직렬화 과정이 필요하기 때문에 이를 위해 사용하였다.

## RootComponent

Decompose에서 가장 핵심이 퇴는 클래스로 애플리케이션이 실행되는 동안 다른 SubComponent들을 관리하는 Host 역할을 한다. 

해당 클래스에서 Navigation들을 정의하고 화면에 대한 이동, 백스택, Args 들을 관리한다. Decompose에서도 안드로이드와 마찬가지로 Stack을 사용하여 화면을 쌓아가며 관리한다.

```kotlin
class RootComponent(
    componentContext: ComponentContext
): ComponentContext by componentContext {

    private val navigation = StackNavigation<Configuration>()
    val childStack = childStack(
        source = navigation,
        serializer = Configuration.serializer(),
        initialConfiguration = Configuration.ItemList,
        handleBackButton = true,
        childFactory = ::createChild
    )
    
    private fun createChild(
        config: Configuration,
        context: ComponentContext
    ): Child {
        return when (config) {
            is Configuration.ItemList -> Child.ItemList(
                ItemListComponent(
                    componentContext = context,
                    onNavigateToDetail = { item ->
                        navigation.pushNew(Configuration.ItemDetail(item))
                    }
                )
            )
            is Configuration.ItemDetail -> Child.ItemDetail(
                ItemDetailComponent(
                    item = config.item,
                    componentContext = context,
                    onBackPressed = {
                        navigation.pop()
                    }
                )
            )
        }
    }
}
```

안드로이드 Navigation에서 NavHost와 같은 중심이 되는 RootComponent를 생성해 주기위해서는 `ComponentContext` 를 만들어 주어야 한다. 이때 기본 값이 되는 DefaultComponentContext 를 사용하여 Lifecycle만을 가지고 있는 구현체를 생성해 줄 수 있다.

### ComponentContext?

Decompose에서 Component는 로직을 캡슐화하는 클래스로 각자 관리되는 요소들이 있다. 기본적으로 Component 클래스를 만들때 `ComponentContext` 인터페이스에 의해 구현된 각 요소들을 가지게 된다.

1. LifecycleOwner
2. StateKeeperOwner
3. InstanceKeeperOwner
4. BackHandlerOwner

> 각 요소들의 역할은 [문서](https://arkivanov.github.io/Decompose/component/overview/#componentcontext)에서 자세하게 확인해 볼 수 있다.

```kotlin
class DefaultComponentContext(
    override val lifecycle: Lifecycle,
    stateKeeper: StateKeeper? = null,
    instanceKeeper: InstanceKeeper? = null,
    backHandler: BackHandler? = null,
) : ComponentContext {

    override val stateKeeper: StateKeeper = stateKeeper ?: StateKeeperDispatcher()
    override val instanceKeeper: InstanceKeeper = instanceKeeper ?: InstanceKeeperDispatcher().attachTo(lifecycle)
    override val backHandler: BackHandler = backHandler ?: BackDispatcher()
    override val componentContextFactory: ComponentContextFactory<ComponentContext> = ComponentContextFactory(::DefaultComponentContext)

    constructor(lifecycle: Lifecycle) : this(
        lifecycle = lifecycle,
        stateKeeper = null,
        instanceKeeper = null,
        backHandler = null,
    )
}
```

### Configuration

Decompose navigation에서 사용하는 용어로 child component를 표현하고 argument들을 담고 있는 클래스들이다.

```kotlin
@Serializable
sealed class Configuration {
    @Serializable
    data object ItemList: Configuration()
    @Serializable
    data class ItemDetail(val item: Item): Configuration()
}
```

### Child Stack

컴포넌트들은 각자 생명주기를 가진다. 새 컴포넌트가 push되면 원래 활성화 되어있던 컴포넌트는 stopped 상태가 되고, 컴포넌트가 pop되면 그 앞의 컴포넌트가 resumed 상태가 된다. 

즉, 백스택에 있는 동안 컴포넌트의 비즈니스 로직을 중단하지 않고 실행한 채로 둘 수 있다.

Child Stack은 2가지 main entity로 이루어져있다.

- ChildStack: Component와 Configuration을 담고 있는 data class이다.
- StackNavigation: navigation 명령을 수행하고 구독한 옵저버들에게 메세지를 전달한다.

```kotlin
private val navigation = StackNavigation<Configuration>()
val childStack = childStack(
    source = navigation,
    serializer = Configuration.serializer(),
    initialConfiguration = Configuration.ItemList,
    handleBackButton = true,
    childFactory = ::createChild
)
```

> 📌 [Decompose Navigation](https://arkivanov.github.io/Decompose/navigation/stack/navigation/) 를 확인하면 console 예시와 함께 다양한 명령들을 확인 할 수 있다.

위에서 말하는 Child는 각 Screen 하나하나를 말한다고 볼 수 있다. 해당 컴포넌트는 부모 컴포넌트의 생명주기가 끝나면 자동으로 같이 파괴된다.

## Child Component

```kotlin
class ItemDetailComponent(
    componentContext: ComponentContext,
    item: Item,
    private val onBackPressed: () -> Unit
): ComponentContext by componentContext {

    val state: Value<Item> = MutableValue(item)

    fun onBack() {
        onBackPressed()
    }
}

class ItemListComponent(
    componentContext: ComponentContext,
    private val onNavigateToDetail: (Item) -> Unit
): ComponentContext by componentContext {

    private var _list = MutableValue(
        (1..10).map { Item("Decompose Item $it") }
    )
    val list: Value<List<Item>> = _list

    fun onItemClick(item: Item) {
        onNavigateToDetail(item)
    }
}
```

각 Screen의 Component에서는 Decompose에서 `Value` 를 통해 안드로이드에서의 StateFlow, LiveData와 같이 State들을 Holder할 수 있는 기능을 제공하며 Decompose-extension에서 제공하는 subscribeAsState()로 변화들을 옵저빙 할 수 있다.

```kotlin
val state by component.list.subscribeAsState()
```

리스트 화면과 Detail 화면에서 사용 예시를 보면 다음과 같다.

```kotlin
@Composable
fun ItemListScreen(
    component: ItemListComponent,
) {
    val state by component.list.subscribeAsState()

    LazyColumn(
        modifier = Modifier.fillMaxSize(),
        state = rememberLazyListState(),
        contentPadding = PaddingValues(16.dp),
        verticalArrangement = Arrangement.spacedBy(8.dp)
    ) {
        items(state) { item ->
            Row(
                modifier = Modifier
                    .fillMaxWidth()
                    .clip(RoundedCornerShape(12.dp))
                    .border(
                        border = BorderStroke(
                            width = 1.dp,
                            color = Color.LightGray
                        ),
                        shape = RoundedCornerShape(12.dp)
                    )
                    .clickable { component.onItemClick(item) }
                    .padding(16.dp),
                horizontalArrangement = Arrangement.SpaceBetween,
                verticalAlignment = Alignment.CenterVertically
            ) {
                Text(item.title)
            }
        }
    }
}

@Composable
fun ItemDetailScreen(
    component: ItemDetailComponent
) {
    val item by component.state.subscribeAsState()

    Scaffold(
        modifier = Modifier
            .fillMaxSize(),
        topBar = {
            TopAppBar(
                title = { Text("Decompose Example") },
                navigationIcon = {
                    IconButton(
                        onClick = {
                            component.onBack()
                        }
                    ) {
                        Icon(
                            imageVector = Icons.AutoMirrored.Filled.ArrowBack,
                            contentDescription = "Back"
                        )
                    }
                }
            )
        }
    ) {
        Box(
            modifier = Modifier.fillMaxSize(),
            contentAlignment = Alignment.Center
        ) {
            Text(text = item.title)
        }
    }
}
```

## Application 실행

각 플랫폼에서 사용할 App Composable 함수에서는 앞서 만든 `RootComponent` 를 사용하여 childStack을 제공하여 어떠한 화면을 보여줄 지 결정하게 된다.

> 추가로 Navigation의 전환 애니메이션 또한 지정해 줄 수 있다.

```kotlin
@Composable
@Preview
fun App(
    root: RootComponent
) {
    MaterialTheme {
        val childStack by root.childStack.subscribeAsState()
        Children(
            stack = childStack,
            animation = stackAnimation(slide())
        ) { child ->
            when (val instance = child.instance) {
                is RootComponent.Child.ItemList -> ItemListScreen(instance.component)
                is RootComponent.Child.ItemDetail-> ItemDetailScreen(instance.component)
            }
        }
    }
}
```

이제 만들어진 RootComponent를 App의 파라미터로 전달하여 안드로이드, Desktop에서 실행해 보자.

### MainActivity

```kotlin
class MainActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        val root = retainedComponent {
            RootComponent(it)
        }

        setContent {
            App(root = root)
        }
    }
}
```

### Desktop

```kotlin
fun main() {
    val lifecycle = LifecycleRegistry()
    val root = runOnUiThread {
        RootComponent(
            componentContext = DefaultComponentContext(lifecycle = lifecycle)
        )
    }
    application {
        val windowState = rememberWindowState()
        Window(
            onCloseRequest = ::exitApplication,
            state = windowState,
            title = "DecomposeExample",
        ) {
            LifecycleController(
                lifecycleRegistry = lifecycle,
                windowState = windowState,
                windowInfo = LocalWindowInfo.current
            )
            App(root)
        }   
    }
}

internal fun <T> runOnUiThread(block: () -> T): T {
    if (SwingUtilities.isEventDispatchThread()) {
        return block()
    }

    var error: Throwable? = null
    var result: T? = null

    SwingUtilities.invokeAndWait {
        try {
            result = block()
        } catch (e: Throwable) {
            error = e
        }
    }

    error?.also { throw it }

    @Suppress("UNCHECKED_CAST")
    return result as T
}
```

<img src="/assets/resources/decompose-example-desktop.gif">

---

## References

- [https://github.com/arkivanov/Decompose](https://github.com/arkivanov/Decompose)
- [https://arkivanov.github.io/Decompose/](https://arkivanov.github.io/Decompose/)