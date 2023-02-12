---
title:  "[Android] Room"

categories:
  - Android
tags:
  - [Android, Android 정리, Room]

toc: true
toc_sticky: true
 
date: 2023-02-11 08:46:02+0900
last_modified_at: 2023-02-11 08:46:02+0900

---

<br>
<br>
<br>

> 아래 내용은 [Android Developer 공식 문서](https://developer.android.com/training/data-storage/room?hl=ko) 내용을 참고하여 작성한 자습 기록입니다.

<br>
<br>

# 📚Android 정리 - Room

<br>

## 📔Room이란

Room은 SQLite의 전체 기능을 활용하면서 보다 강력한 데이터베이스 액세스를 허용하기 위해 SQLite에 대한 추상화 계층을 제공하는 라이브러리이다. Room은

* 더 나은 컴파일 시간
* 향상된 테스트
* 더 쉬운 데이터 마이그레이션 처리

등 기존 SQLite 데이터베이스 액세스에 비해 많은 이점을 제공한다.

기존 SQLite를 활용해서도 데이터베이스 작업을 처리할 수 있지만 아래와 같은 이유로 Room 라이브러리 사용이 권장된다.

![](/assets/images/android/room/room1.png)<center>https://developer.android.com/training/data-storage/sqlite?hl=ko</center>

<br>
<br>

## 📔Room 구성요소

<br>

![](/assets/images/android/room/room2.png)

<br>
<br>

### 📖 Entity
데이터베이스의 테이블을 가리킨다. Entity는 테이블의 구조와 테이블의 열을 정의한다.

### 📖 DAO(Data Access Object)
데이터베이스에서 데이터를 읽고 쓰기 위한 API를 제공한다. 다시 말해, 데이터 삽입, 업데이트 및 검색과 같은 데이터베이스 작업에 해당하는 메서드를 정의한다.

### 📖 Database
데이터베이스에 대한 기본 액세스 포인트 역할을 한다. Entity와 DAO를 포함하며 기본 SQLite 데이터베이스 생성 및 관리를 담당한다.

<br>
<br>

## 📔Room 사용해보기

Room을 사용하기 위해서 성과 이름을 입력한 뒤 버튼을 누르면, 이름이 데이터베이스에 추가되도록 간단하게 구현해 보았다.

<br>

### 📖 라이브러리 추가
Room 라이브러리를 추가하기 위해 아래 항목들을 build.gradle에 추가해 준다.

```kotlin
val room_version = "2.5.0"

implementation("androidx.room:room-runtime:$room_version")
annotationProcessor("androidx.room:room-compiler:$room_version")
```

### 📖 UI 구현

UI는 간단하게 EditText 2개와 Button 1개로 구성해 주었다.

```kotlin
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:padding="32dp"
    tools:context=".MainActivity">

    <TextView
        android:id="@+id/titleTextView"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="정보"
        android:textSize="34sp"
        android:textStyle="bold"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent"
        app:layout_constraintVertical_bias="0.2" />

    <com.google.android.material.textfield.TextInputLayout
        android:id="@+id/firstNameTextInputLayout"
        style="@style/Widget.MaterialComponents.TextInputLayout.OutlinedBox"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:layout_marginTop="40dp"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toBottomOf="@id/titleTextView">

        <com.google.android.material.textfield.TextInputEditText
            android:id="@+id/firstNameTextInputEditText"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:hint="성" />

    </com.google.android.material.textfield.TextInputLayout>

    <com.google.android.material.textfield.TextInputLayout
        android:id="@+id/lastNameTextInputLayout"
        style="@style/Widget.MaterialComponents.TextInputLayout.OutlinedBox"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:layout_marginTop="20dp"
        app:layout_constraintEnd_toEndOf="@id/firstNameTextInputLayout"
        app:layout_constraintStart_toStartOf="@id/firstNameTextInputLayout"
        app:layout_constraintTop_toBottomOf="@id/firstNameTextInputLayout">

        <com.google.android.material.textfield.TextInputEditText
            android:id="@+id/lastNameTextInputEditText"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:hint="이름" />

    </com.google.android.material.textfield.TextInputLayout>

    <Button
        android:id="@+id/addButton"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:layout_marginTop="40dp"
        android:text="확인"
        app:layout_constraintEnd_toEndOf="@id/firstNameTextInputLayout"
        app:layout_constraintStart_toStartOf="@id/firstNameTextInputLayout"
        app:layout_constraintTop_toBottomOf="@id/lastNameTextInputLayout" />

</androidx.constraintlayout.widget.ConstraintLayout>
```

![](/assets/images/android/room/room3.png)

<br>
<br>

### 📖 Entity - User.kt

테이블을 가리키는 Entity 클래스를 정의한다. 각각의 User 인스턴스는 user 테이블의 행 하나를 나타낸다.

```kotlin
@Entity(tableName = "user_table")
data class User(
    @ColumnInfo(name = "first_name") val firstName: String,
    @ColumnInfo(name = "last_name") val lastName: String,
    @PrimaryKey(autoGenerate = true) val uid: Int = 0,
)
```

* @Entity 주석은 해당 Entity 클래스의 데이터가 저장될 데이터베이스의 테이블 이름을 정의하는 데 사용된다. 매개변수인 `tableName`으로 테이블의 이름을 변경할 수 있고 기본적으로 클래스 이름이 테이블의 이름이 된다.
* data class 내의 속성들은 테이블의 컬럼명이 된다. 컬럼명을 변경하고자 한다면, @ColumnInfo 주석을 추가한 뒤 매개변수 name에 이름을 지정해주면 된다.
* @PrimaryKey 주석은 테이블의 primary key를 정의하기 위해 사용된다. autoGenerate 파라미터의 값으로 true를 지정할 시, primary key의 값이 Room 라이브러리에 의해 자동으로 생성된다.

<br>
<br>

### 📖 Dao - UserDao.kt

Dao 인터페이스에서 SQL 쿼리를 메소드와 연결해 호출할 수 있도록 한다.

```kotlin
@Dao
interface UserDao {

    @Query("SELECT * FROM user")
    fun getAll(): List<User>

    @Insert
    fun insertUser(user: User)

    @Query("DELETE FROM user")
    fun deleteAll()
}
```

* @Dao 주석으로 해당 인터페이스가 Dao 임을 명시한다
* Entity 테이블의 모든 데이터를 검색할 수 있는 쿼리를 작성해 주었고 이를 @Query 주석의 인자로 넣어준 뒤, 메서드와 연결해 준다
* @Insert 데이터를 테이블에 추가하는 주석으로 추가적인 쿼리문을 작성하지 않아도 자동으로 추가될 수 있도록 지원해 준다.

<br>
<br>

### 📖 AppDatabase

```kotlin
@Database(entities = [User::class], version = 1)
abstract class AppDatabase : RoomDatabase() {
    abstract fun userDao(): UserDao
}
```

* @Database 주석은 해당 클래스가 Room 데이터베이스임을 나타낸다. entities 매개변수에 Entity 클래스들을 지정하고, version 매개변수는 데이터베이스의 버전을 정의한다
* 추상 클래스로 정의해야 하고, 내부의 추상 메서드로 Dao에 접근할 수 있도록 한다

### 📖 MainActivity.kt

이제 데이터베이스 생성을 위한 준비를 모두 마무리했으므로 생성하면 된다. onCreate() 내부에 db를 생성하고 이후 데이터를 처리할 경우 메인 스레드가 아닌 작업 스레드에서 실행하도록 한다. 

```kotlin
class MainActivity : AppCompatActivity() {


    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        val db =
            Room.databaseBuilder(
                applicationContext,
                AppDatabase::class.java,
                "userDB"
            ).build()


        val firstNameEditText = findViewById<EditText>(R.id.firstNameTextInputEditText)
        val lastNameEditText = findViewById<EditText>(R.id.lastNameTextInputEditText)
        val addButton = findViewById<Button>(R.id.addButton)

        addButton.setOnClickListener {
            Thread {
                val firstName = firstNameEditText.text.toString()
                val lastName = lastNameEditText.text.toString()
                db.userDao().insertUser(User(firstName, lastName))

                runOnUiThread {
                    firstNameEditText.setText("")
                    lastNameEditText.setText("")
                }
            }.start()
        }
    }
}
```