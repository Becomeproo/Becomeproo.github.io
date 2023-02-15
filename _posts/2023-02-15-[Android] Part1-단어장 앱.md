---
title:  "[Android] Part1-단어장 앱"

categories:
  - Android
tags:
  - [Android, Kotlin, 안드로이드 강의]

toc: true
toc_sticky: true
 
date: 2023-02-15 08:11:18+0900
last_modified_at: 2023-02-15 08:11:31+0900
---

<br>
<br>
<br>

> 아래 내용은 패스트 캠퍼스 [35개 프로젝트로 배우는 Android 앱 개발 feat. Jetpack Compose]의 강의 내용을 기반으로 작성한 자습 기록입니다.

<br><br>

# 📚단어장 앱

## ℹ️앱 설명

단어장 앱으로 단어를 추가, 삭제 및 수정을 할 수 있는 앱이다.

<br>

## ✅구현 기능

* 단어장 UI 구현
* 단어 추가
* 단어 수정
* 단어 삭제

<br>

## ✅사용되는 기능

* UI
  * RecyclerView, Adapter
  * TextInputLayout, TextInputEditText
  * ChipGroup, Chip

* Kotlin
  * data class

* Android
  * Room
  * registerForActivityResult
  * Parcelize

---

<br>

## 📔UI 구현 - 단어 목록을 표시하는 메인 화면 구현

우선 단어 목록에 있는 특정 단어를 클릭하였을 때, 단어와 뜻을 보여주는 TextView를 상단에 위치시켜 두었다. 그 아래 단어 목록을 보여주기 위해 RecyclerView를 만들어주었다.

TextView와 RecyclerView 사이에는 영역을 구분하기 위해서 구분선 역할을 할 View를 생성해 주었다.

우측에는 삭제와 수정 기능을 하는 ImageView를 위치시켜 두었는데, 여기서 ImageButton이 아닌 ImageView를 버튼으로서 주로 사용하는 이유는 ImageButton을 사용할 경우, 이미지를 포함한 버튼의 전체 영역이 위젯의 역할을 하기 때문에 배경 또한 같이 적용되어 따로 수정해야 할 수고가 생긴다. 반면, ImageView를 버튼으로 사용하였을 때에는 배경을 따로 지정할 필요 없이 이미지 외부에 누끼(?)가 그대로 적용되어 나오기 때문에 수고를 덜어줄뿐더러 여기에 padding 값을 지정해 주면 버튼으로서 적용될 영역도 설정할 수 있기 때문에 ImageView를 더 많이 사용한다.

본론으로 돌아와 삭제와 수정 기능을 할 두 ImageView를 Barrier 영역으로 감싸주어 그룹 영역을 지정해 주었고, 이를 대상으로 앞서 만들었던 TextView의 끝단 제약을 Barrier의 시작 부분에 적용해 주었다. 

마지막으로 단어 추가 기능을 수행하는 화면으로 이동시키는 버튼을 FloatingActionButton으로 만들어 주었다.

위 내용들의 결과는 다음과 같다.

<br>

### 📖 activity_main.xml
```kotlin
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">

    <TextView
        android:id="@+id/textTextView"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:layout_margin="24dp"
        android:textSize="30sp"
        app:layout_constraintEnd_toEndOf="@id/barrier"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent"
        tools:text="단어 입니다" />

    <TextView
        android:id="@+id/meanTextView"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:layout_marginTop="16dp"
        android:textSize="20sp"
        app:layout_constraintEnd_toEndOf="@id/barrier"
        app:layout_constraintStart_toStartOf="@id/textTextView"
        app:layout_constraintTop_toBottomOf="@id/textTextView"
        tools:text="뜻 입니다" />

    <ImageView
        android:id="@+id/deleteImageView"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginTop="4dp"
        android:padding="16dp"
        android:src="@drawable/ic_baseline_delete_24"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="@id/barrier"
        app:layout_constraintTop_toTopOf="parent" />

    <ImageView
        android:id="@+id/editImageView"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:padding="16dp"
        android:src="@drawable/ic_baseline_edit_24"
        app:layout_constraintEnd_toEndOf="@id/deleteImageView"
        app:layout_constraintTop_toBottomOf="@id/deleteImageView" />

    <androidx.constraintlayout.widget.Barrier
        android:id="@+id/barrier"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        app:barrierDirection="start"
        app:constraint_referenced_ids="deleteImageView, editImageView" />

    <View
        android:id="@+id/line"
        android:layout_width="match_parent"
        android:layout_height="1dp"
        android:layout_marginTop="24dp"
        android:background="#CBCBCB"
        app:layout_constraintTop_toBottomOf="@id/meanTextView" />

    <androidx.recyclerview.widget.RecyclerView
        android:id="@+id/wordRecyclerView"
        android:layout_width="match_parent"
        android:layout_height="0dp"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintTop_toBottomOf="@id/line" />

    <com.google.android.material.floatingactionbutton.FloatingActionButton
        android:id="@+id/addButton"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:src="@drawable/ic_baseline_add_24"
        android:layout_margin="30dp"
        android:backgroundTint="#FFC107"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toEndOf="parent" />

</androidx.constraintlayout.widget.ConstraintLayout>
```

![](/assets\images\fastcampus\part1\notepad/notepad1.png)

<br>

다음으로 RecyclerView에 표시될 아이템 뷰를 만들어보자.

아이템 뷰는 단어와 뜻을 표시하고 해당 단어의 품사를 표시하도록 구성해 주었다. 여기서 품사를 표시할 위젯으로 Chip을 사용할 것이다. Chip은 요즘 웹에서도 태그 같은 것을 표시할 때 많이 보이는 위젯으로 더욱 깔끔한 UI를 구현할 수 있게 해준다. 

<br>

