---
title: "안드로이드 내부 데이터 저장 SQLite"
date: 2023-03-24
update: 2023-03-24
tags:
  - Android
  - SQLite
series: "Android"
---
안드로이드 앱 내부에 데이터를 저장하게 되면 Jetpack 라이브러리에 있는 [Room](https://ppeper.github.io/android/room-database/)을 사용하게 된다. 

처음 안드로이드 개발을 할 시 Room에 대한 학습을 통해 SQLite를 내부적으로 사용을 하고 있다는 것을 알았지만 직접적으로 SQLite를 사용해보지 않아 이번에 SQLite를 직접 사용해 보려고 한다.

# SQLite

- SQLite를 사용하면 Local database 생성 할 수 있다.
- 관계형 데이터 베이스 구조를 따른다.
- 작은 규모의 안드로이드 앱에서 사용하기 적합한 데이터 베이스이다.
- 기존의 SQL문과 동일하여 __insert, delete, update, select__ 문을 사용할 수 있다.

## 클래스 및 메서드
<h4>SQLiteDatabase</h4>

- 데이터베이스를 다루는 작업(추가, 삽입, 삭제, 쿼리) 담당한다.

<h4>SQLiteOpenHelper</h4>

- 데이터베이스의 생성, 열기, 업그레이드 담당한다.

<h4>ContentValues</h4>

- 데이터 베이스에 자료 입력 할 때 사용하는 클래스

<h4>Cursor</h4>

- Cursor는 SQL을 실행하는 객체, 데이터는 테이블 형식
- Cursor에는 현재 가리키고 있는 로우를 나타내는 위치가 있음
- 처음 처서를 반환 받았을 때 커서의 위치는 -1번째 행을 가리킴
    - 주요 메서드
        - `moveToNext()`
            - 다음 행으로 이동하는 메서드. 이동 성공 여부에 따라 true/false 리턴
        - `moveToFirst()`
            - 첫 번째 행으로 위치를 움직여주는 메서드
        - `moveToLast()`
            - 마지막 행으로 위치를 움직여주는 메서드
        - `getColumnIndex(String heading)`
            - 컬럼 헤딩을 넘겨주면 특정 컬럼의 인텍스를 가져오는 메서드

<h3>DBHelper</h3>

- SQLiteOpenHelper를 상속 받고 테이블 생성 및 쿼리 관련 기능 정의
- MainActivity 에서 DBHelper를 이용해 테이블 구성 및 C/R/U/D 처리한다.
- 라이프 사이클 메서드
    - `onCreate` : 테이블 생성 등 초기 설정 처리
    - `onOpen` : 구동될 DB가 있다면 실제 사용 그 DB를 사용
    - `onUpgrade` : 만약 현재의 DB가 구 버전이라면 DB 업그레이드 처리(drop 후 create, alter table등으로 구현)
- _id integer은 SQLite를 사용하기 위해서 무조건 필수인 column이다. (primary key)
    - 없으면 column “_id”  does not exist라는 에러가 발생한다.

# 스키마 정의
안드로이드 공식문서에서는 Contract 클래스를 생성 후  `BaseColumns` 인터페이스를 구현체를 통하여 내부 클래스는 `_ID` 라는 기본 키 필드를 상속할 수 있도록 가이드 한다.

```kotlin
import android.provider.BaseColumns
import android.provider.BaseColumns._ID

object MemoContract {
    
    object MemoEntry : BaseColumns {
        private const val TABLE_NAME = "MEMO"
        private const val COLUMN_TITLE = "title"
        private const val COLUMN_CONTENT = "content"
        private const val COLUMN_DATE = "date"
        private const val CREATE_QUERY = """
            CREATE TABLE $TABLE_NAME (
                $_ID INTEGER PRIMARY KEY AUTOINCREMENT,
                $COLUMN_TITLE TEXT NOT NULL,
                $COLUMN_CONTENT TEXT NOT NULL,
                $COLUMN_DATE TEXT NOT NULL,
            )
        """
        private const val DROP_QUERY = "DROP TABLE if exists $TABLE_NAME"
    }
}
```
## DPHelper 생성
```kotlin 
class MemoDBHelper(
    context: Context,
) : SQLiteOpenHelper(context, DATABASE_NAME, null, DATABASE_VERSION) {
    private lateinit var db: SQLiteDatabase

    override fun onCreate(db: SQLiteDatabase) {
        db.execSQL(CREATE_QUERY)
    }

    override fun onUpgrade(db: SQLiteDatabase, oldVersion: Int, newVersion: Int) {
        db.execSQL(DROP_QUERY)
        onCreate(db)
    }

    // 데이터베이스가 writeable, readable을 호출할 때 open이 된다.
    override fun onOpen(db: SQLiteDatabase) {
        super.onOpen(db)
        this.db = db
    }

    //CRUD 메소드들

    companion object {
        private const val DATABASE_NAME = "memo.db"
        private const val DATABASE_VERSION = 1
    }

}
```

## Select
```kotlin
fun selectAllMemos(): ArrayList<MemoDto> {
    val memoList = ArrayList<MemoDto>()
    val projection = arrayOf(COLUMN_TITLE, COLUMN_CONTENT, COLUMN_DATE)
    val orderBy = "$COLUMN_DATE ASC"
    val cursor = db.query(
        TABLE_NAME, // 테이블 이름
        projection, // 반환할 컬럼들 -> null이면 *과 동일
        null, // WHERE문
        null, // WHERE문에 들어갈 values
        null, // groupBy
        null, // having
        orderBy, // orderBy
        null // limit
    )
    with(cursor) {
        while (moveToNext()) {
            memoList.add(
                MemoDto(
                    getInt(0),
                    getString(1),
                    getString(2),
                    getString(3)
                )
            )
        }
    }
    return memoList
}
```
데이터를 조회하는 방법은 `query` 메소드를 이용하면 쉽게 Cursor 구현체를 반환 받아 사용할 수 있다.

Parameter는 공식문서에 잘 나와 있어, 위의 순서대로 테이블 이름, 조회하려는 칼럼들을 배열로 정의하고 이외 selection, slectionArgs, groupBy, having, orderBy, limit 등으로 다양한 조건을 추가 할 수 있다.

```kotlin
fun selectAllMemos(): ArrayList<MemoDto> {
    val memoList = ArrayList<MemoDto>()
    db.rawQuery("select * from $TABLE_NAME", null).use { cursor ->
        with(cursor) {
            while (moveToNext()) {
                memoList.add(
                    MemoDto(
                        getInt(0),
                        getString(1),
                        getString(2),
                        getString(3)
                    )
                )
            }
        }
    }
    return memoList
}
```
다른 방법으로는 SQL문을 직접 사용하는 `rawQuery` 메소드를 사용하여 원하는 쿼리 결과를 얻을 수도 있다.

`use` 키워드는 코틀린 1.2에서 제공된 AutoCloseable 을 제공해 주는 extension 메서드로 try/catch/finally를 모두 포함하고 있다.

```kotlin
@InlineOnly
public inline fun <T : Closeable?, R> T.use(block: (T) -> R): R {
    contract {
        callsInPlace(block, InvocationKind.EXACTLY_ONCE)
    }
    var exception: Throwable? = null
    try {
        return block(this)
    } catch (e: Throwable) {
        exception = e
        throw e
    } finally {
        when {
            apiVersionIsAtLeast(1, 1, 0) -> this.closeFinally(exception)
            this == null -> {}
            exception == null -> close()
            else ->
                try {
                    close()
                } catch (closeException: Throwable) {
                    // cause.addSuppressed(closeException) // ignored here
                }
        }
    }
}
```

## Insert
데이터를 추가하기 위해서는 `insert` 메서드를 사용할 수 있다.

```kotlin
fun insertMemo(memo: MemoDto): Long {
    val contentValues = ContentValues()
        .apply {
            put(COLUMN_TITLE, memo.title)
            put(COLUMN_CONTENT, memo.content)
            put(COLUMN_DATE, memo.date)
        }
    return db.insert(TABLE_NAME, null, contentValues)
}
```

SQLiteDatabase 클래스 내에서는 다양한 메서드를 제공하여 추가할 데이터를 `ContentValue` 에 key-value의 형태로 저장할 수 있다.

두 번째 인자는 ContentValues에 어떤 값도 삽입하지
 않았을 때 실행할 작업을 알려주는 것이다.
 
위에서와 같이 `null`을 지정하면 값이 없을 때는 행을 삽입하지 않는다.

insert() 메서드는 새로 생성된 행의 ID를 반환하거나 데이터 삽입 시 데이터의 충돌과 같은 오류가 발생하면 -1을 반환한다.

## Update
데이터의 수정은 `update` 메서드를 통하여 수정하려는 행에 대한 selection과 selectionArgs를 지정하고 바꾸려는 contentValues를 생성하여 인자로 넘겨준다.

update 메서드는 데이터베이스에서 영향을 받은 행의 수가 반환된다.

```kotlin
fun updateMemo(memo: MemoDto): Int {
    val selection = "$_ID=?"
    val selectionArgs = arrayOf(memo.id.toString())
    val contentValues = ContentValues()
        .apply {
            put(COLUMN_TITLE, memo.title)
            put(COLUMN_CONTENT, memo.content)
            put(COLUMN_DATE, memo.date)
        }
    return db.update(TABLE_NAME,
        contentValues,
        selection,
        selectionArgs)
}
```

## Delete
데이터의 삭제는 `delete` 메서드를 통하여 update와 마찬가지로 where 조건절을 설정한 후 호출한다.

delete 메서드는 데이터베이스에서 삭제된 행의 수가 반환 된다.

```kotlin
fun deleteMemo(id: Int): Int {
    val selection = "$_ID=?"
    val selectionArgs = arrayOf(id.toString())
    return db.delete(TABLE_NAME, selection, selectionArgs)
}
```
## Activity에서 사용하기
```kotlin
class MainActivity : AppCompatActivity() {
    private val binding by lazy { ActivityMainBinding.inflate(layoutInflater) }
    // DB 선언
    private lateinit var memoDBHelper: MemoDBHelper
    private lateinit var database: SQLiteDatabase

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(binding.root)
        initDatabase()   
    }


    private fun initDatabase() {
        memoDBHelper = MemoDBHelper(this)
        // getWriteableDatabase/getReadbleDatabase를 호출 시 onOpen 메서드 실행
        // database = memoDBHelper.readableDatabase
        database = memoDBHelper.writableDatabase
    }

    override fun onDestroy() {
        memoDBHelper.close()
        super.onDestroy()
    }

}
```
데이터베이스가 닫혀 있을 때 `getWriteableDatabase` 및 `getReadbleDatabase` 메서드가 호출 시에는 리소스가 많이 사용되기 때문에 최대한 데이터베이스 연결을 하도록 하고, 사용이 끝나면 `onDestroy()` 에서 데이터베이스를 받는 것이 좋다.

---

# References
- [https://developer.android.com/training/data-storage/sqlite?hl=ko](https://developer.android.com/training/data-storage/sqlite?hl=ko)