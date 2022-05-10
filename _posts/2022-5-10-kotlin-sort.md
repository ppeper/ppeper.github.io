---
title: 코틀린 리스트 정렬하는 방법 (sort vs sortBy vs sortWith)
categories: Kotlin
tags: ['Kotlin', 'Sort']
header:
    teaser: /assets/teasers/kotlin-list-image.png
last_modified_at: 2022-5-10T00:00:00+09:00
---
<img src="https://user-images.githubusercontent.com/63226023/167560322-2806af0e-35ee-4323-be7d-f289948e24d5.png">

최근에 알고리즘의 풀이를 `자바언어`에서 `코틀린언어`로 바꾸며 진행해 나가면서 알고리즘 문제 풀이에서 많이 사용되는 __정렬을 하는 방법__ 에 대해서 알아가보려고 한다.

- - -

# 리스트 기본 정렬
코틀린에서 리스트의 정렬은 다음과 같이 크게 `sort`, `sortBy`, `sortWith`의 세가지 방법이 있다.

코틀린에서는 __Immutable__ 과 __Mutable__ 로 나뉜다. 여기서 Immutable(데이터의 변경이 불가능)에서는 함수의 끝에 __ed__ 로 끝난다.

> - Mutable(데이터의 변경이 가능): `sort()`, `reverse()`
>
> - Immutable(데이터의 변경 불가능): `sorted()`, `reversed()`

## sort(), sorted() / reverse(), reversed()

> `sort()`는 __데이터의 변경이 가능한__ 리스트에 사용되며, `sort()`는 __리스트의 원형 변경하지 않고__, __리스트를 정렬된(오름차순) 리스트로 변경__ 합니다. 

```kotlin
fun main() {
    val mutablelist = mutableListOf(7, 2, 1, 10, 12, 5, 4)
    mutablelist.sort()
    println("MutableList: $mutablelist")
}
```

> `sorted()`는 __데이터의 변경이 불가능__ 한 리스트에 사용되며, `sorted()`는 __리스트의 원형 변경하지 않고__, __정렬된(오름차순) 리스트를 생성하여 반환__ 합니다. 

```kotlin
fun main() {
    val list = listOf(7, 2, 1, 10, 12, 5, 4)
    val sortedlist = list.sorted()
    println("List: $list")
    println("Sortedlist: $sortedlist")
    //    List: [7, 2, 1, 10, 12, 5, 4]
    //    Sortedlist: [1, 2, 4, 5, 7, 10, 12]
}
```
`reverse()`, `reversed()` 또한 위와 같은 방식으로 리스트를 __역순으로 정렬__ 한다.

- `reverse()` : Mutable 리스트에 사용한다. __리스트를 역순으로 정렬을 시킨다__
- `reversed()` : Immutable 리스트에 사용한다. __역순으로 변경된 리스트를 생성하고 반환 한다__

- - -

# 기본 정렬 -> sort(ed)By
`sortBy()`는 리스트 요소가 1개 이상으로 있을때 어떤 요소를 기준으로 정렬을 할지 결정할 수 있다.

아래의 예시에서 __Info__ 클래스에 대해서 __이름순으로__ 정렬을 하고자 하면 `sortBy`를 사용하면 된다.

```kotlin
data class Info(
    val name: String,
    val count: Int,
    val grade: Double
)

fun main() {
    val list = listOf(
        Info("B", 8, 2.5),
        Info("C", 5, 3.0),
        Info("E", 1, 4.0),
        Info("A", 9, 3.5),
        Info("A", 9, 1.5),
        Info("A", 11, 1.5),
    )

    println("Name으로 정렬")
    list.sortedBy {
        it.name
    }
    //    Name으로 정렬
    //    Info(name=A, count=11, grade=3.5)
    //    Info(name=A, count=9, grade=3.5)
    //    Info(name=A, count=9, grade=1.5)
    //    Info(name=B, count=8, grade=2.5)
    //    Info(name=C, count=5, grade=3.0)
    //    Info(name=E, count=1, grade=4.0)

}
``` 
# 기준이 2개 이상 -> sort(ed)With
위의 예시에서는 Name으로 오름차순으로 정렬을 한것을 볼 수 있다. 여기서 새 프로퍼티 Count를 정렬 조건으로 추가하고 싶다면 `sortWith()`을 사용하면 된다. `sortWith()`는 인자로 Comparator를 구현하여 넘겨주게 된다.

```kotlin
fun main() {
    val list = listOf(
        Info("B", 8, 2.5),
        Info("C", 5, 3.0),
        Info("E", 1, 4.0),
        Info("A", 11, 3.5),
        Info("A", 9, 3.5),
        Info("A", 9, 1.5),
    )

    println("Name으로 정렬후 Count으로 정렬")
    list.sortedWith(
        compareBy<Info> {
            it.name
        }.thenBy {
            it.count
        }
    )
    //    Name으로 정렬후 Count으로 정렬
    //    Info(name=A, count=9, grade=3.5)
    //    Info(name=A, count=9, grade=1.5)
    //    Info(name=A, count=11, grade=3.5)
    //    Info(name=B, count=8, grade=2.5)
    //    Info(name=C, count=5, grade=3.0)
    //    Info(name=E, count=1, grade=4.0)
    println("Name으로 정렬후 Count으로 정렬후 Grade로 정렬")
    list.sortedWith(
        compareBy<Info> {
            it.name
        }.thenBy {
            it.count
        }.thenBy {
            it.grade
        }
    )
    //    Name으로 정렬후 Count으로 정렬후 Grade로 정렬
    //    Info(name=A, count=9, grade=1.5)
    //    Info(name=A, count=9, grade=3.5)
    //    Info(name=A, count=11, grade=3.5)
    //    Info(name=B, count=8, grade=2.5)
    //    Info(name=C, count=5, grade=3.0)
    //    Info(name=E, count=1, grade=4.0)
}
```

# 정리
> 정렬의 기본값은 __오름차순으로 정렬__ 이고 이는 `decending`을 통하여 __내림차순으로 정렬__ 이 가능하다.

- Immutable(데이터의 변경이 불가능)
    - *ed()를 사용: __정렬된 리스트를 생성하고 반환 한다.__
- Mutable(데이터의 변경이 가능)
    - *()를 사용: __현재 리스트를 정렬한다.__

> - __기본정렬__ : sort(오름차순), reverse(역순), *Decending(내림차순)
> - __sortBy, sortByDecending__ : 리스트 요소가 1개 이상일때 > 어떤 요소를 기준으로 정렬을 할지 결정할 수 있다.
>
>
> - __sortWith, sortWithDescending__ : 기준이 2개이상으로 정렬을 하려고자 할때 사용이 가능하다.
>     - `compareBy`, `compareByDescending`: Comparator를 사용하여 정렬기준을 정한다.
>     - `thenBy`, `thenByDecending`: compareBy에 이어서 정렬기준을 정할때 사용한다.