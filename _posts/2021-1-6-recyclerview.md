---
title: '안드로이드 RecyclerView 뿌수기'
categories: Android
tags: ['Android', 'RecyclerView']
header:
    teaser: /assets/teasers/recyclerView_teaser.png
last_modified_at: 2022-1-6T00:00:00+09:00
---
- - -

# RecyclerView
처음 안드로이드 공부를 하면서 친구목록, 채팅방의 기능이 필요할때 RecyclerView를 접하게 되었었다.

 안드로이드 어플을 만든다고하면 `리스트`로 보여주는 기능을 거의 필수적으로 사용된다고 본다!! 다시한번 포스팅을 통하여 안드로이드 개발시 거의 필수적으로 사용되는 `RecyclerView`에 대해서 정리를 하고 숙지하려고 한다. 

  > RecyclerView : 안드로이드 개발시 데이터를 동적으로 스크롤이 가능한 데이터 리스트의 형태로 사용자에게 제공한다. 

# Recycler??
<img src="https://user-images.githubusercontent.com/63226023/148042348-4b059f40-4154-4008-99ac-3d10504c5f93.png" width="50%"> -> Recycler..재활용??

RecyclerView가 `리스트`를 보여준다고 하였는데 이름에서는 재활용?이라는 이름이 붙어 있어서 그냥 ListView라고 이름을 지으면 안 헷갈리지 않을까?? 생각이 들었다.

당연하게 과거에도 이러한 리스트를 보여줄수 있는 `ListView`가 존재 하였었다. -> RecyclerView가 나온 이유가 있지 않을까?

