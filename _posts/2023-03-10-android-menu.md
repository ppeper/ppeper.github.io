---
title: 안드로이드 다양한 Menu들 사용하기
categories: Android
tags: ['Menu', 'Android']
header:
    teaser: /assets/teasers/android-menu-image.png
last_modified_at: 2023-03-10T00:00:00+09:00
---

# 안드로이드 Menu
안드로이드에서는 다양한 Menu들을 제공하여 편하게 옵션들을 보여줄 수 있다. 최근에는 사용자 경험이 바뀌어 Menu들을 통하여 옵션 세트들을 보여주는 방식이 변경이 되었지만 기본이 되는 Menu들의 각 차이점과 사용법을 알아 보려고 한다.

안드로이드에서 Menu는 크게 `Option`, `Context`, `Popup`로 구현할 수 있다.

<h3>Options Menu</h3>
화면 상단의 툴바의 흔히 점 3개로 볼 수 있는 버튼을 누르면 나타나는 Menu이다.
<h3>Context Menu</h3>
View를 길게 눌렀을 때 나타나는 Floating Menu이다.
<h3>Popup Menu</h3>
View를 눌렀을 때 나타나는 Floating Menu이다.

- - -
# Menu
안드로이드에서 Menu를 XML 파일에서 정의하고 Acitivty나 Fragment에서 Menu 리소스를 객체로 로드하여 사용할 수 있다.

Menu를 정의하기 위해서는 프로젝트의 `res/menu` directory에서 XML파일을 생성하고 해당 Menu에서 사용하는 요소들은 다음과 같다.

> - __&lt;menu&gt;__
>    - &lt;item&gt;, &lt;group&gt;을 하위 요소로 가지는 메뉴 항목의 컨테이너이다.
> - __&lt;item&gt;__
>    - Menu 내 단일 요소를 나타내며 속성들을 지정하여 여러가지 설정이 가능하다.
>    - Attribute
>        - id : item의 고유의 id값으로 요소를 구분할 때 사용
>        - showAsAction : v3.x 이상을 사용하며, 메뉴를 ActionBar나 ToolBar 에 보여줄지 여부를 지정 (ifRoom, always, never, withText)
>        - menuCategory : menu category를 정의하는 데 사용
>        - title : menu에 보여줄 이름 문자열
>        - titleCondensed : 이름 문자열이 너무 길었을 때 사용하는 문자열
>        - icon : drawable icon
> - __&lt;group&gt;__
>    - &lt;item&gt;을 분류하기 위한 투명한 컨테이너이다. (선택사항) group을 통하여 분류된 item들의 가시성 등 여러 속성을 공유할 수 있다.

메뉴파일을 생성하기 위해서는 `res` -> `new` -> `Android Resource File` 로 생성 할 수 있다.

<img src="https://user-images.githubusercontent.com/63226023/224061580-725fce7a-401b-48f0-a1a5-a5b3de5b6a92.png" width="60%">

menu xml에서 item 태그 안에 menu를 중첩하여 __sub menu__ 또한 구성이 가능하다.

```xml
<?xml version="1.0" encoding="utf-8"?>
<menu xmlns:android="http://schemas.android.com/apk/res/android">
    <item
        android:id="@+id/example_item"
        android:title="안녕 메뉴아이템" />
    <group android:id="@+id/example_group">
        <item
            android:id="@+id/example_item2"
            android:title="그룹 내부 아이템 1" />
        <item
            android:id="@+id/example_item3"
            android:title="그룹 내부 아이템 2" />
    </group>
    <item android:id="@+id/example_submenu" android:title="상위 메뉴">
        <menu>
            <item android:id="@+id/item_exit" android:title="종료" />
        </menu>
    </item>
</menu>
```
- - -

## Option Menu
Option Menu는 __onCreateOptionsMenu()__ 와 __onCreateOptionsMenu()__ 콜백을 통하여 등록할 수 있다.

```kotlin
class MainActivity : AppCompatActivity() {
    private val binding by lazy { ActivityMainBinding.inflate(layoutInflater) }

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(binding.root)
    }

    // Menu xml을 생성 한다.
    override fun onCreateOptionsMenu(menu: Menu?): Boolean {
        val inflater = menuInflater
        inflater.inflate(R.menu.optionmenu, menu)
        return super.onCreateOptionsMenu(menu)
    }

    // 각 요소는 고유의 등록한 id값으로 체크하여 사용한다.
    override fun onOptionsItemSelected(item: MenuItem): Boolean {
        if (item.itemId == R.id.item_exit) {
            finish()
        } else {
            Toast.makeText(this, "Hello Menu, ${item.title}", Toast.LENGTH_SHORT).show()
        }
        return super.onOptionsItemSelected(item)
    }
}
```

<img src="https://user-images.githubusercontent.com/63226023/224072847-8c19d520-34d5-43ab-90c4-48742ead5bf1.png" width="30%"> <img src="https://user-images.githubusercontent.com/63226023/224072860-f88e87c2-a918-4365-9e7e-9ca6e13b3609.png" width="30%">

