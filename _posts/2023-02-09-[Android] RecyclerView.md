---
title:  "[Android] RecyclerView"

categories:
  - Android
tags:
  - [Android, Android 정리, RecyclerView]

toc: true
toc_sticky: true
 
date: 2023-02-09 17:59:46+0900
last_modified_at: 2023-02-09 17:59:50+0900

---

<br>
<br>
<br>

> 아래 내용은 [Android Developer 공식 문서](https://developer.android.com/guide/topics/ui/layout/recyclerview?hl=ko) 내용을 참고하여 작성한 자습 기록입니다.

<br>
<br>

# 📚Android 정리 - RecyclerView

RecyclerView는 거의 모든 안드로이드 앱에서 볼 수 있는 필수적인 UI라고도 할 수 있다. RecyclerView는 체계적으로 데이터 컬렉션을 표시하는 데 사용되는 강력하고 유연한 UI 컴포넌트이다. ListView와 비교했을 때 RecyclerView는 더 크고 복잡한 데이터들을 쉽게 처리할 수 있다. RecyclerView에 대해서 살펴보자.

## 📔 ListView와 RecyclerView의 차이

ListView와 RecyclerView는 모두 데이터 컬렉션을 표시한다는 점에서 공통점을 가지고 있다. 하지만 두 컴포넌트 사이에는 몇 가지 차이점이 존재한다.

1. 성능: RecyclerView는 ListView의 성능적인 한계를 개선하기 위해서 고안되었다. ListView는 리스트에서 각각의 아이템들에 대한 새로운 뷰들을 모두 생성한다. 이는 다수의 데이터를 처리할 때 분명히 한계가 있다. 반면, RecyclerView는 화면에 들어갈 리스트의 크기를 계산하여 리스트에 뷰가 표시될 방법을 결정하면, 스크롤 할 때마다 해당 뷰들을 계속해서 재사용한다. 이는 다수의 데이터를 처리할 때에도 많은 부담을 덜어줄 수 있다. 
![](/assets/images/android/recyclerview/recyclerview1.drawio.png)

2. 유연성: RecyclerView는 보다 모듈화된 아키텍처를 제공하므로 ListView보다 더욱 유연하다. RecyclerView를 사용하면 커스텀 LayoutManager를 만들어 아이템이 표시되는 방식을 제어하고 커스텀 ViewHolder를 만들어 데이터를 표시할 수 있다.

3. 복잡한 데이터 표시: RecyclerView는 복잡한 데이터를 처리하는 데 더 적합하다. RecyclerView를 사용하면 데이터의 다양한 뷰 형식을 표시할 수 있으므로 복잡한 데이터를 더 쉽게 표현할 수 있다.

4. 애니메이션: RecyclerView는 애니메이션을 위한 강력한 API를 제공하여 리스트에 대한 커스텀 애니메이션을 쉽게 만들 수 있다. 반면 ListView는 애니메이션을 제한적으로 지원한다.

<br>

## 📔 RecyclerView의 기본적인 요소

RecyclerView를 구성하기 위한 기본적인 요소들은 다음과 같다.

|**요소**|설명|
|:---:|:---|
|`RecyclerView.Adapter`|ViewHolder를 생성하고 데이터에 바인딩 하는 역할을 한다|
|`ViewHolder`|ViewHolder는 화면에 뷰와 데이터를 표시하는 역할을 한다|
|`LayoutManager`|LayoutManager는 아이템들이 화면에 표시될 방법을 관리한다. LayoutManager의 종류로는 LinearLayoutManager, GridLayoutManager 등이 있고 이외에도 필요에 따라 커스텀 하여 표시될 방법을 지정할 수 있다.|

이외에도 표시될 수 있는 요소들은 다음과 같다.

|**요소**|설명|
|:---:|:---|
|`Cunstom Animation`|애니메이션을 위한 API를 활용하여 리스트들을 더 보기 좋게 표현할 수 있다|
|`Item Decoration`|리스트들을 더욱 보기 좋게 표현하기 위해 구분선과 그 외의 선들을 활용할 수 있다|
|`선택 동작`|복잡한 데이터들에 대해 각각의 선택 옵션을 지정하는 클릭 동작을 구현할 수 있다|

<br>

## 📔 RecyclerView를 구현해보기

간단하게 RecyclerView를 구현해보자. 먼저 아이템들을 표시할 공간으로 RecyclerView가 전체 화면을 차지하도록 구현해 주었다. 

### 📖 activity_main.xml

```kotlin
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">

    <androidx.recyclerview.widget.RecyclerView
        android:id="@+id/recyclerView"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        />

</androidx.constraintlayout.widget.ConstraintLayout>
```

<br>

다음으로 각각의 아이템들을 표현할 뷰를 새로 만들어 주도록 한다. 즉, 아이템의 틀을 만들 레이아웃을 생성한다. 간단하게 ImageView 하나와 TextView 두 개로 구성된 `item_list.xml` 파일을 만들어 주었다.

### 📖 item_list.xml

```kotlin
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    xmlns:tools="http://schemas.android.com/tools"
    android:padding="36dp">

    <ImageView
        android:id="@+id/userImageView"
        android:layout_width="50dp"
        android:layout_height="50dp"
        tools:src="@drawable/ic_launcher_background"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent" />


    <TextView
        android:id="@+id/nameTextView"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="아무개"
        android:textSize="18sp"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintTop_toTopOf="parent" />

    <TextView
        android:id="@+id/ageTextView"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginTop="4dp"
        android:text="12살"
        android:textSize="16sp"
        app:layout_constraintEnd_toEndOf="@id/nameTextView"
        app:layout_constraintTop_toBottomOf="@id/nameTextView" />


</androidx.constraintlayout.widget.ConstraintLayout>
```

표시될 아이템의 형식은 다음과 같다.

![](/assets/images/android/recyclerview/recyclerview2.png)


<br>

위 과정까지 구현해 주었다면, 이번에는 아이템들의 데이터를 갖고 있는 데이터 클래스를 생성한다. 개인의 이름과 나이, 그리고 이미지의 속성을 가지고 있는 `User`라는 이름의 데이터 클래스를 만들어 준다.

### 📖 User.kt

```kotlin
data class User(
    val image: Int,
    val name: String,
    val age: String,
)
```

<br>

구현한 UI와 데이터를 연결시키기 위해서는 앞서 정리했듯이 Adapter 클래스를 구현해야 한다. 따라서 UserProfileAdapter라는 이름의 클래스를 `RrecyclerView.Adapter<>()`에서 상속을 받아 생성한다. 각각의 메서드에 대한 설명과 기능은 주석에 표시해 두었다.

### 📖 UserProfileAdapter.kt

```kotlin
class UserProfileAdapter(private val users: List<User>) : RecyclerView.Adapter<UserProfileAdapter.UserProfileViewHolder>() {

    /*
    해당 메서드는 리사이클러뷰의 아이템을 위해 새로운 뷰가 생성되어야 할 때 호출된다.
    아이템을 레이아웃에 inflate 하는 역할을 수행하고, inflated 한 뷰를 유지하는 ViewHolder 객체를 반환한다.

    각각의 패러미터들에 대해 설명하면,
    * parent: ViewGroup 은 ViewHolder를 포함할 ViewGroup을 의미한다. 여기서 ViewGroup은 일반적으로 RecyclerView 자신을 의미한다.
    * viewType: Int 는 생성되어야하는 ViewHolder의 타입을 의미한다. 이 파라미터는 동일한 리사이클러뷰에서 다수의 ViewHolder 타입을 보유하고 있을 때,
     또는 다른 타입의 아이템들을 위한 또 다른 ViewHolder를 생성하고자 할 때 유용하다.

     */
    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): UserProfileViewHolder {
        val view = LayoutInflater.from(parent.context).inflate(R.layout.item_list, parent, false)
        return UserProfileViewHolder(view)
    }

    /*
    해당 메서드는 ViewHolder에 아이템의 데이터를 바인딩할 때 사용한다.

    각각의 파라미터들에 대해 설명하면,
    * holder 는 onCreateViewHolder 메서드에 의해 생성된 ViewHolder 클래스의 인스턴스이다. 해당 인스턴스는 각각의 데이터를 반영하기 위해 업데이트되어야 한다.
    * position 은 데이터 항목의 위치를 나타내는 값이다.

    onBindViewHolder 메서드는 주어진 위치에 데이터를 표시하기 위해 ViewHolder의 뷰를 실제로 업데이트하는 곳이다. 예를 들어 데이터의 텍스트와 일치하도록 TextView의 텍스트
    를 설정할 수 있다.

    onBindViewHolder 메서드는 RecyclerView가 새 항목을 표시해야 할 때마다 또는 이전에 표시된 항목이 화면 밖으로 스크롤 되고 새 항목이 스크롤될 때 호출된다는 점에 유의해야한다.
    
     */
    override fun onBindViewHolder(holder: UserProfileViewHolder, position: Int) {
        val user = users[position]
        holder.userImage.setImageResource(user.image)
        holder.userName.text = user.name
        holder.userAge.text = user.age
    }

    override fun getItemCount() = users.size

    /*
        데이터가 바인딩될 ViewHolder에 대한 클래스이다. 구현했던 뷰 위젯들을 가져온다.
    */
    class UserProfileViewHolder(itemView: View) : RecyclerView.ViewHolder(itemView) {
        val userImage: ImageView = itemView.findViewById(R.id.userImageView)
        val userName: TextView = itemView.findViewById(R.id.nameTextView)
        val userAge: TextView = itemView.findViewById(R.id.ageTextView)
    }
}
```

Adapter 클래스까지 생성해 주었다면, RecyclerView를 구현할 준비는 끝났다. 이제 MainActivity에서 RecyclerView 생성하면 된다. 먼저 RecyclerView와 Adapter 클래스를 정의해 주었고, Adapter 클래스에는 dummy 데이터를 생성해 User 데이터의 리스트를 인자로 넣어주었다. 그 후 layoutManager를 수직으로 표시해 주는 LinearLayoutManager로, adapter의 인자로는 앞서 정의해 두었던 UserProfileAdapter를 넣어주었다.

### 📖 MainActivity.kt

```kotlin
class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        val recyclerView = findViewById<RecyclerView>(R.id.recyclerView)

        val dummyUser = listOf(
            User(R.drawable.ic_launcher_background, "kim", "12"),
            User(R.drawable.ic_launcher_background, "park", "16"),
            User(R.drawable.ic_launcher_background, "choi", "11"),
            User(R.drawable.ic_launcher_background, "jeong", "21"),
            User(R.drawable.ic_launcher_background, "yoon", "34"),
            User(R.drawable.ic_launcher_background, "min", "31"),
            User(R.drawable.ic_launcher_background, "lee", "22"),
        )

        val userProfileAdapter = UserProfileAdapter(dummyUser)
        recyclerView.apply {
            layoutManager = LinearLayoutManager(applicationContext, LinearLayoutManager.VERTICAL, false)
            adapter = userProfileAdapter
        }
    }
}
```

결과 화면을 보면, 이미지와 이름, 나이를 표시해 주는 아이템들로 RecyclerView가 생성된 것을 볼 수 있다.

![](/assets/images/android/recyclerview/recyclerview3.jpg)