---
title: "안드로이드 RecyclerView의 DiffUtil 알아보기"
date: 2022-09-01
update: 2022-09-01
tags:
  - Android
  - DiffUtil
  - RecyclerView
series: "Android"
---
# DiffUtil 넌 뭐니
안드로이드를 공부하거나 개발하다보면 대부분 리스트를 보여주기 위하여 `RecyclerView` 의 사용을 하게되고, 리스트의 데이터가 변하게 되면 `notifyDataSetChange()` 를 호출하여 리사이클러뷰를 갱신하였다. 이는 바뀐 데이터가 적더라도 간혹 `notifyDataSetChange()`를 호출하여 갱신하기도 하는데 이는 __앱 성능에 굉장히 악영향__ 을 미치게된다. (난가..😅)

`notifyDataSetChange()` 를 호출하게 되면 __리스트의 모든 데이터를 다시 처음부터 새로운 객체를 생성하여 랜더링__ 하기 때문에 __비용이 크게 발생한다.__

이런 경우를 위해 등장한 것이 __DiffUtil__ 클래스이다. DiffUtil은 __이전 데이터와 현재 데이터 목록의 차이를 계산하여__ 업데이트 해야할 데이터에 대해서만 갱신을 할 수 있게 한다.

<img src="https://user-images.githubusercontent.com/63226023/187503548-029d1017-9e5d-4712-90c3-257e47f2e594.gif">

> 📃 두 데이터간의 차이를 계산은 Eugene W.Myers 의 Diff(erence) Algorithm이 사용되었다고 한다.

# 사용 방법
`DiffUtil` 는 __이전과 현재의 목록의 차이를 계산__ 을 한 뒤 `DiffUtil.Callback` 이라는 추상 클래스를 __콜랙 클래스로 활용__ 하게 된다. 이 클래스는 __4개의 추상 메소드와 1개의 일반 메소드__ 로 이루어져 있으며, 이를 확장하여 메소드를 오버라이딩 하여 사용한다.
> __getOldListSize()__
>
> 이전 목록의 크기를 반환한다.

> __getNewListSize()__
>
> 새로운 목록의 크기를 반환한다.

> __areItemsTheSame(int oldItemPosition, int newItemPosition)__
>
> 두 항목이 같은 객체인지 반환한다. 

> __areContentsTheSame(int oldItemPosition, int newItemPosition)__
>
> 두 항목의 데이터가 같은지 여부를 반환한다. `areItemsTheSame()`이 true를 반환할 때만 호출된다. -> 같은 객체가 아니면 당연히 데이터는 다르기 때문에 비교하는것이 의미가 없음

> __getChangePayload(int oldItemPosition, int newItemPosition)__
>
> 만약 `areItemTheSame()`이 true를 반환하고, `areContentsTheSame()`이 false를 반환했다면 __변경 내용에 대한 페이로드__ 를 가져온다.

먼저 `DiffUtil.Callback` 함수를 (익명)클래스를 구현한다.

```kotlin
class DiffUtilCallback(
    private val oldList: List<Any>,
    private val newList: List<Any>
): DiffUtil.Callback() {
    override fun getOldListSize() = oldList.size

    override fun getNewListSize() = newList.size

    
    // 객체의 고유 값을 비교하는게 좋다.
    override fun areItemsTheSame(oldItemPosition: Int, newItemPosition: Int): Boolean {
        val oldItem = oldList[oldItemPosition]
        val newItem = newList[newItemPosition]
        return oldItem.id == newItem.id
    }

    override fun areContentsTheSame(oldItemPosition: Int, newItemPosition: Int): Boolean {
        val oldItem = oldList[oldItemPosition]
        val newItem = newList[newItemPosition]
        return oldItem == newItem
    }
}
```

이렇게 만든 클래스를 생성한 후에는 해당 부분을 adpter내에서 아래와 같은 함수를 만들어 `DiffUtil.calculateDiff()` 에 해당 __콜백 클래스(DiffUtilCallback)__ 를 통하여 업데이트가 필요한 리스트를 찾는다.

이후 기존의 `notifyDataSetChange()` 대신에 `dispatchUpdatesTo(Adapter adapter)` 를 사용하면 __부분적으로 데이터를 교체하는 notify가 실행__ 된다.

```kotlin
// RecyclerView Adapter 내에서
private val list = mutableListOf<Any>()

...

private fun update(newList: List<Any>) {
    val diffUtil = DiffUtilCallback(list, newList)
    val diffResult: DiffUtil.DiffResult = DiffUtil.calculateDiff(diffUtil)
    list.clear()
    list.addAll(newList)
    diffResult.dispatchUpdatesTo(this)
}
```

- 리스트의 크기가 크다면 비교 연산이 길어질 수 있으므로 `calculateDiff()` 는 __백그라운드 쓰레드에서 처리__ 를 해주고, 메인 쓰레드에서 `DiffUtil.DiffResult` 를 가져와 사용하는 것이 권장된다.
- 구현상 목록의 최대 크기는 2²⁶개 이다.

