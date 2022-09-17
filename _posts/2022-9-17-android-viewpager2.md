---
title: 안드로이드 ViewPager2 사용기
categories: Android
tags: ['Android', 'ViewPager2']
header:
    teaser: /assets/teasers/android-viewpager2-image.png
last_modified_at: 2022-9-17T00:00:00+09:00
---
# 🚀 ViewPager2
안드로이드 어플을 개발을 하다보면 RecyclerView를 통하여 데이터 목록을 보여주는 경우가 많다. 안드로이드에서 데이터의 목록이 아니라 Pager 형식의 ViewHolder를 구현하려면 `ViewPager2`를 사용을 해야한다.

## ViewPager와 ViewPager2
ViewPager2는 AndroidX가 발표된 이후에 새롭게 나온 ViewPager로 구글 공식문서에서 아래와 같은 기능을 지원하여 적극 사용을 권장하고 있다.
- ViewPager2는 세로방향의 페이징을 제공하고 있다. (orientation의 속성)
- RTL(Right To Left) 페이징을 지원한다.
    - RTL 페이징의 경우 설정된 언어에 따라 자동으로 사용 설정되지만 `layoutDirection`의 속성으로 수동으로 설정할 수 있다.
- 수정 가능한 Framgent Collection을 통해 페이징을 지원하며 `notifyDataSetChanged()`를 호출하여 UI를 업데이트 한다.
    - 📌 앱은 런타임 시 Fragment Collection을 동적으로 수정할 수 있다.

__ViewPager2__ 는 RecyclerView를 기반으로 만들어진 컴포넌트로 `RecyclerView.Adapter`와 `FragmentStateAdapter` 두 가지 방법으로 구현이 가능하다.
- - -
# 사용하기 (RecyclerView.Adapter)
## build.gradle 설정
`ViewPager2`를 사용하기 위해서 `build.gradle`에 종속성을 추가한다.

```gradle
dependencies {
    implementation "androidx.viewpager2:viewpager2:1.0.0"
}
```

## ViewPager2 layout 설정

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    tools:context=".MainActivity">

    <androidx.viewpager2.widget.ViewPager2
        android:id="@+id/viewPager1"
        android:layout_width="match_parent"
        android:layout_height="0dp"
        android:layout_weight="0.5"
        android:orientation="horizontal" />

</LinearLayout>
```
layout file에서 위와 같이 `android:orientation="horizontal"`로 설정이 가능하고.

Programmically로도 설정할 수 있다.
`viewPager2.orientation = ViewPager2.ORIENTATION_HORIZONTAL`

## Adapter 설정
간단하게 페이지의 title number를 보여주는 ViewPager Adapter를 만들어 준다.

```kotlin
data class Data(
    var title: String
)

class ViewPager2Adapter(
    private val list: ArrayList<Data>
): RecyclerView.Adapter<ViewPager2Adapter.CustomViewHolder>() {

    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): CustomViewHolder {
        val binding = ItemListBinding.inflate(LayoutInflater.from(parent.context), parent, false)
        return CustomViewHolder(binding)
    }

    override fun onBindViewHolder(holder: CustomViewHolder, position: Int) {
        holder.bind(list[position])
    }

    override fun getItemCount() = list.size

    inner class CustomViewHolder(private val binding: ItemListBinding) :
        RecyclerView.ViewHolder(binding.root) {

        fun bind(data: Data) {
            with(binding) {
                textView.text = data.title
            }
        }
    }
}
```

## Activity에서 사용
사용할 Activity에서 `adpater`를 붙여주어 사용한다.
```kotlin
class MainActivity : AppCompatActivity() {
    private lateinit var binding: ActivityMainBinding

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        binding = ActivityMainBinding.inflate(layoutInflater)
        setContentView(binding.root)

        val list: ArrayList<Data> = ArrayList()
        list.apply {
            add(Data("First"))
            add(Data("Second"))
            add(Data("Third"))
        }
        binding.viewPager1.adapter = ViewPager2Adapter(list)
        // binding.viewPager1.orientation = ViewPager2.ORIENTATION_HORIZONTAL
    }
}
```
## Tablayout 사용하기
layout 파일에 `TabLayout`을 추가한다.
```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    tools:context=".MainActivity">

    <com.google.android.material.tabs.TabLayout
        android:id="@+id/tabLayout1"
        android:layout_width="match_parent"
        android:layout_height="wrap_content" />

    ...