### 📖 item_word.xml
```kotlin
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:padding="16dp">

    <TextView
        android:id="@+id/textTextView"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:textSize="20sp"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintTop_toTopOf="parent"
        tools:text="단어" />

    <TextView
        android:id="@+id/meanTextView"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginTop="8dp"
        app:layout_constraintEnd_toEndOf="@id/textTextView"
        app:layout_constraintTop_toBottomOf="@id/textTextView"
        tools:text="뜻" />

    <com.google.android.material.chip.Chip
        android:id="@+id/typeChip"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="품사" />

</androidx.constraintlayout.widget.ConstraintLayout>
```

![](/assets\images\fastcampus\part1\notepad/notepad2.png)

<br>

위 아이템 뷰가 적용된 RecyclerView의 화면은 다음과 같다. preview로 간단히 아이템 뷰를 적용시키는 방법은 RecyclerView의 속성으로 `tools:listitem="@layout/item_view"` 값을 적용시키면 된다.

![](/assets\images\fastcampus\part1\notepad/notepad3.png)

<br>

## 📔UI 구현 - 단어 추가 및 수정 기능 역할 화면 구현

단어 추가 및 수정 기능을 하는 화면을 구현해 보자.

상단의 화면 제목을 달아주었고, 단어와 뜻, 두 가지에 대한 입력을 받기 위해 두 개의 EditText를 만들어주었다. 여기서 EditText는 TextInputEditText를 사용하였는데, TextInputLayout에 속하여 사용된다. TextInputLayout은 현재까지 입력된 단어의 길이, 조건을 충족하지 못하였을 때 에러 UI 변환 등 EditText에 더욱 풍부한 UI를 제공하도록 돕는 위젯이다. 나중에 더욱 자세히 알아보도록 하자. 품사는 Chip을 통해 선택할 수 있도록 구현할 것인데, 여기서 다수의 Chip을 사용한 구현은 ChipGroup으로 UI 영역을 지정한 후 코드 상에서 각각의 Chip들을 생성한다. 따라서 해당 UI 상에는 보이지 않지만, ChipGroup이 뜻을 나타내는 EditText와 추가 버튼 사이에 위치해있다.

### 📖 activity_add.xml
```kotlin
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:padding="24dp"
    tools:context=".AddActivity">

    <TextView
        android:id="@+id/titleTextView"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_marginTop="30dp"
        android:gravity="center"
        android:text="단어 추가"
        android:textSize="30sp"
        app:layout_constraintTop_toTopOf="parent" />

    <com.google.android.material.textfield.TextInputLayout
        android:id="@+id/textInputLayout"
        style="@style/Widget.MaterialComponents.TextInputLayout.OutlinedBox"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:layout_marginTop="16dp"
        app:counterEnabled="true"
        app:counterMaxLength="30"
        app:errorEnabled="true"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toBottomOf="@id/titleTextView">

        <com.google.android.material.textfield.TextInputEditText
            android:id="@+id/textInputEditText"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:hint="단어" />

    </com.google.android.material.textfield.TextInputLayout>

    <com.google.android.material.textfield.TextInputLayout
        android:id="@+id/meanTextInputLayout"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_marginTop="16dp"
        app:counterEnabled="true"
        app:layout_constraintTop_toBottomOf="@id/textInputLayout">

        <com.google.android.material.textfield.TextInputEditText
            android:id="@+id/meanTextInputEditText"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:hint="뜻" />

    </com.google.android.material.textfield.TextInputLayout>

    <com.google.android.material.chip.ChipGroup
        android:id="@+id/typeChipGroup"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_marginTop="16dp"
        app:layout_constraintTop_toBottomOf="@id/meanTextInputLayout"
        app:selectionRequired="true"
        app:singleSelection="true" />

    <Button
        android:id="@+id/addButton"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_marginTop="16dp"
        android:text="추가"
        app:layout_constraintTop_toBottomOf="@id/typeChipGroup" />

</androidx.constraintlayout.widget.ConstraintLayout>
```

![](/assets\images\fastcampus\part1\notepad/notepad4.png)

<br>
<br>

만들어 두었던 ChipGroup에 각각의 Chip을 추가하도록 한다. AddActivity에서 initViews() 메서드 내부에 8품사로 이루어진 리스트를 변수에 지정해 주었다. 그리고 Chip을 반환하고 8품사의 텍스트를 인자로 받는 createChip() 메서드를 만들어 각각의 Chip의 텍스트를 지정하고 선택 가능하도록 설정해 준 뒤 반환하도록 만들어주었다. 다시 돌아와 ChipGroup의 addView()에 createChip() 메서드로 만들어 둔 각각의 Chip들을 인자로 넣어주면 생성되는 것을 볼 수 있다.

### 📖 AddActivity.kt

```kotlin
class AddActivity : AppCompatActivity() {
    private lateinit var binding: ActivityAddBinding

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        binding = ActivityAddBinding.inflate(layoutInflater)
        setContentView(binding.root)

        initViews()
    }

    private fun initViews() {
        val types = listOf(
            "명사", "동사", "대명사", "형용사", "부사", "감탄사", "전치사", "접속사"
        )
        binding.typeChipGroup.apply {
            types.forEach { text ->
                addView(createChip(text))
            }
        }
    }

    private fun createChip(text: String): Chip {
        return Chip(this).apply {
            setText(text)
            isCheckable = true
            isClickable = true
        }
    }
}
```

![](/assets\images\fastcampus\part1\notepad/notepad5.jpg)

<br>
<br>

## 📔기능 구현 - Room 데이터베이스 구축

<br>

Room을 활용한 데이터베이스로 기능을 구현하므로 Room 라이브러리와 수정 기능을 적용할 때 해당 데이터 클래스 값으로 데이터를 전달할 것이므로 parcelize에 대한 플러그인도 추가해 주었다.


### 📖 build.gradle (:app)

```kotlin
plugins {
    ...
    id 'kotlin-kapt'
    id 'kotlin-parcelize'
}

...

dependencies {
    ...
    // room
    def room_version = "2.5.0"
    implementation("androidx.room:room-runtime:$room_version")
    kapt("androidx.room:room-compiler:$room_version")
}
```