- - -
# AsyncListDiffer
위에서 말한 `DiffUtil`을 백그라운드 쓰레드에서 수행할 수 있게 해주는 클래스이다. __`adapter`와 `DiffUtil`을 인자로 받아 백그라운드에서 수행 후 RecyclerView에 결과를 반영__ 할 수 있게 해준다. 

## 사용 방법
먼저 리스트의 요소를 비교할 때 호출할 `DiffUtil.ItemCallback`를 구현한다.

```kotlin
class AsyncDiffUtilCallback : DiffUtil.ItemCallback<Any>() {

    override fun areItemsTheSame(oldItem: Any, newItem: Place) {
        return oldItem.id == newItem.id
    }
        

    override fun areContentsTheSame(oldItem: Any, newItem: Any) {
        return oldItem == newItem
    }
}
```
마찬가지로 adapter 내부에서 `AsyncListDiffer` 객체를 선언하여 사용하면 된다.

> __getCurrentList()__
>
> adapter에서 사용하는 리스트에 접근하고 싶을때 사용한다.

> __submitList()__
>
> 리스트의 데이터를 교체할 때 사용한다.

```kotlin
class CustomAdapter(): RecyclerView.Adapter<CustomAdapter.CustomViewHolder>() {
    val differ = AsyncListDiffer(this, AsyncDiffUtilCallback())

    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): CustomViewHolder { ... }

    override fun onBindViewHolder(holder: CustomViewHolder, position: Int) {
        // getCurrentList()
        val any = differ.currentList[position]
        holder.bind(any)
    }

    override fun getItemCount() = differ.currentList.size
    
    // submitList()
    fun update(newItems: List<Any>) = diffUtil.submitList(newItems)
}
```
> ~~(AsyncListDiffer 클래스 최고다!!)~~ 아직 한발 남았다..

- - -
# ListAdapter✨
이러한 AsyncListDiffer을 더 편리하게 사용하도록 한 Wrapper 클래스가 `ListAdapter`이다.

`ListAdapter`는 DiffUtil을 활용하여 리스트를 업데이트하는 기능이 추가된 Adapter라고 생각하면 된다.

`ListAdapter` 의 인자로 __제너릭 타입 T에는 데이터의 타입, 두번째로 RecyclerView.ViewHolder__ 를 넣어준다.

```kotlin
public abstract class ListAdapter<T, VH extends RecyclerView.ViewHolder>
        extends RecyclerView.Adapter<VH> {
    final AsyncListDiffer<T> mDiffer;
    private final AsyncListDiffer.ListListener<T> mListener =
            new AsyncListDiffer.ListListener<T>() {
        @Override
        public void onCurrentListChanged(
                @NonNull List<T> previousList, @NonNull List<T> currentList) {
            ListAdapter.this.onCurrentListChanged(previousList, currentList);
        }
    };
```
`ListAdapter`는 AsyncListDiffer를 포함하는 클래스로, AsyncListDiffer의 객체 생성이 필요없이 __백그라운드 쓰레드에서 DiffUtil의 비교 연산__ 을 편하게 수행할 수 있다.

> __getItem(position: Int)__
>
> ListAdapter 내부의 리스트에 해당 position 데이터를 사용할때 활용한다.

> __getCurrentList()__
>
> ListAdapter에서 사용하는 리스트에 접근하고 싶을때 사용한다.

> __submitList()__
>
> 리스트의 데이터를 교체할 때 사용한다.

```kotlin
// 제너릭 T와 ViewHolder를 넣어준다
class CustomListAdapter(): ListAdapter<Any, CustomListAdapter.CustomViewHolder>(diffUtil) {
    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): CustomViewHolder { ... }

    override fun onBindViewHolder(holder: CustomViewHolder, position: Int) {
        // getItem()
        holder.bind(getItem(position))
    }

    // DiffUtil.ItemCallback
    companion object {
        val diffUtil = object: DiffUtil.ItemCallback<Any>() {
            override fun areItemsTheSame(oldItem: Any, newItem: Any): Boolean {
                return oldItem.id == newItem.id
            }

            override fun areContentsTheSame(oldItem: Any, newItem: Any): Boolean {
                return oldItem == newItem
            }
        }
    }
}
```
`ListAdapter`를 사용하면 자체에서 `submitList()`를 지원하므로 메소드를 노출시키지 않고 adapter에서 바로 호출할 수 있다.

```kotlin
// Activity & Fragment
adpater.submitList(newList)
```

- - -
# References
- [https://hungseong.tistory.com/24](https://hungseong.tistory.com/24)
- [https://blog.kmshack.kr/RecyclerView-DiffUtil로-성능-향상하기](https://blog.kmshack.kr/RecyclerView-DiffUtil%EB%A1%9C-%EC%84%B1%EB%8A%A5-%ED%96%A5%EC%83%81%ED%95%98%EA%B8%B0/)