</LinearLayout>
```

`TabLayout` 과 `ViewPager` 간의 연결은 __TabLayoutMediator__ 를 구현한다. 

__TabLayoutMediator__ 에는 TabLayout과 ViewPager2 객체를 전달하고 `attach()` 함수를 호출하여 연동한다.

구현된 인터페이스는 Tab을 눌렀을때 해당하는 position마다 tab title text를 설정해 줄 수 있다.

```kotlin
class MainActivity : AppCompatActivity() {
        .
        .
        .
        // TabLayout 추가
        TabLayoutMediator(binding.tabLayout1, binding.viewPager1) { tab, position ->
            tab.text = "Tab ${position + 1}"
        }.attach()
    }
}
```
- - -
# 사용하기 (FragmentStateAdapter)
Fragment를 보여주도록 하는 `Adapter`가 바로 __FragmentAdapter__ 이다. 이는 RecyclerView의 `Adapter`를 만들때 RecyclerView.Adapter를 상속 받아 만들었을때와 같은 `Adapter`이다.

__FragmentStateAdapter__ 는 RecyclerView에서 Adapter에 ViewHolder를 지정하는 것과 마찬가지로, `FragmentViewHolder` 가 지정되어 구현 된다.

__FragmentStateAdapter__ 를 구현하기 위해서는 `getItemCount()`와 `createFragment()`를 반드시 구현해야 한다.

> 🔔 구글 공식 문서에서 __FragmentStateAdapter()__ 는 3종류의 생성자가 있다

```java
    public FragmentStateAdapter(@NonNull FragmentActivity fragmentActivity) {
        this(fragmentActivity.getSupportFragmentManager(), fragmentActivity.getLifecycle());
    }

    /**
     * @param fragment if the {@link ViewPager2} lives directly in a {@link Fragment} subclass.
     *
     * @see FragmentStateAdapter#FragmentStateAdapter(FragmentActivity)
     * @see FragmentStateAdapter#FragmentStateAdapter(FragmentManager, Lifecycle)
     */
    public FragmentStateAdapter(@NonNull Fragment fragment) {
        this(fragment.getChildFragmentManager(), fragment.getLifecycle());
    }

    /**
     * @param fragmentManager of {@link ViewPager2}'s host
     * @param lifecycle of {@link ViewPager2}'s host
     *
     * @see FragmentStateAdapter#FragmentStateAdapter(FragmentActivity)
     * @see FragmentStateAdapter#FragmentStateAdapter(Fragment)
     */
    public FragmentStateAdapter(@NonNull FragmentManager fragmentManager,
            @NonNull Lifecycle lifecycle) {
        mFragmentManager = fragmentManager;
        mLifecycle = lifecycle;
        super.setHasStableIds(true);
    }
```

> __1. FragmentStateAdapter(FragmentActivity fragmentActivity)__
> - 주로 ViewPager2가 Activity내에서 동작할 때 사용된다.

> __2. FragmentStateAdapter(Fragment fragment)__
> - Fragment 내에서 동작하는 ViewPager2를 만들 때 사용한다.

> __3. FragmentStateAdapter(FragmentManager fragmentManager, Lifecycle lifecycle)__
> - `fragmentManager`와 `lifecycle`이 호출되는 것으로 보아 1, 2번의 생성자 함수에서 호출되는 함수이다.

예시에서는 CustomFragment를 두개 생성하여 FragmentActivity로 받아 사용하였다.

__activity_main.xml__
```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    tools:context=".MainActivity">

    ...

    <com.google.android.material.tabs.TabLayout
        android:id="@+id/tabLayout2"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"/>

    <androidx.viewpager2.widget.ViewPager2
        android:id="@+id/viewPager2"
        android:layout_width="match_parent"
        android:layout_height="0dp"
        android:layout_weight="0.5"
        android:orientation="horizontal" />

</LinearLayout>
```
```kotlin
class ViewPagerAdapter(fa: FragmentActivity): FragmentStateAdapter(fa){
    override fun getItemCount() = 2
    override fun createFragment(position: Int): Fragment {
        return when (position) {
            0 -> CustomFragment1()
            1 -> CustomFragment2()
            else -> throw IndexOutOfBoundsException()
        }
    }
}
```


## Fragment 생성

Tab을 눌렀을때 Fragment를 통하여 보여주도록 CustomFragment(1,2)를 생성해 준다.

```kotlin
class CustomFragment1 : Fragment() {
    private lateinit var binding: FragmentCustomBinding

    override fun onCreateView(
        inflater: LayoutInflater, container: ViewGroup?,
        savedInstanceState: Bundle?
    ): View {
        binding = FragmentCustomBinding.inflate(inflater, container, false)
        return binding.root
    }
}
```

## Activity에서 사용
`Acitivty` <- `FragmentActivity` <- `AppCompatActivity`로 MainActivity에서 __ViewPagerAdapter(this)__ 로 adapter를 설정할 수 있다.

```kotlin
class MainActivity : AppCompatActivity() {
    private lateinit var binding: ActivityMainBinding

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        binding = ActivityMainBinding.inflate(layoutInflater)
        setContentView(binding.root)

        ...

        binding.viewPager2.adapter = ViewPagerAdapter(this)
        // TabLayout 추가
        TabLayoutMediator(binding.tabLayout2, binding.viewPager2) { tab, position ->
            tab.text = "Fragment ${position + 1}"
        }.attach()
    }
}
```
<p align="center">
<img src="https://user-images.githubusercontent.com/63226023/190859015-dd54e6ba-8cc2-4082-9252-896f9e58e987.gif" width="30%">
</p>
- - -
# References
- [https://developer.android.com/training/animation/screen-slide-2?hl=ko](https://developer.android.com/training/animation/screen-slide-2?hl=ko)