<br>

다음으로 데이터베이스를 구축하기 위한 준비를 해보도록 하자. 데이터베이스의 테이블 역할을 할 Entity로 Word라는 이름의 데이터 클래스를 만들어 준다. 해당 클래스가 데이터베이스의 Entity임을 표시하도록 `@Entity` 어노테이션을 달아주고 테이블 이름은 보통 클래스의 이름으로 기본값이 지정되지만 명시적으로 `tableName = "word"`을 작성하여 테이블 이름을 지정해 주었다. 속성값은 총 4개로 단어, 뜻, 품사가 들어간다. 이 세 개의 값 모두 String 타입으로 지정해 주었다. 나머지 하나는 데이터베이스에서 각각의 값들을 구별할 수 있도록 id 값을 지정해 주었는데 @PrimaryKey 어노테이션을 사용하여 해당 값이 구분 키라는 것을 명시해 주었다. 매개변수인 `autoGenerate = true` 값은 데이터가 추가될 때마다 id 값을 자동으로 생성하도록 지정해 준 것이다.

### 📖 Word.kt

```kotlin
@Entity(tableName = "word")
data class Word(
    val text: String,
    val mean: String,
    val type: String,
    @PrimaryKey(autoGenerate = true) val id: Int = 0,
)
```
  
<br>

다음으로 데이터베이스의 쿼리 작업을 메서드와 연결시키는 Dao 인터페이스에 대해서 구현해 보자. WordDao라는 이름으로 인터페이스를 생성하고 이 또한 Dao임을 명시하도록 `@Dao` 어노테이션을 작성해 주었다. 인터페이스 내부에는 각각의 기능을 할 쿼리문을 작성해 준다. 쿼리문을 사용하여 작업할 내용은 총 5가지로 다음과 같다.

1. 모든 데이터 출력
2. 가장 최근의 데이터 출력
3. 데이터 추가
4. 데이터 삭제
5. 데이터 수정

여기서 출력 역할을 하는 메서드의 경우 `@Query` 어노테이션을 사용하여 직접 쿼리문을 작성해 준다. 이외의 삭제, 추가, 수정과 같은 기능은 Room에서 각각 @Insert, @Delete, @Update를 지원해 주므로 따로 쿼리문을 작성할 필요 없이 기능에 필요한 데이터를 매개변수로 지정해 주기만 하면 된다.

### 📖 WordDao.kt

```kotlin
@Dao
interface WordDao {
    @Query("SELECT * FROM word ORDER BY id DESC") // 여기서의 word는 Word 클래스에서 tableName으로 지정된 word
    fun getAll(): List<Word>

    @Query("SELECT * FROM word ORDER BY id DESC LIMIT 1")
    fun getLatestWord(): Word

    @Insert
    fun insert(word: Word)

    @Delete
    fun delete(word: Word)

    @Update
    fun update(word: Word)
}
```

<br>

마지막으로 위에서 만들어 둔 내용으로 구성된 데이터베이스를 만들어 주도록 한다. AppDatabase라는 이름의 클래스를 만들어주고 `@Database` 어노테이션을 사용하여 해당 클래스가 데이터베이스 클래스임을 명시해 준다. 여기서 어노테이션의 매개변수로 entities와 version이 요구된다. entities의 경우, 앞서 만들어 두었던 Word 데이터 클래스를 인자로 넣어주면 되고 이외 다른 데이터 클래스도 적용시킬 경우 매개변수가 배열이기 때문에 포함시켜 주면 된다. 버전은 1로 지정해 주었다.

AppDatabase 클래스는 RoomDatabase 클래스를 상속받는다. 또한 해당 클래스를 추상 클래스로 명시해 준다. 앞서 만들어 두었던 Dao를 반환해 줄 추상 메서드도 필요하기 때문에 `wordDao()` 메서드도 만들어 준다.

마지막으로 런타임 상에서 데이터베이스는 여러 개가 생성될 필요가 없기 때문에 companion object를 사용하여 한 번만 사용되도록 해준다.

> ### 💡 궁금증 <br>
> 해당 코드 내에서 synchronized() 메서드는 왜 사용되는가? <br><br>
> 📌앞서 언급했듯이 AppDatabase는 런타임 상에서 한 번만 사용될 필요가 있다. 만일 데이터베이스의 여러 스레드가 동시에 액세스한다면 무거운 연산을 초래할 수 있기 때문이다. 이는 앱이 사용되는 데 있어서 좋은 작업이 아니다. 따라서 오직 하나의 스레드만 접근할 수 있도록 제한하기 위해 synchronized() 메서드가 사용된다. AppDatabase 클래스의 인스턴스가 하나만 생성되도록 하여 만일 두 개의 스레드가 동시에 데이터베이스에 액세스하려고 할 경우, synchronized 블록에 액세스하는 첫 번째 스레드가 인스턴스를 생성하고 두 번째 스레드는 첫 번째 스레드가 완료할 때까지 대기하게 된다. 다시 말해 synchronized() 메서드는 멀티 스레드 환경에서 스레드 안정성을 보장한다.

궁금증도 해결되었으니 다시 돌아와서 null값으로 지정되어 있던 INSTANCE 변수에 `Room.databaseBuilder()`를 사용하여 인자로 현재 context의 applicationContext, 사용되는 AppDatabase, 데이터베이스의 이름을 넣어주고 마지막에 build를 해주면 데이터베이스 인스턴스가 생성된다. 마지막으로 이를 반환해 주면 데이터베이스를 사용할 준비는 끝났다.

### 📖 AppDatabase.kt

```kotlin
@Database(entities = [Word::class], version = 1)
abstract class AppDatabase : RoomDatabase() {
    abstract fun wordDao(): WordDao

    companion object {
        private var INSTANCE: AppDatabase? = null
        fun getInstance(context: Context): AppDatabase? {
            if (INSTANCE == null) {
                synchronized(AppDatabase::class.java) {
                    INSTANCE = Room.databaseBuilder(
                        context.applicationContext,
                        AppDatabase::class.java,
                        "app-database.db"
                    ).build()
                }
            }
            return INSTANCE
        }
    }
}
```

