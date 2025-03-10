---
title: "안드로이드 Room 사용하면서 알아보기"
date: 2022-03-04
update: 2022-03-04
tags:
  - Android
  - Room
series: "Android"
---
<img src="https://user-images.githubusercontent.com/63226023/151594101-266890b7-079a-47c5-9daa-2afbc335ccb7.png">

# Room
Room은 안드로이드 __앱 내부에서 데이터를 저장__ 하기 위한 jetpack 라이브러리이다. Room은 데이터베이스의 데이터(객체)를 자바나 코틀린 객체로 매핑해주는 __ORM 라이브러리__ 이다.

> 📍ORM(Object Relational Mapping) : 객체-관계 매핑의 줄인말로 __객체와 관계형 데이터베이스의 데이터__ 를 __자동으로 매핑(연결)__ 해주는 것을 말한다.

 1. 객체 지향 프로그래밍에서는 __클래스__ 를 사용하고, 관계형 데이터베이스는 __테이블__ 을 사용하기 때문에 둘간의 __불일치__ 가 발생한다.
 2. ORM을 통하여 객체의 관계를 자동으로 SQL문을 생성을 해주어 이러한 불일치 문제를 해결해준다.

 Room 라이브러리가 나오기전에는 SQLite을 사용하여 데이터를 저장하였다. SQLite를 사용하여 DB를 사용하기 위해선 여러가지 많은 작업들을 해주어야 한다.
 > 🤔 [SQLite를 사용하여 데이터 저장](https://developer.android.com/training/data-storage/sqlite?hl=ko)
 
 Room도 내부적으론 SQLite를 사용하지만 이를 좀더 편하게 사용할 수 있도록 제공하며 다양한 장점이 있다.

 - 컴파일 동안 SQL 쿼리의 유효성 검사 가능.
 - 오류가 발생하기 쉬운 상용구 코드없이 OBM 라이브러리를 통하여 매핑 가능.
 - 스키마 변경시 자동으로 업데이트가 가능.
 - LiveData, RxJava와 같이 사용 가능.

- - -

# Room의 구조 & Annotation
<p align="center"><img src="https://user-images.githubusercontent.com/63226023/156314796-580f6154-d163-4642-912a-df39d48aa217.png" width="70%"></p>

Room 라이브러리의 구성요소는 Entity(데이터 모델), DAO(데이터 접근 객체), Database(데이터베이스 소유 객체)로 되어있다.

## Entity
데이터베이스에서 Table역할을 한다. class의 변수들의 column이 된다.

- __@Entity(tableName = 원하는 테이블 이름)__
    - Table의 이름을 설정해준다.(기본적으론 Entity class 이름을 Database table의 이름으로 인식한다) 

- __@PrimaryKey__
    - 데이터베이스 의 기본키를 지정해준다.(모든 Entity는 __하나의 Primary Key__ 를 가지고 있어야한다)
    - __(autoGenerate=true)__ 로 설정해주면 자동으로 Key값을 생성한다.

- __@ColumnInfo__
    - Table 내의 column을 변수들과 매칭한다.(기본적으론 Entity의 필드 이름을 열이름으로 인식한다)

## DAO
데이터베이스에 접근하여 상호작용하기 위한 작업들을 메서드 형태로 정의한다.

- __@Insert__
    - 파라미터로 넘겨받은 데이터를 테이블에 저장한다.
    - __onConflict = OnConflictStrategy.REPLACE__ 를 사용하면 Update와 동일한 기능으로 사용할 수 있다. IGNORE을 사용하면 데이터 중복시 기존 데이터를 유지한다.

- __@Update__
    - 데이터베이스의 값을 업데이트한다. 리턴 값으로 업데이트된 행 수를 받을 수 있다.

- __@Delete__
    - 데이터베이스의 값을 삭제한다. 리턴 값으로 삭제된 행 수를 받을 수 있다.

- __@Query__
    - SQL 쿼리문을 사용하여 데이터베이스에서 읽기/쓰기 작업을 실행할 수 있다. 쿼리에 매개변수를 전달하려면 __:컬럼 이름__ 을 사용해야한다.
    - Room은 컴파일 시간에 SQL 쿼리를 검증한다. 즉, 쿼리에 문제가 있으면 런타임 실패가 아닌 컴파일 오류가 발생한다.

## Room Database
데이터베이스의 소유자 역할로 __DB의 생성 및 버전을 관리__ 한다.

Room Database에서 DAO의 객체를 가져와 데이터를 "CRUD" 한다.

- __@Database__
    - class가 데이터베이스임을 알린다.
        - __entities__ : DB에 어떤 테이블들이 있는지 정의한다.
        - __version__ : DB의 버전을 정의한다. 스키마가 바뀌게 되면(Entity가 바뀐다면) version을 바꾸거나 Migration을 해야한다.
        - __exportSchema__ : Room의 Schema 구조를 폴더로 Export 할 수 있다. DB 버전 히스토리를 기록할 수 있다는 점에서 __true로 설정하는 것이 좋다__.

- - -

# 사용법
## 1. gradle 설정
가장 먼저 room을 사용하기 위해서는 gradle을 설정해주어야 한다.

```gradle
plugins {
        .
        .
    id 'kotlin-kapt'
}
        .
        .
def roomVersion = "2.4.1"
implementation "androidx.room:room-runtime:$roomVersion"
kapt "androidx.room:room-compiler:$roomVersion"
```

📍Room 라이브러리와 Coroutine을 함께 사용하기 위해서는 아래를 추가해줘야 한다.
```gradle
implementation "androidx.room:room-ktx:$roomVersion"
```

## 2. Entity 생성
```kotlin
@Entity(tableName = "myEntity")
data class Subscriber(

    @PrimaryKey(autoGenerate = true) // 자동으로 기본키를 생성
    @ColumnInfo(name = "myEntity_id")
    var id: Int,

    @ColumnInfo(name = "myEntity_name")
    var name: String,

    @ColumnInfo(name = "myEntity_email")
    var email: String

)

``` 
위의 예시에선 Entity의 테이블의 이름을 myEntity로 생성하고 Column의 이름을 사용자에 맞게 생성하였다.

## 3. DAO 생성
데이터베이스에 접근하여 사용할 메소드들을 생성해준다.

```kotlin
@Dao
interface SubscriberDAO {
    @Insert
    suspend fun insertSubscriber(subscriber: Subscriber)

    @Update
    suspend fun updateSubscriber(subscriber: Subscriber)

    @Delete
    suspend fun deleteSubscriber(subscriber: Subscriber)

    @Query("DELETE FROM subscriber_data_table")
    suspend fun deleteAll()
}
```
### 코루틴 suspend 

데이터베이스 접근이나 네트워킹과 같은 __메인 쓰레드를 Block 시킬 위험이 있는 작업__ 은 __백그라운드 쓰레드__ 로 스위칭을 해주어야 한다.

코루틴의 `suspend` 키워드를 함수 앞에 붙임으로서 __suspend를 호출한 쓰레드는 일시 suspend(정지)__ 가 되고, 해당 suspend가 붙은 함수의 실행이 끝나고 다시 Resume(재개)된다. 

Room, Retrofit과 같은 라이브러리는 `Main-safety` 가 보장되어 있어 메인 쓰레드에서 동작하여도 되어 `suspend` 를 사용하여 메인 쓰레드에서 동작하도록 하면 된다.

## 4. Database 생성
데이터베이스의 생성은 @Database를 통하여 이 클래스가 데이터베이스임을 명시해주고 __RoomDatabase()를 상속하여 abstract로 생성__ 한다. 또한 Dao를 데이터베이스에서 사용하기 위하여 __abstract dao를 가져야한다.__

Room 객체는 많은 리소스를 소모하며, 단일 프로세스 내에선 여러 인스턴스에 접근할 필요가 거의 없기 때문에 __Singleton__ 패턴을 사용하여 생성해 준다.


```kotlin
@Database(entities = [Subscriber::class], version = 1)
abstract class SubscriberDatabase: RoomDatabase() {

    abstract val subscriberDAO : SubscriberDAO

    companion object {
        @Volatile
        private var INSTANCE : SubscriberDatabase? = null

        fun getInstance(context: Context): SubscriberDatabase {
            synchronized(this) {
                var instance = INSTANCE
                if (instance == null) {
                    instance = Room.databaseBuilder(
                        context.applicationContext,
                        SubscriberDatabase::class.java,
                        "subscriber_data_database"
                    ).build()
                }
            return instance
            }
        }
    }
}
```
synchronized 블록을 통하여 싱글톤 인스턴스가 중복되는것을 방지해준다. 또한 [@Volatile 어노테이션](https://nesoy.github.io/articles/2018-06/Java-volatile)을 접근가능한 변수의 값을 cache를 통해 사용하지 않고 쓰레드가 직접 main memory에 접근 하게하여 동기화한다. 

## 5. 생성된 DB 사용
데이터베이스까지 완료되었다면 getInstance()함수를 통하여 사용할 위치에서 생성하여 사용한다.
```kotlin
val db = SubscriberDatabase.getInstance(application)
```

# 마무리
이번 Room 라이브러리를 다시 정리해보며 코루틴의 `suspend`사용과 더불어 몰랐던 많은 새로운 내용들을 알게되었다🤔

최근에 MVVM 패턴에 대한 내용을 접하면서 Room 라이브러리는 Repository와 같이 사용하는 것을 알게되었고, 정리한 내용들을 이러한 패턴과 적용시켜서 만들어 봐야겠다고 생각이 들었다.
- - -

# References

- [Room을 사용하여 로컬 데이터베이스에 데이터 저장](https://developer.android.com/training/data-storage/room?hl=ko)
- [[DB] ORM이란](https://gmlwjd9405.github.io/2019/02/01/orm.html)
- [[Android, Kotlin] Retrofit, Room과 같은 동작에서의 Coroutine 쓰레드 처리](https://hungseong.tistory.com/9)