## RecyclerView vs ListView
RecyclerView와 ListView의 차이점은 블로그를 참고하여 가져왔다.
![image](https://user-images.githubusercontent.com/63226023/148350892-f82acb13-beb0-46d6-8053-6f032ef9a5b0.png)

RecyclerView와 ListView의 큰 차이점은 ViewHolder 패턴을 강제로 하는가에 달려있다.

ListView에서는 일반적으로 `getView()`함수를 통하여 화면에 보여주었다. 이러한 구조의 문제점은 10개의 리스트를 보여주고 아래로 스크롤을 하게되면 고비용의 `findViewById()`의 사용으로 메모리와 성능에 안좋은 영향을 미칠수 있었다. -> ListView에서도 ViewHolder를 사용할 수 있지만 구현이 귀찮은 작업이 되었었다.
 > -> ViewHolder는 아래에서 설명하겠다. 

RecyclerView는 ViewHolder 패턴을 강제적으로 사용을 하게되어있어서 10개의 리스트에서 스크롤이되었을때 안보이게 된 (ex: 1,2번)을 11,12에 사용을 `재활용`하여 메모리나 성능에 악 영향을 미치는것을 줄여 주었다.
- ListView에서 ViewHolder패턴을 사용하면 RecyclerView와 성능에서는 똑같다고 한다.

- - -
## RecyclerView 구현
RecyclerView를 사용하기 위해서는 크게 3가지로 구성된다.
1. 레이아웃 매니저 설정
2. Adapter() 설정
3. ViewHolder() 패턴 설정

### 1. 레이아웃 매니저 설정
RecyclerView가 ListView와 또 다른점 중 하나는 RecyclerView는 `LayoutManager`을 사용하여 세로 스크롤 뿐만아니라 다른 방법으로 리스트 항목을 보여주는 방법이 존재한다.
1. LinearLayoutManager
 > 리스트 항목이 `수평` 또는 `수직`의 스크롤이 가능한 형태로 나타난다. 
2. GridLayoutManager
 > 리스트 항목이 `격자(grid)` 형태로 나타나게 된다. -> 리스트의 항목이 균일한 크기일 때는 이 레이아웃 매니저를 사용하는것이 가장 좋다!
3. StaggeredGridLayoutManager
 > 리스트의 항목이 `일정하지 않은 크기`의 격자 형태로 나타난다.

 - - -
### 2. Adapter() 설정
Adapter()는 사용자에게 보여줄 데이터와(ViewHolder) RecyclerView 인스턴스 간의 중개자 역할을 한다.

RecyclerView.Adapter 클래스의 서브 클래스로 생성이 되어 다음 3가지의 함수를 구현하야한다. 
- __getItemCount()__ - 이 함수에서는 리스트에 보여줄 항목의 개수를 반환한다.
- __onCreateViewHolder()__ - 이 함수는 데이터를 보여 주는 데 사용되는 View를 갖도록 초기화된다. 
    
  > 여기서 ViewHolder 객체가 생성되고 반환된다! 이때 해당 뷰는 xml 파일로 생성된 파일을 inflate하여 생성한다.
- __onBindViewHolder()__ - 이 함수에서는 두개의 인자를 받는다. 함수의 이름에서 알 수 있듯이 생성된 ViewHolder를 바인드해주어 사용자에게 보여 줄 수 있다.
   - `ViewHolder 객체`와 보여 줄 리스트의 항목의 `인덱스`를 나타내는 정수값을 받는다.

### 3. ViewHolder() 패턴 설정
이름에서도 알 수 있듯이 View를 보관(Hold)하는 객체이다.

- 여기서 왜 RecyclerView는 ViewHolder를 강제 하였는지 알 수 있다. ViewHolder를 통하여 뷰에 해당하는 xml 파일을 inflate을 할때 고 비용의 `findViewById()`를 사용하지않고 `가지고`(Hold) 있다가 재활용 할 수 있도록 한다!!

- - -

# 사용 예시
[다음은 RecyclerView + 툴바 파일이다]("https://github.com/ppeper/Android_Arctic-Fox/tree/main/CardDemo")
## 1. 레이아웃 설정
`card_layout.xml` - List를 보여줄 ViewHolder에 사용될 xml파일 이다.

  <img src="https://user-images.githubusercontent.com/63226023/148355805-70c15815-d24a-46b6-855f-852252973074.png" width="30%">

`content_main.xml` - RecyclerView를 추가하여 사용자에게 리스트로 보여준다. 
<img src="https://user-images.githubusercontent.com/63226023/148356179-7375e768-b62e-4472-b2db-7ab56bc8c35d.png" width="30%"> <img src="https://user-images.githubusercontent.com/63226023/148356551-522be9c4-62a0-4ecd-9173-be3a26c8fb41.png" width="30%"> 
  
  여기서 `tools:listitem="@layout/card_layout"` 를 사용하여 미리 내가 보여줄 뷰를 볼 수 있다.

## 2. RecyclerView 생성
`RecyclerAdapter.kt`를 설정해준다. 

여기서는 앞서 말했던 `getItemCount()`,`onCreateViewHolder()`,`onBindViewHolder()`를 필수적으로 구현한다.

```kotlin
class RecyclerAdapter : RecyclerView.Adapter<RecyclerAdapter.CustomViewHolder>() {

    private val titles = arrayOf("Chapter One", "Chapter Two", "Chapter Three", "Chapter Four",
        "Chapter Five", "Chapter Six", "Chapter Seven", "Chapter Eight")

    private val details = arrayOf("Item one details", "Item two details", "Item three details",
        "Item four details", "Item five details", "Item six details", "Item seven details", "Item eight details")

    private val images = intArrayOf(R.drawable.android_image_1, R.drawable.android_image_2, R.drawable.android_image_3,
        R.drawable.android_image_4, R.drawable.android_image_5, R.drawable.android_image_6, R.drawable.android_image_7,
        R.drawable.android_image_8)

    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): CustomViewHolder {
        val binding = CardLayoutBinding.inflate(LayoutInflater.from(parent.context), parent, false)
        return CustomViewHolder(binding)
    }

    // 매개변수 holder position -> 뷰에 데이터를 참조
    override fun onBindViewHolder(holder: CustomViewHolder, position: Int) {
        holder.bindTitle(titles[position])
        holder.bindDetail(details[position])
        holder.bindImage(images[position])

        holder.itemView.setOnClickListener {
            Snackbar.make(holder.itemView, "Click detected on item $position",
            Snackbar.LENGTH_SHORT).setAction("Action", null).show()
        }
    }
    override fun getItemCount() =  titles.size

    /*
    ViewHolder 클래스 -> card_view사용
     */
    inner class CustomViewHolder(private val binding: CardLayoutBinding) : RecyclerView.ViewHolder(binding.root) {
        fun bindTitle(s: String) {
            binding.itemTitle.text = s
        }

        fun bindDetail(s: String) {
            binding.itemDetail.text = s
        }

        fun bindImage(image: Int) {
            binding.itemImage.setImageResource(image)
        }

    }
}
```
### 1. getItemCount()
```kotlin
    override fun getItemCount() =  titles.size
```
리스트를 보여줄 때 data클래스를 통하여 하나의 `ArrayList`를 사용하여 item을 count하지만 책의 예시를 따라가겠다.

### 2. onCreateViewHolder()
```kotlin
    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): CustomViewHolder {
        val binding = CardLayoutBinding.inflate(LayoutInflater.from(parent.context), parent, false)
        return CustomViewHolder(binding)
    }
```
책과는 조금 다르게 여기서는 `viewBinding`을 사용하였다. 앞서 말했던 뷰에 해당하는 xml 파일을 `inflate`하여 ViewHolder가 보관하여 재활용 할 수 있도록 binding된 값을 리턴해준다.

### 3. onBindViewHolder()
```kotlin
 // 매개변수 holder position -> 뷰에 데이터를 참조
    override fun onBindViewHolder(holder: CustomViewHolder, position: Int) {
        holder.bindTitle(titles[position])
        holder.bindDetail(details[position])
        holder.bindImage(images[position])

        holder.itemView.setOnClickListener {
            Snackbar.make(holder.itemView, "Click detected on item $position",
            Snackbar.LENGTH_SHORT).setAction("Action", null).show()
        }
    }
```
ViewHolder에 적절한 이름을 찾아 해당 position에 연결해준다.
- - -
## 3. ViewHolder() 생성
여기서는 `CustomViewHolder`를 inner class로 생성하였다.
```kotlin
    /*
    ViewHolder 클래스 -> card_view사용
     */
    inner class CustomViewHolder(private val binding: CardLayoutBinding) : RecyclerView.ViewHolder(binding.root) {
        fun bindTitle(s: String) {
            binding.itemTitle.text = s
        }

        fun bindDetail(s: String) {
            binding.itemDetail.text = s
        }

        fun bindImage(image: Int) {
            binding.itemImage.setImageResource(image)
        }
    }
```
매개변수로 받은 binding으로 각 해당하는 데이터를 연결해준다.

-> Adapter와 ViewHolder를 만들었다면 이제 사용하면 된다
```kotlin
        // 1. 현재 recyclerview에 레이아웃 매니저 설정
        layoutManager = LinearLayoutManager(this)
        binding.contentMain.recyclerView.layoutManager = layoutManager
        // 2. 어댑터 생성 -> 현재 recyclerview에 어뎁터 설정
        adapter = RecyclerAdapter()
        binding.contentMain.recyclerView.adapter = adapter
```

# References
- [안드로이드 RecyclerView]("https://developer.android.com/guide/topics/ui/layout/recyclerview?hl=ko")
- [[안드로이드] ListView vs RecyclerView]("https://woovictory.github.io/2019/01/03/Android-Diff-of-ListView-and-RecyclerView/")
- [[안드로이드] RecyclerView ( ListView와 차이 )]("https://armful-log.tistory.com/27")