<br><br>

## 📔기능 구현 - 단어 추가

<br>

단어를 추가하기 위해서는 AddActivity에서 기능을 구현하기 때문에 Intent를 통해 화면을 이동시켜주어야 한다. FloatingActionButton을 클릭하였을 때 AddActivity로 이동하도록 Intent를 구현해 주도록 한다.

이후 AddActivity에서 추가 버튼을 눌렀을 때 단어가 DB에 저장되고 추가되므로 addButton으로 클릭 리스너를 설정해 준다. 리스너 내부에서 add() 메서드를 호출하도록 한다. 메서드 내부에서 각각의 속성값들을 변수로 만들어주는데 단어와 뜻은 EditText에서, 품사는 선택된 Chip의 id 값을 받아야 한다. 이는 findViewById를 통해 받아올 수 있다. 이렇게 받아온 세 개의 값으로 Word 클래스를 생성해 주고 변수에 지정해 준다.

변수로 지정된 데이터 클래스의 값을 데이터베이스에 넣어주도록 하자. DB 작업은 다른 스레드를 열어 작업하는 것을 지향하므로 Thread 블록을 열어주고 앞서 만들어 두었던 AppDatabase를 열어 db를 가져온 뒤, Dao 인터페이스의 insert 메서드를 사용한다. 해당되는 인자 값으로 데이터 클래스의 변숫값을 넣어주면 된다. 추가가 완료되었음을 표시하기 위해 Toast를 사용해 알려주고, 이 과정에서 Toast는 UI에 표시되므로 runOnUiThread 블록을 생성해 실행 시키도록 한다. 모든 db 작업 및 ui 작업이 완료되었을 경우, finish()를 사용해 AddActivity를 종료한다. 마지막으로 Thread 블록 마지막에 start()를 선언하는 것을 잊지말자.

### 📖 AddActivity.kt

```kotlin
class AddActivity : AppCompatActivity() {
    private lateinit var binding: ActivityAddBinding

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        binding = ActivityAddBinding.inflate(layoutInflater)
        setContentView(binding.root)

        initViews()
        binding.addButton.setOnClickListener {
          add()
        }
    }

    private fun initViews() {
        val types = listOf(
            "명사", "동사", "대명사", "형용사", "부사", "감탄사", "전치사", "접속사"
        )
        binding.typeChipGroup.apply {
            types.forEach { text ->
                addView(createChip(text))
            }
        }
    }

    private fun createChip(text: String): Chip {
        return Chip(this).apply {
            setText(text)
            isCheckable = true
            isClickable = true
        }
    }

    private fun add() {
        val text = binding.textInputEditText.text.toString()
        val mean = binding.meanTextInputEditText.text.toString()
        val type = findViewById<Chip>(binding.typeChipGroup.checkedChipId).text.toString()
        val word = Word(text, mean, type)

        Thread {
            AppDatabase.getInstance(this)?.wordDao()?.insert(word)
            runOnUiThread {
                Toast.makeText(this, "저장을 완료했습니다.", Toast.LENGTH_SHORT).show()
            }
            finish()
        }.start()
    }
}
```

결과를 보면, 입력된 각각의 정보가 DB상에 저장된 것을 볼 수 있다.

![](/assets\images\fastcampus\part1\notepad/notepad7.jpg)

![](/assets\images\fastcampus\part1\notepad/notepad6.png)


<br>

데이터가 추가되었으니 UI에도 추가를 해주도록 해야 한다. 저장된 데이터들을 RecyclerView에 보여줄 것이므로 먼저 Adapter를 생성해 주어야 한다.

WordAdapter라는 이름의 클래스를 생성해 준다. 해당 클래스는 RecyclerView.Adapter를 상속받고 사용될 클래스로 WordViewHolder를 지정한다. 클래스의 속성으로 Word 타입의 리스트를 가진다.

먼저 클래스 내부에 ViewHolder로 쓰일 WordViewHolder를 생성해 주었다. 뷰 바인딩을 사용하므로 속성으로 아이템 뷰의 바인딩을 가지며 ViewHolder를 상속받는다. 클래스 생성뿐만 아니라 뷰 또한 해당 클래스 내부에서 연결해 주기 위해 bind() 메서드를 만들어 주었다. 메서드 내부에서 데이터 클래스의 속성과 이에 해당하는 ui에 각각의 값들을 연결해 준다.

onCreateViewHolder() 메서드에서 연결될 레이아웃을 지정해 주었고, onBindViewHolder를 통해 ui와 데이터를 연결시켜 주었다.

### 📖WordAdapter.kt

```kotlin
class WordAdapter(
    val list: MutableList<Word>
) : RecyclerView.Adapter<WordAdapter.WordViewHolder>() {

    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): WordViewHolder {
        val inflater =
            parent.context.getSystemService(Context.LAYOUT_INFLATER_SERVICE) as LayoutInflater
        val binding = ItemWordBinding.inflate(inflater, parent, false)
        return WordViewHolder(binding)
    }

    override fun onBindViewHolder(holder: WordViewHolder, position: Int) {
        val word = list[position]
        holder.bind(word)
    }

    override fun getItemCount(): Int {
        return list.size
    }

    class WordViewHolder(private val binding: ItemWordBinding) :
        RecyclerView.ViewHolder(binding.root) {
        fun bind(word: Word) {
            binding.apply {
                textTextView.text = word.text
                meanTextView.text = word.mean
                typeChip.text = word.type
            }
        }
    }
}
```

MainActivity로 돌아와 RecyclerView를 생성해 준다. initRecyclerView() 메서드를 생성한 뒤 adapter와 layoutManager, 그리고 추가적으로 구분선을 넣어주었다. 구현이 잘 되었는지 더미 데이터를 넣어 확인해 보자.

### 📖MainActivity.kt