## Context Menu
Context Menu는 버튼과 같은 특정 뷰를 사용자가 길게 눌렀을 때 활성화 된다. 

Context Menu는 Option Menu와 유사하게 __onCreateContextMenu()__ 와 __onContextItemSelect()__ 콜백을 등록하지만 __registerForContextMenu(getListView())__ 를 통하여 Context Menu와 연결해야하는 `View` 를 등록한다.

```kotlin
class MainActivity : AppCompatActivity() {
    private val binding by lazy { ActivityMainBinding.inflate(layoutInflater) }

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(binding.root)
        // Context Menu를 View에 등록한다.
        registerForContextMenu(binding.contextMenuBtn)
    }

    // Menu xml을 생성 한다.
    override fun onCreateContextMenu(menu: ContextMenu?, v: View?, menuInfo: ContextMenu.ContextMenuInfo?) {
        super.onCreateContextMenu(menu, v, menuInfo)
        
        menuInflater.inflate(R.menu.contextmenu, menu)
    }

    override fun onContextItemSelected(item: MenuItem): Boolean = with(binding) {
        when(item.itemId){
            R.id.context_menu_blue -> contextMenuBtn.setTextColor(Color.BLUE)
            R.id.context_menu_red -> contextMenuBtn.setTextColor(Color.RED)
            R.id.context_menu_green -> contextMenuBtn.setTextColor(Color.GREEN)
        }
        return super.onContextItemSelected(item)
    }
}
```
<img src="https://user-images.githubusercontent.com/63226023/224073041-5a238c30-4638-4f08-8718-dc13aee54727.png" width="30%"> <img src="https://user-images.githubusercontent.com/63226023/224073053-8c7710df-ffbe-448d-bbe1-f51a96e96091.png" width="30%">

## Popup Menu
Popup Menu는 특정 View를 눌렀을 때 활성화 된다.

Option, Context와는 다르게 `PopupMenu` 를 인스턴스화 하여 View에 생성하여 사용한다. PopupMenu를 보여주기 위해서는 `show()` 를 호출해야 한다.

PopupMenu에서는 __PopupMenu.OnMenuItemClickListener__ 인터페이스를 구현한 후 `setOnMenuItemclickListener()` 를 호출하여 PopupMenu에 등록하여 사용자가 항목을 누르면 __onItemClick()__ 콜백을 호출하도록 한다.

```kotlin
// Implement OnMenuItemClickListener
class MainActivity : AppCompatActivity(), OnMenuItemClickListener {
    private val binding by lazy { ActivityMainBinding.inflate(layoutInflater) }

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(binding.root)

        with(binding) {
            // Popup Menu 생성
           PopupMenu(this@MainActivity, binding.popupMenuBtn).apply { 
                inflate(R.menu.popupmenu)
                setOnMenuItemClickListener(this@MainActivity)
                show()
            }
        }
    }

    override fun onMenuItemClick(menuItem: MenuItem): Boolean {
        when (menuItem.itemId) {
            R.id.popup_menu1 -> {
                Toast.makeText(this@MainActivity, "메뉴 1 클릭", Toast.LENGTH_SHORT).show()
            }
            R.id.popup_menu2 -> {
                Toast.makeText(this@MainActivity, "메뉴 2 클릭", Toast.LENGTH_SHORT).show()
            }
            else -> {
                Toast.makeText(this@MainActivity, "메뉴 3 클릭", Toast.LENGTH_SHORT).show()
            }
        }
        return false
    }
}
```

두 번째 방법으로는 Popup Menu를 생성하하고 `setOnMenuItemClickListener()` 를 호출하여 사용할 수 있다.
```kotlin
class MainActivity : AppCompatActivity() {
    private val binding by lazy { ActivityMainBinding.inflate(layoutInflater) }

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(binding.root)

        with(binding) {
            popupMenuBtn.setOnClickListener {
                val popupMenu = PopupMenu(this@MainActivity, it)
                menuInflater.inflate(R.menu.pupupmenu, popupMenu.menu)
                // 콜백 메소드 구현
                popupMenu.setOnMenuItemClickListener { menuItem ->
                    when (menuItem.itemId) {
                        R.id.popup_menu1 -> {
                            Toast.makeText(this@MainActivity, "메뉴 1 클릭", Toast.LENGTH_SHORT).show()
                        }
                        R.id.popup_menu2 -> {
                            Toast.makeText(this@MainActivity, "메뉴 2 클릭", Toast.LENGTH_SHORT).show()
                        }
                        else -> {
                            Toast.makeText(this@MainActivity, "메뉴 3 클릭", Toast.LENGTH_SHORT).show()
                        }
                    }
                    false
                }
                // show() 를 호출하여 Popup Menu를 보여준다.
                popupMenu.show();
            }
        }
    }
}
```

<img src="https://user-images.githubusercontent.com/63226023/224073190-1c82e3d2-a0c0-48de-8628-4441b8a720f2.png" width="30%">

---
# References
- [https://developer.android.com/guide/topics/ui/menus?hl=ko#groups](https://developer.android.com/guide/topics/ui/menus?hl=ko#groups)