```kotlin
class MainActivity : AppCompatActivity() {
    private lateinit var binding: ActivityMainBinding
    private lateinit var wordAdapter: WordAdapter

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        binding = ActivityMainBinding.inflate(layoutInflater)
        setContentView(binding.root)

        initRecyclerView()
        binding.addButton.setOnClickListener {
            Intent(this, AddActivity::class.java).let {
                startActivity(it)
            }
        }
    }

    private fun initRecyclerView() {

        val dummyWord = mutableListOf(
            Word("duck", "오리", "명사"),
            Word("note", "공책", "명사"),
            Word("nail", "손톱", "명사"),
            Word("book", "책", "명사"),
            Word("pencil", "연필", "명사")
        )

        wordAdapter = WordAdapter(dummyWord, this)
        binding.wordRecyclerView.apply {
            adapter = wordAdapter
            layoutManager =
                LinearLayoutManager(applicationContext, LinearLayoutManager.VERTICAL, false)
            val dividerItemDecoration =
                DividerItemDecoration(applicationContext, DividerItemDecoration.VERTICAL)
            addItemDecoration(dividerItemDecoration)
        }
    }
}
```

잘 구현된 것을 확인할 수 있다.

![](/assets\images\fastcampus\part1\notepad/notepad8.jpg)

<br>

더미 데이터가 아닌 데이터베이스에 저장된 값을 불러와야 하기 때문에 더미 데이터는 지워주도록 한다. initRecyclerView() 메서드에서 스레드를 열고 추가했을 때와 같이 AppDatabase를 호출하여 db를 불러온 뒤, dao의 getAll() 메서드를 사용한다. 값이 없을 경우 emptyList()로 설정해 준다. 이 값을 "list" 이름의 변수에 넣어주고 WordAdapter의 속성으로 지정해두었던 MutableList 자료형의 속성값에 addAll()을 사용하여 불러온 값들을 넣어준다. 이로써 데이터를 Adapter로 가져와 주었고 이를 UI로 표시해야 하기 때문에 runOnUiThread를 열어 notifyDataSetChanged() 메서드를 사용하여 변경 사항을 알려 UI가 업데이트 되도록 한다.

구현된 결과를 보면 다음과 같다.

```kotlin
class MainActivity : AppCompatActivity() {

    ...

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        binding = ActivityMainBinding.inflate(layoutInflater)
        setContentView(binding.root)

        ...
    }

    private fun initRecyclerView() {
        wordAdapter = WordAdapter(mutableListOf(), this)
        binding.wordRecyclerView.apply {
            adapter = wordAdapter
            layoutManager =
                LinearLayoutManager(applicationContext, LinearLayoutManager.VERTICAL, false)
            val dividerItemDecoration =
                DividerItemDecoration(applicationContext, DividerItemDecoration.VERTICAL)
            addItemDecoration(dividerItemDecoration)
        }

        Thread {
            val list = AppDatabase.getInstance(this)?.wordDao()?.getAll() ?: emptyList()
            wordAdapter.list.addAll(list)
            runOnUiThread { wordAdapter.notifyDataSetChanged() }
        }.start()
    }

}
```

앞서 추가했던 dog와 더불어 새로 추가된 단어가 db와 ui에 적용된 것을 볼 수 있다.

![](/assets\images\fastcampus\part1\notepad/notepad9.jpg)

<br>

하지만 위의 결과는 앱이 시작되면서 호출되는 initRecyclerView()에서 적용될 뿐 단어를 계속해서 추가했을 때에는 추가된 단어가 ui에 업데이트되지 않는다. 단어가 추가되었을 때 적용되는 메서드를 따로 추가해 주어야 한다. AddActivity가 호출되었다가 종료할 때 추가된 단어를 다시 MainActivity에 전달해 주어야 한다. 이를 위해서 사용되는 기능은 registerForActivityResult이다.

이전에는 StartActivityForResult를 사용하면 됐으나 현재 deprecated 되었고 대신 사용되는 것이 바로 registerForActivityResult이다. 따라서 해당 기능을 사용하여 ui를 업데이트 해보도록 하자.

클래스의 변수로 "updateAddWordResult"를 만들어 여기에 registerForActivityResult를 지정해 준다. registerForActivityResult는 두 개의 매개변수를 요구한다. 하나는 사용될 ActivityResultContracts를, 다른 하나는 이를 통한 구현이다. 따라서 첫 번째 매개변수에는 ActivityResultContracts.StartActivityForResult()를 넣어주면 되고, 값을 받아온 구현 결과로 resultCode와 올바른 값을 받아왔을 경우, updateAddWord()를 호출하도록 했다.

### 📖MainActivity.kt
```kotlin
class MainActivity : AppCompatActivity(), WordAdapter.ItemClickListener {
    private lateinit var binding: ActivityMainBinding
    private lateinit var wordAdapter: WordAdapter
    
    private val updateAddWordResult = registerForActivityResult(
        ActivityResultContracts.StartActivityForResult()
    ) { result ->
        val isUpdated = result.data?.getBooleanExtra("isUpdated", false) ?: false

        if (result.resultCode == RESULT_OK && isUpdated) {
            updateAddWord()
        }
    }

    ...

}

```

위에서 설정한 값을 적용하기 위해 Intent를 사용해 AddActivity로 보내주어야 한다. 앞서 작성했던 Intent 코드를 수정해 준다.

### 📖MainActivity.kt

```kotlin
override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    binding = ActivityMainBinding.inflate(layoutInflater)
    setContentView(binding.root)

    initRecyclerView()
    binding.addButton.setOnClickListener {
        Intent(this, AddActivity::class.java).let {
            updateAddWordResult.launch(it)
        }
    }
}
```

Intent로 넘어온 값을 AddActivity에서 처리해서 다시 보내주어야 한다. AddActivity로 와서 add() 메서드에서 다음과 같이 단어가 추가되었음을 알려주는 intent를 다시 보내주어야 한다.

### 📖AddActivity.kt

```kotlin
private fun add() {
    val text = binding.textInputEditText.text.toString()
    val mean = binding.meanTextInputEditText.text.toString()
    val type = findViewById<Chip>(binding.typeChipGroup.checkedChipId).text.toString()
    val word = Word(text, mean, type)

    Thread {
        AppDatabase.getInstance(this)?.wordDao()?.insert(word)
        runOnUiThread {
            Toast.makeText(this, "저장을 완료했습니다.", Toast.LENGTH_SHORT).show()
        }

        val intent = Intent().putExtra("isUpdated", true)
        setResult(RESULT_OK, intent)

        finish()
    }.start()
}
```



다시 MainActivity로 돌아와 추가된 단어를 가져왔음을 알리는 intent를 수신한 경우, updateAddWord() 메서드에서 db에 접근하기 위해 스레드를 열고 Dao 메서드로 getLatestWord()를 사용해 가장 최근에 추가된 값을 가져온다. 해당 값을 "wordAdapter"에 새로 추가해 준 뒤, notifyDataSetChanged()를 통해 변경 사항을 적용시켜 준다.

### 📖MainActivity.kt

```kotlin
private fun updateAddWord() {
    Thread {
        AppDatabase.getInstance(this)?.wordDao()?.getLatestWord()?.let { word ->
            wordAdapter.list.add(0, word)
            runOnUiThread {
                wordAdapter.notifyDataSetChanged()
            }
        }
    }.start()
}
```

위 과정들을 통해 추가된 값이 바로 UI에 적용된 것을 볼 수 있다.

![](/assets\images\fastcampus\part1\notepad/notepad10.jpg)

## 📔기능 구현 - 단어 삭제

단어 삭제는 단어 목록에 있는 단어를 선택했을 때, 해당 단어와 뜻이 상단 TextView 두 개에 표시되고 삭제 이미지 버튼을 눌렀을 때 삭제되도록 처리한다. 이를 위해서 RecyclerView의 각각의 아이템이 클릭이 될 수 있도록 먼저 처리해 주어야 한다.

WordAdapter의 속성으로 기본값이 null인 ItemClickListener를 추가로 정의해 준다. 클래스 내부에 ItemClickListener 인터페이스를 생성해 주고 오버라이딩될 메서드로 onClick()을 생성해 주었다. onBindViewHolder()에서 "holder"의 itemView를 클릭했을 때 반응함을 적용하기 위해 클릭 리스너를 구현해 주었고, 그 내부에 ItemClickListener의 onClick() 메서드가 호출되도록 설정해 주었다.

### 📖WordAdapter.kt

```kotlin
class WordAdapter(
    val list: MutableList<Word>,
    private val itemClickListener: ItemClickListener? = null
) : RecyclerView.Adapter<WordAdapter.WordViewHolder>() {

    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): WordViewHolder {
        val inflater =
            parent.context.getSystemService(Context.LAYOUT_INFLATER_SERVICE) as LayoutInflater
        val binding = ItemWordBinding.inflate(inflater, parent, false)
        return WordViewHolder(binding)
    }

    override fun onBindViewHolder(holder: WordViewHolder, position: Int) {
        val word = list[position]
        holder.bind(word)
        holder.itemView.setOnClickListener { itemClickListener?.onClick(word) }
    }

    override fun getItemCount(): Int {
        return list.size
    }

    class WordViewHolder(private val binding: ItemWordBinding) :
        RecyclerView.ViewHolder(binding.root) {
        fun bind(word: Word) {
            binding.apply {
                textTextView.text = word.text
                meanTextView.text = word.mean
                typeChip.text = word.type
            }
        }
    }

    interface ItemClickListener {
        fun onClick(word: Word)
    }
}
```

클릭 리스너를 MainActivity에서 상속받도록 해준다. 상속하게 됨에 따라 `onClick()` 메서드를 재정의 해주어야 한다. 재정의 메서드 내부에서 클릭했을 때, 위에서 미리 정의해 준 선택된 단어의 레퍼런스를 갖고 있는 "selectedWord" 변수의 값을 초기화하고 데이터에 저장된 단어와 뜻 각각을 TextView의 text로 지정해 준다. 이렇게 하면 목록에 있는 단어 중 하나를 클릭했을 때 상단 TextView에 해당 단어와 뜻이 표시된다. 


### 📖MainActivity.kt

```kotlin
class MainActivity : AppCompatActivity(), WordAdapter.ItemClickListener {

    ...

    private var selectedWord: Word? = null

    ...

    override fun onCreate(savedInstanceState: Bundle?) {
        
        ...

    }

    ...

    override fun onClick(word: Word) {
        selectedWord = word
        binding.textTextView.text = word.text
        binding.meanTextView.text = word.mean
    }
}
```

이제 삭제 버튼을 눌렀을 때 데이터와 UI 상에서 해당 단어를 삭제해 주면 된다. 먼저 삭제 이미지를 눌렀을 때 클릭 리스너를 호출하도록 하고 그 내부에 delete() 메서드를 호출하도록 해준다. delete 메서드 내부에서는 먼저 "selectedWord" 변수에 값이 존재하는지 널 체크를 먼저 해준 뒤에 값이 존재한다면 스레드를 열고 db를 불러와 Dao에서 delete() 메서드를 넘겨준 뒤, runOnUiThread를 열어 "wordAdapter"에 저장되어 있는 해당 단어의 값을 삭제함과 더불어 notifyDataSetChanged()로 변경 사항을 적용시키도록 한다. 해당 과정들을 통해 데이터와 ui에서 모두 해당 단어가 삭제되었으므로 상단 TextView 두 개의 값을 모두 빈 값으로 변환시켜주고 "selectedWord"의 값을 다시 널 값으로 지정해 준다.

결과를 보면 해당 값이 db와 UI에서 모두 삭제되었음을 알 수 있다.

### 📖MainActivity.kt

```kotlin
private fun delete() {
    if(selectedWord == null) return

    Thread {
        selectedWord?.let { word ->
            AppDatabase.getInstance(this)?.wordDao()?.delete(word)
            runOnUiThread {
                wordAdapter.list.remove(word)
                wordAdapter?.notifyDataSetChanged()
                binding.textTextView.text = ""
                binding.meanTextView.text = ""
                Toast.makeText(this, "삭제가 완료됐습니다.", Toast.LENGTH_SHORT).show()
            }
            selectedWord = null
        }
    }.start()
}
```

![](/assets\images\fastcampus\part1\notepad/notepad11.jpg)
![](/assets\images\fastcampus\part1\notepad/notepad12.jpg)
![](/assets\images\fastcampus\part1\notepad/notepad13.png)

<br>
<br>

## 📔기능 구현 - 단어 수정

마지막으로 단어 수정 기능을 구현하여 보자. 단어 수정을 위해서는 단어 목록에서 단어를 클릭했을 때 해당 단어가 상단 TextView에 보이고 수정 버튼을 클릭했을 때 단어 수정 화면으로 넘어가 이를 수행하도록 구현해야 한다. 

먼저 직렬화와 역직렬화 과정을 통해 데이터를 주고받기 때문에 라이브러리를 선언했을 때 미리 작성해두었던 parcelize를 데이터 클래스에 어노테이션으로 선언해 주어야 하고 Parcelable을 상속해 주어야 한다.

### 📖Word.kt
```kotlin
@Parcelize
@Entity(tableName = "word")
data class Word(
    val text: String,
    val mean: String,
    val type: String,
    @PrimaryKey(autoGenerate = true) val id: Int = 0, // autoGenerate: 자동으로 값을 생성
) : Parcelable
```

다시 MainActivity로 돌아와 직렬화 과정으로 받아온 값을 처리하기 위해 단어를 추가했을 때도 사용했던 registerForActivityResult를 사용한다. ActivityResultContracts.StartActivityForResult를 인자로 넣어주는 과정까지는 추가 기능 구현 때와 같다. 이후 AddActivity에서 받아온 값을 처리하는 부분에서 직렬화와 역직렬화를 통해 데이터를 주고받기 때문에 updateEditWord() 메서드를 생성하여 받아온 값을 넘겨준다. 이렇게 정리한 값을 updateEditWordResult 변수에 넣어준다. 

다음으로 수정 버튼이 눌렸을 때 클릭 리스너가 호출되도록 구현해 준 뒤, edit() 메서드를 호출하도록 구현해 주었다. edit() 메서드에서, 먼저 클릭되었을 때 값이 있어야 하므로 앞서 삭제 기능을 구현할 때도 사용했던 selectedWord 변수를 사용해 해당 변수에 값이 존재할 경우에만 기능할 수 있도록 널 체크를 먼저 해준다. 이후 Intent를 사용해 "originWord"라는 키값으로 selectedWord를 AddActivity에 전달한다.

### 📖MainActivity.kt
```kotlin
class MainActivity : AppCompatActivity(), WordAdapter.ItemClickListener {
    
    ...

    private val updateEditWordResult = registerForActivityResult(
        ActivityResultContracts.StartActivityForResult()
    ) { result ->
        val editWord = result.data?.getParcelableExtra<Word>("editWord")
        if (result.resultCode == RESULT_OK && editWord != null) {
            updateEditWord(editWord)
        }
    }

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        binding = ActivityMainBinding.inflate(layoutInflater)
        setContentView(binding.root)

        ...

        binding.editImageView.setOnClickListener {
            edit()
        }
    }
    
    ...

    private fun edit() {
        if(selectedWord == null) return

        val intent = Intent(this, AddActivity::class.java).putExtra("originWord", selectedWord)
        updateEditWordResult.launch(intent)
    }

    ...

}
```

<br>

AddActivity에서 수정 기능을 처리하기 위해 변수로 Word 형의 "originWord"를 널 값으로 선언해두었다. 해당 값을 활용해서 확인 버튼이 눌렸을 때 "originWord"값이 널 값이면 추가를 의미하므로 add() 메서드를, 값이 존재한다면 수정을 의미하므로 edit() 메서드를 실행시키도록 코드를 수정해 두었다. 

edit() 메서드를 구현하기 앞서 MainActivity에서 보낸 데이터를 받아야 한다. 미리 선언해두었던 "originWord"에 `intent.getParcelableExtra("originWord")`를 통해 키값으로 넘겨준 값들을 넣어준다. 값을 제대로 받아왔다면 수정 기능이므로 각각의 TextView와 Chip에 기존에 저장해 두었던 단어와 뜻, 그리고 품사를 적용시켜 준다.

edit() 메서드에서도 add() 메서드와 같이 세 개의 값을 같은 방식으로 받아오지만, 추가 기능에서는 값을 Word()에 넣어줬다면, 수정 기능에서는 같은 방식으로 Word()에 값을 넣어줄 경우 값은 수정이 되지만, 자동으로 생성되는 id값이 값은 값이라 인지하지 못하고 또 다른 id값을 생산하게 된다. 따라서 수정 기능 구현해 있어서 Word()에 값을 넣어주기보다는 copy() 메서드를 사용하여 값으 변경해 주도록 한다. 이후 스레드를 열고 수정된 값의 변수인 editWord를 사용하여 db를 열어 값을 수정하는 update()를 호출하고 add() 메서드 때와 같이 수정된 사항을 intent에 다시 넣어 `RESULT_OK` 결과 코드와 함께 보내준다.

```kotlin
class AddActivity : AppCompatActivity() {

    ...

    private var originWord: Word? = null

    override fun onCreate(savedInstanceState: Bundle?) {
        
        ...

        binding.addButton.setOnClickListener {
            if (originWord == null) {
                add()
            } else edit()
        }
    }

    private fun initViews() {
        
        ...

        originWord = intent.getParcelableExtra("originWord")
        originWord?.let { word ->
            binding.textInputEditText.setText(word.text)
            binding.meanTextInputEditText.setText(word.mean)
            val selectedChip =
                binding.typeChipGroup.children.firstOrNull { (it as Chip).text == word.type } as? Chip
            selectedChip?.isChecked = true
        }
    }

    ...

    private fun edit() {
        val text = binding.textInputEditText.text.toString()
        val mean = binding.meanTextInputEditText.text.toString()
        val type = findViewById<Chip>(binding.typeChipGroup.checkedChipId).text.toString()
        val editWord = originWord?.copy(text = text, mean = mean, type = type)

        Thread {
            editWord?.let { word ->
                AppDatabase.getInstance(this)?.wordDao()?.update(word)
                val intent = Intent().putExtra("editWord", editWord)
                setResult(RESULT_OK, intent)
                runOnUiThread {
                    Toast.makeText(this, "수정을 완료했습니다.", Toast.LENGTH_SHORT).show()
                }
                finish()
            }
        }.start()
    }
}
```

AddActivity에서의 작업이 모두 완료되었다면 다시 MainActivity로 돌아와 AddActivity에서 넘겨준 값을 처리해야 한다. 값을 잘 받아왔을 경우 updateEditWord()를 호출하기로 했다. updateEditWord(word: Word)에서 wordAdapter에 저장된 기존 단어의 id 값과 수정된 단어의 id 값의 동일 여부를 파악하여 인덱스를 찾아낸다. 해당되는 인덱스의 값으로 wordAdapter의 해당 인덱스의 값을 수정된 단어로 변환하여 준다. 이후 runOnUiThread 블록 내부에서 각각의 변경 사항을 적용시켜주면 된다.

### 📖MainActivity.kt
```kotlin
class MainActivity : AppCompatActivity(), WordAdapter.ItemClickListener {
    private lateinit var binding: ActivityMainBinding
    private lateinit var wordAdapter: WordAdapter
    private var selectedWord: Word? = null
    private val updateAddWordResult = registerForActivityResult(
        ActivityResultContracts.StartActivityForResult()
    ) { result ->
        val isUpdated = result.data?.getBooleanExtra("isUpdated", false) ?: false

        if (result.resultCode == RESULT_OK && isUpdated) {
            updateAddWord()
        }
    }

    private val updateEditWordResult = registerForActivityResult(
        ActivityResultContracts.StartActivityForResult()
    ) { result ->
        val editWord = result.data?.getParcelableExtra<Word>("editWord")
        if (result.resultCode == RESULT_OK && editWord != null) {
            updateEditWord(editWord)
        }
    }

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        binding = ActivityMainBinding.inflate(layoutInflater)
        setContentView(binding.root)

        ...

        binding.editImageView.setOnClickListener {
            edit()
        }
    }

    ...

    private fun updateEditWord(word: Word) {
        val index = wordAdapter.list.indexOfFirst { it.id == word.id }
        wordAdapter.list[index] = word
        runOnUiThread {
            selectedWord = word
            wordAdapter.notifyItemChanged(index)
            binding.textTextView.text = word.text
            binding.meanTextView.text = word.mean
        }
    }

    ...

    private fun edit() {
        if(selectedWord == null) return

        val intent = Intent(this, AddActivity::class.java).putExtra("originWord", selectedWord)
        updateEditWordResult.launch(intent)
    }

    ...

}
```

![](/assets\images\fastcampus\part1\notepad/notepad14.jpg)
![](/assets\images\fastcampus\part1\notepad/notepad13.jpg)
![](/assets\images\fastcampus\part1\notepad/notepad15.jpg)
![](/assets\images\fastcampus\part1\notepad/notepad16.png)

<br>
<br>

## 📔기능 구현 - TextInputLayout Error 처리

단어장에 관한 추가, 삭제, 기능에 대한 정리는 모두 완료하였다. 마지막으로 TextInputLayout의 에러를 처리해 보자. AddActivity에서 단어를 입력하는 "textInputEditText"에 addTextChangedListener를 사용해 입력할 때마다 리스너를 호출하도록 한다. 글자 수에 따라 에러 표시를 처리할 것이므로 "textInputLayout.error"에 when 절을 사용하여 에러 표시 범위를 나타내준다.

```kotlin
binding.textInputEditText.addTextChangedListener {
    it?.let { text ->
        binding.textInputLayout.error = when (text.length) {
            0 -> "값을 입력해주세요"
            1 -> "2자 이상을 입력해주세요"
            else -> null
        }
    }
}
```

에러 처리가 잘 표시되는 것을 볼 수 있다. 

![](/assets\images\fastcampus\part1\notepad/notepad17.jpg)
![](/assets\images\fastcampus\part1\notepad/notepad18.jpg)
![](/assets\images\fastcampus\part1\notepad/notepad19.jpg)

<br>

## 📔전체 코드
<https://github.com/Becomeproo/stopwatch>

<br>

## 📔마무리
이번 앱을 구현하며 Room 데이터베이스와 RecyclerView 사용에 대해 좀 더 확실히 알고 갈 수 있었다. 이전까지는 어떤 부분에서 데이터 처리를 해야 하는지, 또 어떤 부분에서 runOnUiThread를 사용하여 UI를 업데이트해야하는지에 대해 명확히 알지 못하였다. 저번부터 정리한 스레드와 함께 이번에는 정리 후가 아닌 정리하기 앞서 room과 recyclerView에 대해 정리하면서 좀 더 명확하게 배운 내용을 구체적으로 정리할 수 있었다. 또한 이전에 사용했던 startActivityForResult가 deprecated 되며 대체된 registerForResultActivity를 사용하는 방법에 대해 알게 되었다. 개인적으로 좀 더 어려워진 거 같은데 왜 굳이.. 라는 생각을 했으나 익숙해지면 잘 사용할 수 있을 것이다. 마지막으로 recyclerview와 데이터를 연동하는 작업은 앱을 구현하는 데 있어 필수적인 사항이니 계속해서 연습해야겠다는 생각을 한다.