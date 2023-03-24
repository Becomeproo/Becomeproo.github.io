---
title:  "[Android] Part2-채팅 앱"

categories:
  - Android
tags:
  - [Android, Kotlin, 안드로이드 강의]

toc: true
toc_sticky: true
 
date: 2023-03-22 19:38:37+0900
last_modified_at: 2023-03-24 22:44:54+0900
---

<br>
<br>
<br>

> 아래 내용은 패스트 캠퍼스 [35개 프로젝트로 배우는 Android 앱 개발 feat. Jetpack Compose]의 강의 내용을 기반으로 작성한 자습 기록입니다.

<br><br>

# 채팅 앱

## ℹ️앱 설명

로그인 및 회원가입을 통해 목록에 있는 사용자와 채팅할 수 있는 앱

<br>

## ✅구현 기능

* Firebase Authnetication 기능을 활용한 회원가입 및 로그인 구현
* Firebase Realtiem Database를 활용한 채팅 구현
* FCM(Firebase Cloud Message)를 활용한 채팅 알림 기능 구현

<br>

## ✅사용되는 기능

* Android
    * Firebase Authnetication
    * Firebase Realtime Database
    * Firebase Cloud Message
    * RecyclerView
    * BottomNavigationView

---

<br><br>

Firebase의 환경 설정(패키지 등록 및 사용 기능 설정)이 모두 되어있는 상황을 가정하여 작성하였다.

## 📔 라이브러리 추가

로그인 및 회원가입을 위한 Firebase Authnetication, 실시간 채팅을 위한 Firebase Realtime Database, 메시지 알림을 위한 Firebase Cloude Message를 사용하기 위해 build.gradle에 다음과 같이 해당하는 라이브러리를 추가해 준다.

<b>app:build.gradle</b>

```kotlin

plugins {
    
    ...

    id 'com.google.gms.google-services'
}

...

dependencies {

    ...

    // Firebase
    implementation platform('com.google.firebase:firebase-bom:31.2.2')
    implementation 'com.google.firebase:firebase-auth-ktx'
    implementation 'com.google.firebase:firebase-database-ktx'
    implementation 'com.google.firebase:firebase-messaging-ktx'

    ...
}
```

## 📔 로그인 및 회원 가입 구현

Firebase Authnetication를 활용하여 로그인 및 회원가입 기능을 구현해 보자. 먼저 새로운 액티비티로 LoginActivity를 생성해 준다. 생성된 액티비티의 xml 파일은 간단하게 EditText 2개와 Button 2개로 구성하였다. EditText는 각각 이메일과 비밀번호를 입력받는 기능을 한다. 버튼 두 개는 각각 로그인과 회원가입 기능을 수행하기 위한 버튼이다. 

<b>activity_login.xml</b>

```kotlin
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".LoginActivity">

    <EditText
        android:id="@+id/emailEditText"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:layout_marginTop="100dp"
        android:hint="email"
        android:inputType="textEmailAddress"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent" />

    <EditText
        android:id="@+id/passwordEditText"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:hint="password"
        android:inputType="textPassword"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toBottomOf="@id/emailEditText" />

    <Button
        android:id="@+id/signInButton"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:layout_marginHorizontal="30dp"
        android:layout_marginTop="50dp"
        android:text="Sign In"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toBottomOf="@id/passwordEditText" />

    <Button
        android:id="@+id/signUpButton"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:text="Sign Up"
        app:layout_constraintEnd_toEndOf="@id/signInButton"
        app:layout_constraintStart_toStartOf="@id/signInButton"
        app:layout_constraintTop_toBottomOf="@id/signInButton" />

</androidx.constraintlayout.widget.ConstraintLayout>
```

LoginActivity를 구현하기 앞서 앱이 시작하면 MainActivity를 먼저 실행하므로 초기에는 아무런 계정도 추가되어 있는 상태가 아니기 때문에 MainActivity에서 LoginActivity로 이동시켜야 한다. 따라서 MainActivity에서 먼저 조건문을 통해 현재 계정에 등록되지 않은 상태인 경우 LoginActivity로 이동하도록 구현시켜 주었다.

<b>MainActivity.kt</b>

```kotlin
class MainActivity : AppCompatActivity() {
    private lateinit var binding: ActivityMainBinding

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        binding = ActivityMainBinding.inflate(layoutInflater)
        setContentView(binding.root)

        val currentUser = Firebase.auth.currentUser
        if (currentUser == null) {
            startActivity(Intent(this, LoginActivity::class.java))
            finish()
        } else {

        }
    }
}
```

위 과정에 의하면 이제 계정이 존재하지 않을 경우 LoginActivity를 실행하기 되어 앱이 실행될 때 가장 먼저 해당 액티비티가 보이는 것처럼 구현하였다. 이제 LoginActivity에서 회원가입과 로그인을 구현하면 된다. 회원가입 기능을 구현하기 위해 회원가입 버튼일 눌렸을 때 이메일과 비밀번호를 받아와 Firebase의 auth의 기능을 사용하여 정보를 넘겨준다. 이때 사용되는 메서드는 `Firebase.auth.createUserWithEmailAndPassword()`이고 해당 결과를 받는 리스너로 `addOnCompleteListener()`를 사용한다. 리스너 내부에서 회원가입 작업이 성공한 경우 이를 알리는 Toast 메시지와 반대로 실패했을 경우를 알리는 Toast 메시지를 각각 구현해 주었다.

<b>LoginActivity.kt</b>

```kotlin
class LoginActivity : AppCompatActivity() {
    private lateinit var binding: ActivityLoginBinding

    override fun onCreate(savedInstanceState: Bundle?) {
        
        ...

        binding.signUpButton.setOnClickListener {
            val email = binding.emailEditText.text.toString()
            val password = binding.passwordEditText.text.toString()

            if (email.isEmpty() || password.isEmpty()) {
                Toast.makeText(this, "이메일 또는 비밀번호가 입력되지 않았습니다.", Toast.LENGTH_SHORT).show()
                return@setOnClickListener
            }

            Firebase.auth.createUserWithEmailAndPassword(email, password).addOnCompleteListener(this) { task ->
                if (task.isSuccessful) {
                    Toast.makeText(this, "회원가입에 성공했습니다.", Toast.LENGTH_SHORT).show()
                } else {
                    Toast.makeText(this, "회원가입에 실패했습니다.", Toast.LENGTH_SHORT).show()
                }
            }
        }
    }
}
```

회원가입이 성공적으로 완료되었다면 로그인을 통해 다시 MainActivity로 이동시키도록 하자. 로그인에서도 마찬가지로 이메일과 비밀번호 값을 받아 서버로 넘겨주는데 이때 사용하는 메서드는 `Firebase.auth.signInWithEmailAndPassword()`이다. 이 메서드 또한 `addOnCompleteListener()` 리스너로 받아 작업이 실패했을 경우에는 Toast 메시지를 띄우도록, 성공했을 경우에는 MainActivity로 이동하고 해당 액티비티를 종료하도록 구현해 주었다.

<b>LoginActivity.kt</b>

```kotlin
class LoginActivity : AppCompatActivity() {
    private lateinit var binding: ActivityLoginBinding

    override fun onCreate(savedInstanceState: Bundle?) {
        
        ...

        binding.signInButton.setOnClickListener {
            val email = binding.emailEditText.text.toString()
            val password = binding.passwordEditText.text.toString()

            if (email.isEmpty() || password.isEmpty()) {
                Toast.makeText(this, "이메일 또는 비밀번호가 입력되지 않았습니다.", Toast.LENGTH_SHORT).show()
                return@setOnClickListener
            }

            Firebase.auth.signInWithEmailAndPassword(email, password).addOnCompleteListener(this) { task ->
                if (task.isSuccessful) {
                    startActivity(Intent(this, MainActivity::class.java))
                    finish()
                } else {
                    Log.e("LoginActivity", task.exception.toString())
                    Toast.makeText(this, "로그인에 실패했습니다.", Toast.LENGTH_SHORT).show()
                }
            }
        }

        ...

    }
}
```

실행 결과를 보면 회원가입 기능이 잘 구현되고 로그인 버튼을 눌렀을 때 MainActivity로 이동한 것을 볼 수 있다. 또한 Firebase 서버 상에도 잘 등록된 것을 볼 수 있다.

![](/assets/images/fastcampus/part2/chat/chat1.jpg)
![](/assets/images/fastcampus/part2/chat/chat2.jpg)
![](/assets/images/fastcampus/part2/chat/chat3.png)

<br><br>

## 📔 사용자 목록 화면 UI 및 하단 메뉴 탭 구현

채팅 앱은 총 3개의 메뉴(사용자 리스트, 채팅, 마이 페이지)를 가지고 해당 메뉴들을 하단 탭으로 구성하여 구현할 것이다. 이를 위해서 BottomNavigationView를 사용한다.

3개의 메뉴로 나뉘기 때문에 기능에 맞게 3개의 패키지 파일을 추가해 준다. 패키지 파일의 이름은 각각 "userlist", "chatlist", "mypage"로 명명했다. 그리고 3개의 패키지마다 해당하는 Fragment 파일을 생성해 주었고 이에 따라 xml 파일 또한 생성해 주었다.

우선 사용자 목록 화면을 먼저 구성할 것이다. 해당 화면은 사용자 리스트를 보여주므로 RecyclerView에 아이템 뷰를 적용하여 구현할 것이다. 따라서 `fragment_userlist.xml` 파일을 RecyclerView로 채워주고 `item_user.xml` 파일을 새로 생성해 주었다.

<b>fragment_user.xml</b>

```kotlin
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <androidx.recyclerview.widget.RecyclerView
        android:id="@+id/userListRecyclerView"
        android:layout_width="match_parent"
        android:layout_height="match_parent" />

</androidx.constraintlayout.widget.ConstraintLayout>
```

사용자 리스트를 표시하는 아이템 뷰는 사용자 이미지를 표시하는 ImageView와 사용자 이름, 사용자 상태 메시지를 표시하도록 TextView로 구성해 주었다. 내용은 아래와 같다.

<b>item_user.xml</b>

```kotlin
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:padding="16dp">

    <ImageView
        android:id="@+id/profileImageView"
        android:layout_width="30dp"
        android:layout_height="30dp"
        android:src="@drawable/ic_baseline_supervised_user_circle_24"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent" />

    <TextView
        android:id="@+id/nicknameTextView"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:layout_marginStart="8dp"
        android:textColor="@color/black"
        android:textSize="18sp"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toEndOf="@id/profileImageView"
        app:layout_constraintTop_toTopOf="parent"
        tools:text="사용자 이름" />

    <TextView
        android:id="@+id/stateMessageTextView"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:layout_marginStart="8dp"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toEndOf="@id/profileImageView"
        app:layout_constraintTop_toBottomOf="@id/nicknameTextView"
        tools:text="상태 메시지" />

</androidx.constraintlayout.widget.ConstraintLayout>
```

![](/assets/images/fastcampus/part2/chat/chat4.png)

사용자 리스트 화면 외의 2개 화면은 나중에 구현할 것이므로 간단하게 배경색을 지정해 구분해 주었다.

이제 `activity_main.xml` 파일에서 BottomNavigationView를 생성해 주고 해당 뷰의 영역을 제외한 나머지 부분을 전부 FrameLayout으로 지정한다. 

<b>activity_main.xml</b>

```kotlin
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">

    <FrameLayout
        android:id="@+id/frameLayout"
        android:layout_width="0dp"
        android:layout_height="0dp"
        app:layout_constraintBottom_toTopOf="@id/bottomNavigationView"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent" />

    <com.google.android.material.bottomnavigation.BottomNavigationView
        android:id="@+id/bottomNavigationView"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:menu="@menu/bottom_navigation_menu" />

</androidx.constraintlayout.widget.ConstraintLayout>
```

하단의 아이템을 표시하기 위해서 res 파일에 menu 폴더를 추가한 뒤 해당 menu 폴더에 `bottom_navigation_menu.xml` 파일을 만들어 준다. 해당 파일에서 3개의 메뉴의 id와 title, icon을 지정한다. 

<b>bottom_navigation_menu.xml</b>

```kotlin
<?xml version="1.0" encoding="utf-8"?>
<menu xmlns:android="http://schemas.android.com/apk/res/android">
    <item
        android:id="@+id/userList"
        android:icon="@drawable/ic_baseline_supervised_user_circle_24"
        android:title="유저리스트" />

    <item
        android:id="@+id/chatList"
        android:icon="@drawable/ic_baseline_chat_bubble_24"
        android:title="채팅" />

    <item
        android:id="@+id/myPage"
        android:icon="@drawable/ic_baseline_person_pin_24"
        android:title="내 정보" />

</menu>
```

다시 `activity_main.xml` 파일로 돌아와 BottomNavigationView에 `app:menu="@menu/bottom_navigation_menu"`을 추가해 주면 메뉴 아이템이 적용된 것을 볼 수 있다.

![](/assets/images/fastcampus/part2/chat/chat5.png)

다음으로 `MainActivity.kt` 파일에서 메뉴 아이템이 선택되었을 때 해당하는 Fragment로 이동하도록 한다. 이때 사용되는 리스너가 BottomNavigationView의 `setOnItemSelectedListener`이다. 리스너에서 MenuItem을 넘겨주므로 MenuItem의 itemId를 받아 분기문으로 만들어 준 뒤, 각각의 메뉴 아이템의 id에 맞게 해당하는 Fragment로 이동시켜 준다. 이때 Fragment가 메뉴가 바뀔 때마다 계속해서 새로 생성되는 것은 비효율적이므로 클래스 변수로 미리 만들어 준다. 또한 Fragment를 이동시키는 방법은 같으므로 `navigateFragment()` 메서드로 만들어 해당 메서드를 사용한다. 

<b>MainActivity.kt</b>

```kotlin
class MainActivity : AppCompatActivity() {

    ...

    private val userFragment = UserFragment()
    private val chatListFragment = ChatListFragment()
    private val myPageFragment = MyPageFragment()

    override fun onCreate(savedInstanceState: Bundle?) {
        
        ...

        binding.bottomNavigationView.setOnItemSelectedListener {
            when (it.itemId) {
                R.id.userList -> {
                    navigateFragment(userFragment)

                    return@setOnItemSelectedListener true
                }
                R.id.chatList -> {
                    navigateFragment(chatListFragment)

                    return@setOnItemSelectedListener true
                }
                R.id.myPage -> {
                    navigateFragment(myPageFragment)

                    return@setOnItemSelectedListener true
                }
                else -> {
                    return@setOnItemSelectedListener false
                }
            }
        }
    }

    private fun navigateFragment(fragment: Fragment) {
        supportFragmentManager.beginTransaction().apply {
            replace(R.id.frameLayout, fragment)
            commit()
        }
    }
}
```

<br><br>

다음으로 유저리스트 화면을 구현하기 위해서 "userlist" 패키지 내부에 UserAdapter와 UserItem 파일을 추가한다. UserItem 파일은 Firebase의 DB의 데이터를 받아와 저장될 예정이다. 유저 리스트 데이터의 값으로는 사용자의 Id와 사용자 이름, 상태 메시지가 표시될 것이다.

<b>UserItem.kt</b>

```kotlin
data class UserItem(
    val userId: String? = null,
    val username: String? = null,
    val stateMessage: String? = null
)
```

UserAdapter 파일은 항상 구현했듯이 ListAdapter를 상속받아 구현해 준다. 

<b>UserAdapter.kt</b>

```kotlin
class UserAdapter : ListAdapter<UserItem, UserAdapter.UserViewHolder>(diffUtil) {

    inner class UserViewHolder(private val binding: ItemUserBinding) : RecyclerView.ViewHolder(binding.root) {
        fun bind(item: UserItem) {
            binding.nicknameTextView.text = item.username
            binding.stateMessageTextView.text = item.stateMessage
        }
    }

    companion object {
        val diffUtil = object : DiffUtil.ItemCallback<UserItem>() {
            override fun areItemsTheSame(oldItem: UserItem, newItem: UserItem): Boolean {
                return oldItem.userId == newItem.userId
            }

            override fun areContentsTheSame(oldItem: UserItem, newItem: UserItem): Boolean {
                return oldItem == newItem
            }

        }
    }

    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): UserViewHolder {
        return UserViewHolder(
            ItemUserBinding.inflate(
                LayoutInflater.from(parent.context),
                parent,
                false
            )
        )
    }

    override fun onBindViewHolder(holder: UserViewHolder, position: Int) {
        holder.bind(currentList[position])
    }
}
```

`UserFragment.kt` 파일에서 "userListRecyclerView"의 adapter와 layoutManager를 지정해 주고, 임의의 데이터를 넣어 화면에 잘 구현되는지 확인해 준다. 

<b>UserFragment.kt</b>

```kotlin
class UserFragment : Fragment(R.layout.fragment_userlist){
    private lateinit var binding: FragmentUserlistBinding

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)
        binding = FragmentUserlistBinding.bind(view)

        val userListAdapter = UserAdapter()
        binding.userListRecyclerView.apply {
            layoutManager = LinearLayoutManager(context)
            adapter = userListAdapter
        }

        userListAdapter.submitList(mutableListOf(
            UserItem("asdiofjaiojiowejfawe", "KimKim", "Hello!")
        ))
    }
}
```

![](/assets/images/fastcampus/part2/chat/chat6.jpg)
![](/assets/images/fastcampus/part2/chat/chat7.jpg)
![](/assets/images/fastcampus/part2/chat/chat8.jpg)

<br><br>

## 📔 채팅방 목록 페이지 구현

채팅방 목록을 구현하는 방법은 크게 어렵지 않다. 이제는 RecyclerView를 만들어 아이템 뷰와 Adapter를 연결시키는 일을 많이 했기 때문에 똑같이 하면 된다. 먼저 채팅방을 나타내는 xml 파일인 `fragment_chatlist.xml` 파일에서 앞서 설정해 두었던 배경색을 제거하고 RecyclerView가 화면을 채우도록 지정한다. 

<b>fragment_chatlist.xml</b>

```kotlin
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <androidx.recyclerview.widget.RecyclerView
        android:id="@+id/chatListRecyclerView"
        android:layout_width="match_parent"
        android:layout_height="match_parent" />

</androidx.constraintlayout.widget.ConstraintLayout>
```

채팅방을 구성하는 아이템 뷰 또한 생성해 주었다. 내부 요소로 유저 리스트를 보여주는 아이템 뷰와 크게 다르지 않다. 다만 상태 메시지 부분이 채팅방의 마지막 메시지를 나타내도록 변경 해주었다.

<b>item_chatroom.xml</b>

```kotlin
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:padding="16dp">

    <ImageView
        android:id="@+id/profileImageView"
        android:layout_width="50dp"
        android:layout_height="50dp"
        android:src="@drawable/ic_baseline_supervised_user_circle_24"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent" />

    <TextView
        android:id="@+id/nicknameTextView"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:layout_marginStart="8dp"
        android:textColor="@color/black"
        android:textSize="18sp"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toEndOf="@id/profileImageView"
        app:layout_constraintTop_toTopOf="parent"
        tools:text="사용자 이름" />

    <TextView
        android:id="@+id/lastMessageTextView"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:layout_marginStart="8dp"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toEndOf="@id/profileImageView"
        app:layout_constraintTop_toBottomOf="@id/nicknameTextView"
        tools:text="마지막 메시지" />


</androidx.constraintlayout.widget.ConstraintLayout>
```

![](/assets/images/fastcampus/part2/chat/chat9.png)


다음으로 채팅방을 구성하는 데이터를 나타내는 데이터 클래스를 만들어 주었다. 채팅방 데이터 클래스의 요소는 채팅방 id, 상대 사용자 이름, 마지막 메시지로 구성된다.

<b>ChatRoomItem.kt</b>

```kotlin
data class ChatRoomItem(
    val chatRoomId: String? = null,
    val otherUserName: String? = null,
    val lastMessage: String? = null,
)
```

Adapter 클래스 또한 UserAdapter와 유사한 모양으로 생성한다.

<b>ChatListAdapter.kt</b>

```kotlin
class ChatListAdapter : ListAdapter<ChatRoomItem, ChatListAdapter.ChatListViewHolder>(diffUtil) {

    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): ChatListViewHolder {
        return ChatListViewHolder(
            ItemChatroomBinding.inflate(
                LayoutInflater.from(parent.context),
                parent,
                false
            )
        )
    }

    override fun onBindViewHolder(holder: ChatListViewHolder, position: Int) {
        holder.bind(currentList[position])
    }

    inner class ChatListViewHolder(private val binding: ItemChatroomBinding) : RecyclerView.ViewHolder(binding.root) {
        fun bind(item: ChatRoomItem) {
            binding.nicknameTextView.text = item.otherUserName
            binding.lastMessageTextView.text = item.lastMessage
        }
    }

    companion object {
        val diffUtil = object : DiffUtil.ItemCallback<ChatRoomItem>() {
            override fun areItemsTheSame(oldItem: ChatRoomItem, newItem: ChatRoomItem): Boolean {
                return oldItem.chatRoomId == newItem.chatRoomId
            }

            override fun areContentsTheSame(oldItem: ChatRoomItem, newItem: ChatRoomItem): Boolean {
                return oldItem == newItem
            }

        }
    }
}
```

마지막으로 ChatListFragment에서 RecyclerView를 지정해 LinearLayoutManager와 Adapter를 연결해 주면 된다.

<b>ChatListFragment.kt</b>

```kotlin
class ChatListFragment : Fragment(R.layout.fragment_chatlist) {
    private lateinit var binding: FragmentChatlistBinding

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)
        binding = FragmentChatlistBinding.bind(view)

        val chatListAdapter = ChatListAdapter()

        binding.chatListRecyclerView.apply {
            adapter = chatListAdapter
            layoutManager = LinearLayoutManager(context)
        }

        chatListAdapter.submitList(mutableListOf(
            ChatRoomItem("asdifojwe", "김아무개", "안녕하세요")
        ))
    }
}

```

![](/assets/images/fastcampus/part2/chat/chat10.jpg)

<br><br>

## 📔 마이 페이지 UI 및 로그아웃 구현

마이 페이지는 리스트 형식으로 구성되지 않기 때문에 따로 Adapter 클래스와 데이터 클래스가 필요하지 않다. 따라서 사용자의 이름과 상태 메시지 설정 수정 적용 버튼, 로그아웃 버튼으로 구성된 UI를 만들어 주었다.

<b>fragment_mypage.xml</b>

```kotlin
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".LoginActivity">

    <EditText
        android:id="@+id/usernameEditText"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:layout_marginTop="100dp"
        android:hint="username"
        android:inputType="text"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent" />

    <EditText
        android:id="@+id/stateMessageEditText"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:hint="상태 메시지"
        android:inputType="text"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toBottomOf="@id/usernameEditText" />

    <Button
        android:id="@+id/applyButton"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:layout_marginHorizontal="30dp"
        android:layout_marginTop="50dp"
        android:text="변경사항 적용"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toBottomOf="@id/stateMessageEditText" />

    <Button
        android:id="@+id/signOutButton"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:text="로그아웃"
        app:layout_constraintEnd_toEndOf="@id/applyButton"
        app:layout_constraintStart_toStartOf="@id/applyButton"
        app:layout_constraintTop_toBottomOf="@id/applyButton" />

</androidx.constraintlayout.widget.ConstraintLayout>
```

MyPageFragment에서 우선 "applyButton"의 클릭 리스너 내부에 사용자 이름과 상태 메시지를 EditText로 받도록 하고, 사용자 이름이 널 값이면 return 하도록 한다. "signOutButton"이 클릭 되었을 때에는 `Firebase.auth.signOut()`을 통해 서버에 로그아웃을 수행함을 알려주고 로그아웃 처리 이후 LoginActivity에서 다시 로그인 및 회원가입을 수행하도록 한다.

<b>MyPageFragment.kt</b>

```kotlin
class MyPageFragment : Fragment(R.layout.fragment_mypage) {
    private lateinit var binding: FragmentMypageBinding

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)
        binding = FragmentMypageBinding.bind(view)

        binding.applyButton.setOnClickListener {
            val username = binding.usernameEditText.text.toString()
            val stateMessage = binding.stateMessageEditText.text.toString()

            if (username.isEmpty()){
                Toast.makeText(context, "사용자 이름을 입력해주세요.", Toast.LENGTH_SHORT).show()
                return@setOnClickListener
            }
        }

        binding.signOutButton.setOnClickListener {
            Firebase.auth.signOut()
            startActivity(Intent(context, LoginActivity::class.java))
            activity?.finish()
        }
    }
}
```

결과 화면을 보면 로그아웃 버튼을 누르면 로그아웃이 진행되고 로그인 화면으로 이동하는 것을 볼 수 있다.

![](/assets/images/fastcampus/part2/chat/chat11.jpg)
![](/assets/images/fastcampus/part2/chat/chat12.jpg)

<br><br>

## 📔 데이터베이스 설계

Firebase DB에 사용자 정보 업데이트 또는 채팅 기능을 구현하기 앞서 해당 과정들을 위해서는 데이터베이스를 어떤식으로 구성할 지에 대한 설계가 필요하다. 어떤식으로 설계를 해도 상관없지만 되도록 효율적이고 비용이 덜 소모되는 방향으로 구성하는 것이 바람직하다. 이번 앱에서의 DB 구조는 다음과 같다.

<b>chatDB</b>

```json
{
    "User" : { // 사용자 정보
        "userId1" : { // 사용자 id를 상위로 두어 해당 사용자의 조회를 쉽게 구성
            "userId" : "userId1", // 사용자 id, 조회를 더욱 쉽게 하기 위해 하위에도 사용자 id를 받아옴
            "username" : "aaaa", // 사용자 이름
            "stateMessage" : "messaage1" // 상태 메시지
        },
        "userId2" : { // 사용자 id2
            "userId" : "userId2", 
            "unsername" : "bbbb",
            "stateMessage" : "message2"
        },
        "userId3" : {
            "userId" : "userId3",
            "unsername" : "cccc",
            "stateMessage" : "message3"
        },
        "userId4" : {
            "userId" : "userId4",
            "unsername" : "dddd",
            "stateMessage" : "message4"
        }

        ...

    },
    "ChatRoom" : { // 채팅방 정보, 사용자 id를 사용해 해당 사용자가 속한 채팅방 정보 조회
        "userId1" :{ // 사용자 id1을 키로 하여 해당 사용자가 속한 채팅방 조회
            "userId2" : { // 사용자1이 속한 채팅방1
                "chatRoomId" : "chatRoomId1", // 채팅방 id
                "lastMessage" : "lastMessage", // 해당 채팅방의 마지막 메시지
                "otherUserId" : "userId2", // 채팅 상대의 id
                "otherUserName" : "bbbb" // 채팅 상대의 이름
            }

            ...

        },
        "userId2" : { 
            "userId1" : {
                "chatRoomId" : "chatRoomId1",
                "lastMessage" : "lastMessage",
                "otherUserId" : "userId1",
                "otherUserName" : "aaaa"
            }

            ...

        }

        ...

    },
    "Chat" : { // 채팅
        "chatRoomId1" : { // 채팅방 id1의 메시지 조회
            "messageId1" : { // 메시지1
                "chatId" : "...", // 해당 메시지의 id
                "message" : "...", // 메시지 내용
                "userId" : "..." // 입력한 사용자 id
            },
            "messageId2" : { // 메시지2
                "chatId" : "...",
                "message" : "...",
                "userId" : "..."
            },
            "messageId3" : {
                "chatId" : "...",
                "message" : "...",
                "userId" : "..."
            }

            ...

        }

        ...

    }
}
```

<br><br>

## 📔 실시간 DB에 사용자 정보 추가 및 유저 리스트에 다른 사용자 표시 

위에서 설계한 DB의 구성을 토대로 사용자 정보를 먼저 추가해 보자. 사용자가 추가되는 부분은 로그인이 성공적으로 기능했을 때 추가하도록 한다. 따라서 LoginActivity에서 로그인이 성공하고 `addOnCompleteListener(this)` 리스너를 호출할 때의 블록에서 사용자 정보를 추가한다. 먼저 현재 사용자가 정보를 불러와 널 체크를 먼저 해주고 Map 객체를 만들어 사용자 정보의 키와 값을 지정해 준다. 사용자 정보의 항목들은 위에서 설정한 바와 같이 사용자 ID, 사용자 이름을 지정해 주도록 한다. 사용자 상태 메시지는 로그인할 때에는 요구되지 않으므로 마이 페이지의 변경사항 수정 버튼이 눌렸을 때 추가해 주도록 한다. 이는 처음 데이터 클래스를 생성할 때 각 속성의 값들을 null로 지정해 주었기에 가능하다. 또한 Firebase 서버에서 값을 받아올 때 빈 생성자를 호출해 데이터를 전달하므로 Firebase 문서에서도 [다음 내용과 같이](https://firebase.google.com/docs/database/android/read-and-write?hl=ko#write_data) 권고된다. 다시 본론으로 돌아와 MutableMap 자료형의 구조로 데이터를 추가해 준 뒤, `Firebase.database.reference.child(DB_USERS).child(userId).updateChildren(user)`의 경로로 데이터를 추가해 준다. 여기서 `updateChildren()`이 데이터의 추가 및 업데이트를 의미한다.

<b>LoginActivity.kt</b>

```kotlin
class LoginActivity : AppCompatActivity() {

    ...

    override fun onCreate(savedInstanceState: Bundle?) {
        
        ...

        binding.signInButton.setOnClickListener {
            
            ...

            Firebase.auth.signInWithEmailAndPassword(email, password).addOnCompleteListener(this) { task ->
                val currentUser = Firebase.auth.currentUser
                if (task.isSuccessful && currentUser != null) {
                    val userId = currentUser.uid

                    val user = mutableMapOf<String, Any>()
                    user["userId"] = userId
                    user["username"] = email

                    Firebase.database.reference.child(DB_USERS).child(userId).updateChildren(user)

                    startActivity(Intent(this, MainActivity::class.java))
                    finish()
                } else {
                    Log.e("LoginActivity", task.exception.toString())
                    Toast.makeText(this, "로그인에 실패했습니다.", Toast.LENGTH_SHORT).show()
                }
            }
        }

        ...

    }
}
```

위에서 DB에 접근하는 각각의 키는 자주 사용될 것이므로 Key 클래스를 만들어 따로 추가해 주었다.

<b>Key.kt</b>

```kotlin
class Key {
    companion object {
        const val DB_USERS = "Users"
        const val DB_CHAT_ROOMS = "ChatRooms"
        const val DB_CHATS = "Chats"
    }
}
```

앱을 실행하여 새로운 계정으로 회원가입을 한 뒤 로그인을 진행해 보면, Firebase 실시간 DB 서버에 위에서 설계된 데이터베이스와 같은 모양으로 데이터가 추가된 것을 볼 수 있다.

![](/assets/images/fastcampus/part2/chat/chat13.png)

위와 같이 사용자 정보가 성공적으로 서버에 추가되었으므로 사용자 리스트 화면에 다른 사용자의 정보를 표시해 보도록 하자. 

UserFragment에서 `Firebase.database.reference.child(DB_USERS)`를 통해 DB의 사용자 정보에 접근한 뒤 `addListenerForSingleValueEvent()`를 활용해 데이터를 가져오도록 한다. 여기서 위의 리스너는 실시간으로 데이터가 변경되거나 추가될 때마다 해당 정보를 불러오는 것이 아닌 리스너가 호출되는 한번만 호출된다. 사용자 정보를 표시하는 것은 사용자 리스트 화면에 접근하고 나서 한 번만 불리는 것이 효율적이므로 해당 리스너를 사용하였다.

`addListenerForSingleValueEvent()` 리스너는 `onDataChange()`와 `onCancelled()` 메서드를 재정의한다. 말 그대로 데이터가 변경됨을 인지했을 때와 데이터를 불러오는 것을 실패했을 때 호출한다. 데이터 변경이 인지되었을 때에는 snapshot을 가져오는데 `snapshot.children`을 사용해 데이터의 내용들을 불러올 수 있다. 하나의 children은 위에서 설정한대로 `Users` 하위의 사용자 id를 의미한다. 여기서 children을 forEach 함수를 사용해 각각의 사용자를 UserItem 클래스에 매핑한다. 이 과정에서 현재 사용자의 정보를 표시할 수도 있지만 해당 앱에서는 표시하지 않으므로 다른 사용자만 표시하도록 조건문을 설정해 준다. 데이터를 불러오는 것이 완료되었다면 `userListAdapter.submitList(userList)`를 통해 뷰에 표시한다.

<b>UserFragment.kt</b>

```kotlin
class UserFragment : Fragment(R.layout.fragment_userlist){

    ...

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        
        ...

        val currentUserId = Firebase.auth.currentUser?.uid ?: ""

        Firebase.database.reference.child(DB_USERS)
            .addListenerForSingleValueEvent(object : ValueEventListener {
            override fun onDataChange(snapshot: DataSnapshot) {
                val userList = mutableListOf<UserItem>()
                snapshot.children.forEach {
                    val user = it.getValue(UserItem::class.java)
                    user ?: return

                    if (user.userId != currentUserId) {
                        userList.add(user)
                    }
                }

                userListAdapter.submitList(userList)
            }

            override fun onCancelled(error: DatabaseError) {

            }

        })

    }
}
```

결과를 보면, 사용저 정보 리스트에 다른 사용자만 표시된 것을 볼 수 있다.

![](/assets/images/fastcampus/part2/chat/chat14.jpg)

<br><br>

## 📔 Firebase Realtime Database의 정보를 채팅방 리스트 화면에 표시

현재 사용자의 채팅방 리스트를 가져오기 위해서는 "ChatRooms"의 "사용자 id"로 접근해야 한다. 따라서 ChatListFragment에서 현재 사용자의 id를 받아온 후 `Firebase.database.reference.child(DB_CHAT_ROOMS).child(currentUserId)`을 통하면 위와 같은 경로로 접근할 수 있다. 이 DB 레퍼런스를 변수로 따로 만든 뒤, `addValueEventListener()`를 사용해 데이터를 가져온다. 해당 리스너는 접근한 경로의 데이터가 변경될 때마다 호출된다. 마찬가지로 `onDataChanged()` 메서드를 재정의하므로 각각의 데이터를 ChatRoomItem에 매핑시켜 주도록 한다. 

<b>ChatListFragment.kt</b>

```kotlin
class ChatListFragment : Fragment(R.layout.fragment_chatlist) {

    ...

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        
        ...

        val currentUserId = Firebase.auth.uid ?: return
        val chatRoomDB = Firebase.database.reference.child(DB_CHAT_ROOMS).child(currentUserId)
        chatRoomDB.addValueEventListener(object : ValueEventListener {
            override fun onDataChange(snapshot: DataSnapshot) {
                val chatRoomList = snapshot.children.map {
                    it.getValue(ChatRoomItem::class.java)
                }

                chatListAdapter.submitList(chatRoomList)
            }

            override fun onCancelled(error: DatabaseError) {
                TODO("Not yet implemented")
            }

        })
    }
}
```

## 📔 Firebase Realtime Database에 사용자 정보 업데이트 및 데이터 가져오기

다음으로 마이 페이지에서 데이터를 읽고 써보도록 한다. MyPageFragment에서 "현재 사용자의 id"와 "현재 사용자의 정보를 갖고 있는 DB 레퍼런스"를 가져온다. 가져온 DB 레퍼런스인 "currentUserDB"의 `currentUserDB.get().addOnSuccessListener`를 통해 정보를 가져온다. 해당 리스너 또한 `addListenerForSingleValueEvent` 리스너와 마찬가지로 데이터를 한번만 가져온다. 해당 리스너 내부에서 현재 사용자의 정보를 가져와 사용자 이름과 상태 메시지를 표시하는 EditText에 값을 적용한다. 

<b>MyPageFragment.kt</b>

```kotlin
class MyPageFragment : Fragment(R.layout.fragment_mypage) {

    ...

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        
        ...

        val currentUserId = Firebase.auth.currentUser?.uid ?: ""
        val currentUserDB = Firebase.database.reference.child(DB_USERS).child(currentUserId)

        currentUserDB.get().addOnSuccessListener {
            val currentUserItem = it.getValue(UserItem::class.java) ?: return@addOnSuccessListener
            binding.usernameEditText.setText(currentUserItem.username)
            binding.stateMessageEditText.setText(currentUserItem.stateMessage)
        }

        ...

    }
}
```

데이터를 가져오는 것 뿐만 아니라 업데이트도 해줘야하므로 "applyButton"이 클릭되었을 때 사용자 정보를 MutableMap 자료형으로 만들어 데이터를 업데이트 해주도록 한다.

<b>MyPageFragment.kt</b>

```kotlin
class MyPageFragment : Fragment(R.layout.fragment_mypage) {

    ...

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        
        ...

        binding.applyButton.setOnClickListener {
            val username = binding.usernameEditText.text.toString()
            val stateMessage = binding.stateMessageEditText.text.toString()

            if (username.isEmpty()) {
                Toast.makeText(context, "사용자 이름을 입력해주세요.", Toast.LENGTH_SHORT).show()
                return@setOnClickListener
            }

            val user = mutableMapOf<String, Any>()
            user["username"] = username
            user["stateMessage"] = stateMessage
            currentUserDB.updateChildren(user)
        }

        ...

    }
}
```

결과를 보면 사용자 정보가 업데이트된 것을 볼 수 있다.

![](/assets/images/fastcampus/part2/chat/chat16.jpg)
![](/assets/images/fastcampus/part2/chat/chat15.png)

<br><br>

## 📔 채팅방 화면 구현

상대방과 대화를 주고받을 수 있는 채팅방 페이지를 구현해 보자. 먼저 "chatdetail"이라는 이름의 패키지 파일을 하나 생성해 준다. 그리고 ChatActivity라는 이름으로 새로운 액티비티를 추가한다. 

우선 액티비티를 새로 만들었으니 xml 파일을 통해 페이지 구성을 구현한다. `activity_chat.xml` 파일로 가서 채팅 아이템을 표시하기 위한 RecyclerView와 메시지를 입력할 EditText, 전송을 위한 Button으로 구성한다. 

<b>activity_chat.xml</b>

```kotlin
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".chatdetail.ChatActivity">

    <androidx.recyclerview.widget.RecyclerView
        android:id="@+id/chatRecyclerView"
        android:layout_width="0dp"
        android:layout_height="0dp"
        app:layout_constraintBottom_toTopOf="@id/messageBoxLayout"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent" />

    <androidx.appcompat.widget.LinearLayoutCompat
        android:id="@+id/messageBoxLayout"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:background="@color/purple_200"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent">

        <EditText
            android:id="@+id/messageEditText"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="1" />

        <Button
            android:id="@+id/sendButton"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="전송" />

    </androidx.appcompat.widget.LinearLayoutCompat>

</androidx.constraintlayout.widget.ConstraintLayout>
```

메시지를 표시할 아이템 뷰는 간단하게 사용자 이름과 메시지를 표시하도록 TextView 2개로 구성한다. 

<b>item_chat.xml</b>

```kotlin
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:padding="12dp">

    <TextView
        android:id="@+id/usernameTextView"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:textSize="13sp"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent"
        tools:text="닉네임" />

    <TextView
        android:id="@+id/messageTextView"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:layout_marginTop="4dp"
        android:textColor="@color/black"
        android:textSize="16sp"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toBottomOf="@id/usernameTextView"
        tools:text="메시지" />

</androidx.constraintlayout.widget.ConstraintLayout>
```

표시될 채팅 화면의 모습은 다음과 같다.

![](/assets/images/fastcampus/part2/chat/chat17.png)

<br><br>

채팅 화면은 RecyclerView로 표시될 것이기 때문에 데이터 클래스와 Adapter 클래스를 생성해 준다. 

우선 ChatItem이라는 이름으로 데이터 클래스로 생성해 주고 해당 클래스는 Firebase DB의 "Chats"와 연동하기 때문에 채팅 id, 메시지 입력자 id, 메시지를 받는다. 따라서 3개의 속성으로 데이터 클래스를 생성해 준다.

<b>ChatItem.kt</b>

```kotlin
data class ChatItem(
    val chatId: String? = null,
    val userId: String? = null,
    val message: String? = null,
)
```

Adapter 클래스는 이전과 동일하게 구현하면 되긴 하나 채팅방을 구현하는 과정에서 상대방이 입력한 내용과는 구분되어야 하므로 속성으로 "otherUserItem"을 받아준다. 이 속성을 활용해서 inner class인 ViewHolder 클래스의 bind 메서드 내부에서 메시지의 입력한 사용자 id값과 상대방 id 값을 비교해 채팅방을 구분해서 구성해 주도록한다.

<b>ChatAdapter.kt</b>

```kotlin
class ChatAdapter : ListAdapter<ChatItem, ChatAdapter.ChatViewHolder>(diffUtil) {

    var otherUserItem: UserItem? = null

    companion object {
        val diffUtil = object : DiffUtil.ItemCallback<ChatItem>() {
            override fun areItemsTheSame(oldItem: ChatItem, newItem: ChatItem): Boolean {
                return oldItem.chatId == newItem.chatId
            }

            override fun areContentsTheSame(oldItem: ChatItem, newItem: ChatItem): Boolean {
                return oldItem == newItem
            }

        }
    }

    inner class ChatViewHolder(private val binding: ItemChatBinding) : RecyclerView.ViewHolder(binding.root) {
        fun bind(item: ChatItem) {
            if (item.userId == otherUserItem?.userId) {
                binding.usernameTextView.text = otherUserItem?.username
                binding.usernameTextView.isVisible = true
                binding.messageTextView.text = item.message
                binding.messageTextView.gravity = Gravity.START
            } else {
                binding.usernameTextView.isVisible = false
                binding.messageTextView.text = item.message
                binding.messageTextView.gravity = Gravity.END
            }
        }
    }

    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): ChatViewHolder {
        return ChatViewHolder(
            ItemChatBinding.inflate(
                LayoutInflater.from(parent.context),
                parent,
                false
            )
        )
    }

    override fun onBindViewHolder(holder: ChatViewHolder, position: Int) {
        holder.bind(currentList[position])
    }
}
```

이제 ChatActivity에서 Firebase DB의 "Chats" 레퍼런스에 접근하기 위해서는 UserFragment 또는 ChatListFragment에서 넘어오는 값을 받아와서 적용해야 한다. 이외에도 다른 사용자의 Id 또한 Intent로 넘어오는 값을 받아 사용하기 때문에 클래스 변수로 선언해 주고 `onCreate()`에서 이 값들을 받아준다. 해당 과정에서 현재 사용자의 id 값도 같이 받아준다.

<b>ChatActivity.kt</b>

```kotlin
class ChatActivity : AppCompatActivity() {
    private lateinit var binding: ActivityChatBinding

    private var chatRoomId: String = ""
    private var otherUserId: String = ""
    private var myUserId: String = ""

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        binding = ActivityChatBinding.inflate(layoutInflater)
        setContentView(binding.root)

        chatRoomId = intent.getStringExtra("chatRoomId") ?: return
        otherUserId = intent.getStringExtra("otherUserId") ?: return
        myUserId = Firebase.auth.currentUser?.uid ?: ""
    }
}
```

다음으로 ChatAdapter를 생성해 RecyclerView에 LinearLayoutManager와 함께 연결해 준다.

<b>ChatActivity.kt</b>

```kotlin
class ChatActivity : AppCompatActivity() {
    
    ...

    override fun onCreate(savedInstanceState: Bundle?) {
        
        ...

        val chatAdapter = ChatAdapter()

        binding.chatRecyclerView.apply {
            layoutManager = LinearLayoutManager(context)
            adapter = chatAdapter
        }

        ...

    }
}
```

이후 채팅방에 표시할 이름을 받아오기 위해 "Users" 경로로 현재 사용자의 정보를 받아오고, Intent로 받아온 "otherUserId"로 다른 사용자의 정보를 받아온다. 다른 사용자의 정보는 채팅방에 상대방의 이름을 표시하기 위해 사용한다. 받은 상대방의 값을 ChatAdapter의 "otherUserItem" 값으로 넣어준다.

<b>ChatActivity.kt</b>

```kotlin
class ChatActivity : AppCompatActivity() {
    
    ...

    override fun onCreate(savedInstanceState: Bundle?) {
        
        ...

        Firebase.database.reference.child(DB_USERS).child(myUserId).get()
            .addOnSuccessListener {
                val myUserItem = it.getValue(UserItem::class.java)
                val myUserName = myUserItem?.username
            }

        Firebase.database.reference.child(DB_USERS).child(otherUserId).get()
            .addOnSuccessListener {
                val otherUserItem = it.getValue(UserItem::class.java)

                chatAdapter.otherUserItem = otherUserItem
            }
    }
}
```

이제 채팅 값을 받아오기 위해 Firebase DB의 "Chats"의 "채팅방 id"로 접근해 채팅 메시지가 업데이트될 때마다 이를 전달 받을 수 있도록 `addChildEventListener()` 리스너를 사용해 받아온다. 재정의 메서드로는 값이 추가되었을 때에만 처리할 것이기 때문에 `onChildAdded()`만을 재정의 하고 다른 4개의 재정의 메서드는 사용하지 않는다. 해당 메서드 내부에서 "chatItem" 변수로 받아 chatAdapter에 값을 추가하도록 한다. 아래 값에서 `submitList()` 내부에 `toMutableList()`를 호출한 이유는 해당 메서드 없이 그냥 값을 넣으면 `submitList()`는 값은 값을 가리킨다고 판단하여 업데이트를 하지 않는다. 따라서 `toMutableList()`는 Array를 새로 반환하므로 다른 값임을 명시할 수 있는 효과를 얻을 수 있다.

<b>ChatActivity.kt</b>

```kotlin
class ChatActivity : AppCompatActivity() {
    
    ...

    override fun onCreate(savedInstanceState: Bundle?) {
        
        ...

        Firebase.database.reference.child(DB_CHATS).child(chatRoomId)
            .addChildEventListener(object : ChildEventListener {
                override fun onChildAdded(snapshot: DataSnapshot, previousChildName: String?) {
                    val chatItem = snapshot.getValue(ChatItem::class.java)
                    chatItem ?: return

                    chatItemList.add(chatItem)
                    chatAdapter.submitList(chatItemList.toMutableList())
                }

                override fun onChildChanged(snapshot: DataSnapshot, previousChildName: String?) {}
                override fun onChildRemoved(snapshot: DataSnapshot) {}
                override fun onChildMoved(snapshot: DataSnapshot, previousChildName: String?) {}
                override fun onCancelled(error: DatabaseError) {}

            })
    }
}
```

<br><br>

## 📔 사용자 정보 화면에서 다른 사용자 클릭 시 채팅방으로 이동 구현

먼저 사용자 아이템을 클릭했을 때 채팅방으로 이동해야 하므로 `UserAdapter`에서 클릭 메서드를 추가해 주도록 한다.

<b>UserAdapter.kt<b>

```kotlin
class UserAdapter(val onClick: (UserItem) -> Unit) : ListAdapter<UserItem, UserAdapter.UserViewHolder>(diffUtil) {

    inner class UserViewHolder(private val binding: ItemUserBinding) : RecyclerView.ViewHolder(binding.root) {
        fun bind(item: UserItem) {
            binding.nicknameTextView.text = item.username
            binding.stateMessageTextView.text = item.stateMessage

            binding.root.setOnClickListener {
                onClick(item)
            }
        }
    }

    ...

}
```

UserFragment에서 UserAdapter를 생성한 부분을 재정의 해주도록 한다. 클릭 메서드 내부에서는 ChatActivity로 "채팅방 id"와 "상대방의 정보"를 넘겨주어야 한다. 따라서 현재 사용자의 id를 가져와 "myUserId" 변수에 넣어주고 채팅방 id를 위해 접근하는 경로인 `Firebase.database.reference.child(DB_CHAT_ROOMS).child(myUserId).child(otherUser.userId ?: "")`을 통해 채팅방 DB의 레퍼런스에 접근하여 "chatRoomDB" 변수에 넣어준다. 이제 두 가지 경우를 확인해야 한다.
1. 상대방과의 채팅방을 처음 생성한 경우
2. 해당 채팅방이 이미 생성되어 있는 경우

위 두 가지 경우를 고려해야 한다. 이를 해결하는 방법은 우선 "chatRoomDB" 레퍼런스 내부의 값이 있는지부터 확인하여 있을 경우와 없을 경우로 나누어서 처리하면 된다. 값이 존재할 경우에는 snapshot으로 가져와 ChatRoomItem 클래스에 넣어주고 여기서 채팅방 id 값을 가져와 적용하면 된다. 만약 채팅방이 처음 생성되는 경우에는 `UUID` 클래스를 사용하여 새로 채팅방 id를 생성한다. UUID의 `randomUUID()` 메서드는 무작위의 키를 생성한다. 따라서 해당 키를 String 형으로 받아 id로 등록해주고 ChatRoomItem에 넣어 "chatRoomDB"의 `setValue()`를 통해 데이터를 서버에 저장시켜 주면 된다. 마지막으로 ChatActivity로 넘겨우어야 할 두 값을 Intent에 넣어 보내주면 된다. Intent에서 보낼 때의 키 값은 companion object에 생성해 주도록 한다.

```kotlin
class UserFragment : Fragment(R.layout.fragment_userlist){

    ...

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        
        ...

        val userListAdapter = UserAdapter { otherUser ->
            val myUserId = Firebase.auth.currentUser?.uid ?: ""
            val chatRoomDB = Firebase.database.reference.child(DB_CHAT_ROOMS).child(myUserId).child(otherUser.userId ?: "")

            chatRoomDB.get().addOnSuccessListener {

                var chatRoomId = ""
                if (it.value != null) {
                    val chatRoom = it.getValue(ChatRoomItem::class.java)
                    chatRoomId = chatRoom?.chatRoomId ?: ""
                } else {
                    chatRoomId = UUID.randomUUID().toString()
                    val newChatRoom = ChatRoomItem(
                        chatRoomId = chatRoomId,
                        otherUserName = otherUser.username,
                        otherUserId = otherUser.userId
                    )
                    chatRoomDB.setValue(newChatRoom)
                }

                val intent = Intent(context, ChatActivity::class.java)
                intent.putExtra(EXTRA_OTHER_USER_ID, otherUser.userId)
                intent.putExtra(EXTRA_CHAT_ROOM_ID, chatRoomId)
                startActivity(intent)
            }
        }

        ...

    }

    companion object {
        const val EXTRA_CHAT_ROOM_ID = "chatRoomId"
        const val EXTRA_OTHER_USER_ID = "otherUserId"
    }
}
```

우선 위 과정을 통해서 앞서 작성해 두었던 ChatListFragment상에 채팅방이 생성된다.

ChatActivity에서 Intent로 값을 받아와 변수에 저장해 주는 것은 앞에서 작성했으므로 지금까지의 결과를 확인해 보자.

결과를 확인해 보면, 사용자 리스트에서 상대방을 클릭했을 때 성공적으로 채팅방으로 화면이 넘어간 것을 볼 수 있고, 이후 채팅방 리스트에서 해당 사용자와의 채팅방이 추가된 것을 볼 수 있다.

![](/assets/images/fastcampus/part2/chat/chat18.jpg)
![](/assets/images/fastcampus/part2/chat/chat19.jpg)


<br><br>

## 📔 채팅방 리스트에서 특정 채팅방 클릭 시, 해당 채팅방으로 이동 구현

해당 기능을 구현하는 방법은 위 방법과 같다. 마찬가지로 ChatListAdapter의 속성으로 클릭 메서드를 추가하고, ChatListFragment에서 값을 Intent로 넘겨주면 된다.

<b>ChatListAdapter.kt</b>

```kotlin
class ChatListAdapter(val onClick: (ChatRoomItem) -> Unit) : ListAdapter<ChatRoomItem, ChatListAdapter.ChatListViewHolder>(diffUtil) {

    ...

    inner class ChatListViewHolder(private val binding: ItemChatroomBinding) : RecyclerView.ViewHolder(binding.root) {
        fun bind(item: ChatRoomItem) {
            binding.nicknameTextView.text = item.otherUserName
            binding.lastMessageTextView.text = item.lastMessage

            binding.root.setOnClickListener {
                onClick(item)
            }
        }
    }

    ...

}
```

<b>ChatListFragment.kt</b>

```kotlin
class ChatListFragment : Fragment(R.layout.fragment_chatlist) {
    private lateinit var binding: FragmentChatlistBinding

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)
        binding = FragmentChatlistBinding.bind(view)

        val chatListAdapter = ChatListAdapter { chatRoomItem ->

            val intent = Intent(context, ChatActivity::class.java)
            intent.putExtra(UserFragment.EXTRA_CHAT_ROOM_ID, chatRoomItem.chatRoomId)
            intent.putExtra(UserFragment.EXTRA_OTHER_USER_ID, chatRoomItem.otherUserId)

            startActivity(intent)
        }

        ...

    }
}
```

<br><br>

## 📔 메시지 전송 기능 구현

"sendButton"의 클릭 리스너 내부에서 전송 기능을 구현한다. "messageEditText"에 입력된 내용을 "message" 변수를 생성하여 할당해 준다. 이후 예외 처리를 한 뒤, ChatItem을 생성하여 message와 userId의 값을 먼저 넣어주고 이를 "newChatItem" 변수에 할당해 준다. 채팅 id는 채팅방 id의 하위 요소를 만듦과 동시에 생긴 레퍼런스의 키 값으로 한다. ChatItem의 매개변수에 값을 모두 할당했으므로 `setValue()`하여 데이터를 저장한다. 

<b>ChatActivity.kt</b>

```kotlin
class ChatActivity : AppCompatActivity() {
    
    ...

    override fun onCreate(savedInstanceState: Bundle?) {
        
        ...

        binding.sendButton.setOnClickListener {
            val message = binding.messageEditText.text.toString()

            if (message.isEmpty()) {
                Toast.makeText(this, "메시지를 입력해주세요.", Toast.LENGTH_SHORT).show()
                return@setOnClickListener
            }

            val newChatItem = ChatItem(
                message = message,
                userId = myUserId
            )
        }
    }
}
```

이제 채팅방 리스트의 마지막 메시지와 상대방의 DB에도 업데이트를 적용해 주도록 한다. 업데이트 해야 할 요소는 5가지이다.

 1. "ChatRooms"/"현재 사용자 id"/"상대방 id"/"마지막 메시지" : "최근 메시지" -> 현재 사용자 채팅방의 최근 메시지 업데이트
 2. "ChatRooms"/"상대방 id"/"현재 사용자 id"/"마지막 메시지" : "최근 메시지" -> 상대 채팅방의 최근 메시지 업데이트
 3. "ChatRooms"/"상대방 id"/"현재 사용자 id"/"채팅방 id" : "채팅방 id" -> 상대방 db에서는 채팅방의 생성이 업데이트 되지 않았으므로 추가
 4. "ChatRooms"/"상대방 id"/"현재 사용자 id"/"상대방 id" : "현재 사용자 id" -> 상대방 db에서 현재 사용자는 상대방이므로 해당 정보 업데이트
 5. "ChatRooms"/"상대방 id"/"현재 사용자 id"/"상대방 이름" : "현재 사용자 이름" -> 위와 동일한 이유로 현재 사용자의 이름 업데이트

위의 수정 내용을 적용한 코드는 다음과 같다.

```kotlin
val updates: MutableMap<String, Any> = hashMapOf(
    "${DB_CHAT_ROOMS}/$myUserId/$otherUserId/lastMessage" to message,
    "${DB_CHAT_ROOMS}/$otherUserId/$myUserId/lastMessage" to message,
    "${DB_CHAT_ROOMS}/$otherUserId/$myUserId/chatRoomId" to chatRoomId,
    "${DB_CHAT_ROOMS}/$otherUserId/$myUserId/otherUserId" to myUserId,
    "${DB_CHAT_ROOMS}/$otherUserId/$myUserId/otherUserName" to myUserName,
)
```

위 변경된 내용의 데이터를 저장한 변수 "updates"를 `updateChildren()`을 통해 적용해 주고, 채팅창에 입력한 내용은 지워준다.

<b>ChatActivity.kt</b>

```kotlin
class ChatActivity : AppCompatActivity() {
    
    ...

    override fun onCreate(savedInstanceState: Bundle?) {
        
        ...

        binding.sendButton.setOnClickListener {
            
            ...

            val updates: MutableMap<String, Any> = hashMapOf(
                "${DB_CHAT_ROOMS}/$myUserId/$otherUserId/lastMessage" to message,
                "${DB_CHAT_ROOMS}/$otherUserId/$myUserId/lastMessage" to message,
                "${DB_CHAT_ROOMS}/$otherUserId/$myUserId/chatRoomId" to chatRoomId,
                "${DB_CHAT_ROOMS}/$otherUserId/$myUserId/otherUserId" to myUserId,
                "${DB_CHAT_ROOMS}/$otherUserId/$myUserId/otherUserName" to myUserName,
            )

            Firebase.database.reference.updateChildren(updates)

            binding.messageEditText.text.clear()
        }
    }
}
```

결과를 보면 채팅이 성공적으로 전송된 것을 볼 수 있고, Firebase DB에도 추가되거나 변경된 사항이 잘 반영된 것을 볼 수 있다.

![](/assets/images/fastcampus/part2/chat/chat20.jpg)
![](/assets/images/fastcampus/part2/chat/chat21.png)
![](/assets/images/fastcampus/part2/chat/chat22.png)

<br><br>

## 📔 채팅 알림 기능 구현(1)

채팅 앱을 실행하고 있지 않을 때 상대방에게서 채팅이 오면 이를 표시할 알림을 설정해 보자. 우선 FirebaseMessagingService를 상속받는 MyFirebaseMessagingService 클래스를 생성해 준다.

Manifest 파일에 일림 기능을 사용함을 나타내기 위해 다음과 같은 문장들을 추가한다.

<b>AndroidManifest.xml</b>

```kotlin
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools">

    <application

        ...
    
        >
    
        ...

        <service
            android:name=".MyFirebaseMessagingService"
            android:exported="false">
            <intent-filter>
                <action android:name="com.google.firebase.MESSAGING_EVENT" />
            </intent-filter>
        </service>
        <meta-data
            android:name="com.google.firebase.messaging.default_notification_channel_id"
            android:value="@string/default_notification_channel_id" />
    </application>

</manifest>
```

다시 MyFirebaseMessagingService로 돌아와 두 개의 메서드를 재정의 해주는데 우선 `onNewToken()` 메서드는 권장되지만 사용하지는 않으므로 구현만 해주도록 한다. 재정의할 메서드는 메시지를 수신했을 때 호출되는 `onMessageReceived()` 메서드를 재정의한다. 메서드 내부에서 알림 채널을 먼저 생성해주고 이를 NotificationManager와 연결시켜준 뒤, 메시지의 내용을 NotificationBuilder를 사용하여 알림의 형식을 설정하고 적용하도록 한다.

<b>MyFirebaseMessagingService.kt</b>

```kotlin
class MyFirebaseMessagingService : FirebaseMessagingService() {

    override fun onMessageReceived(message: RemoteMessage) {
        super.onMessageReceived(message)

        val name = "채팅 알림"
        val descriptionText = "채팅 알림입니다."
        val importance = NotificationManager.IMPORTANCE_DEFAULT
        val mChannel = NotificationChannel(getString(R.string.default_notification_channel_id), name, importance)
        mChannel.description = descriptionText
        val notificationManager = getSystemService(NOTIFICATION_SERVICE) as NotificationManager
        notificationManager.createNotificationChannel(mChannel)

        val body = message.notification?.body ?: ""
        val notificationBuilder =  NotificationCompat.Builder(applicationContext, getString(R.string.default_notification_channel_id))
            .setSmallIcon(R.drawable.ic_baseline_chat_bubble_24)
            .setContentTitle(getString(R.string.app_name))
            .setContentText(body)

        notificationManager.notify(0, notificationBuilder.build())
    }

    override fun onNewToken(token: String) {
        super.onNewToken(token)
    }
}
```

다음으로 MainActivity에서 알림 권한을 받도록 설정해주어야 한다. 

<b>MainActivity.kt</b>

```kotlin
class MainActivity : AppCompatActivity() {
    
    ...

    override fun onCreate(savedInstanceState: Bundle?) {
        
        ...

        askNotificationPermission()

        ...

    }

    ...

    private val requestPermissionLauncher = registerForActivityResult(
        ActivityResultContracts.RequestPermission()
    ) { isGranted ->
        if (isGranted) {

        } else {

        }
    }

    private fun askNotificationPermission() {
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.TIRAMISU) {
            if (ContextCompat.checkSelfPermission(
                    this,
                    android.Manifest.permission.POST_NOTIFICATIONS
                ) == PackageManager.PERMISSION_GRANTED
            ) {

            } else if (shouldShowRequestPermissionRationale(android.Manifest.permission.POST_NOTIFICATIONS)) {
                showPermissionRationalDialog()
            } else {
                requestPermissionLauncher.launch(android.Manifest.permission.POST_NOTIFICATIONS)
            }
        }
    }

    @RequiresApi(Build.VERSION_CODES.TIRAMISU)
    private fun showPermissionRationalDialog() {
        AlertDialog.Builder(this).apply {
            setMessage("메시지 알림을 위한 권한이 필요합니다.")
            setPositiveButton("허용") { _, _ ->
                requestPermissionLauncher.launch(android.Manifest.permission.POST_NOTIFICATIONS)
            }
            setNegativeButton("취소", null)
        }.show()
    }
}
```

이후 FCM 토큰을 받기 위해 LoginActivity에서 로그인이 처리되었을 때 토큰을 받아오는 코드를 넣어주도록 한다. 받아온 토큰을 기존 사용자 정보를 등록하는 코드에 추가하여 같이 등록하도록 한다.

<b>LoginActivity.kt</b>

```kotlin
class LoginActivity : AppCompatActivity() {

    ...

    override fun onCreate(savedInstanceState: Bundle?) {
        
        ...

        binding.signInButton.setOnClickListener {
            
            ...

            Firebase.auth.signInWithEmailAndPassword(email, password).addOnCompleteListener(this) { task ->
                val currentUser = Firebase.auth.currentUser
                if (task.isSuccessful && currentUser != null) {
                    
                    ...

                    Firebase.messaging.token.addOnCompleteListener {
                        val token = it.result

                        val user = mutableMapOf<String, Any>()
                        user["userId"] = userId
                        user["username"] = email
                        user["fcmToken"] = token

                        Firebase.database.reference.child(DB_USERS).child(userId).updateChildren(user)
                        startActivity(Intent(this, MainActivity::class.java))
                        finish()
                    }
                } else {
                    
                    ...

                }
            }
        }

        ...

    }
}
```

결과를 보면 권한 설정을 위한 팝업을 제대로 띄우는 것을 볼 수 있고, Firebase 서버에 FCM 토큰이 추가된 것을 볼 수 있다.

![](/assets/images/fastcampus/part2/chat/chat24.jpg)
![](/assets/images/fastcampus/part2/chat/chat23.png)

<br><br>

추가된 FCM 토큰을 사용해 Firebase 홈페이지의 클라우드 메시징에서 받은 토큰을 추가해 테스트를 해보면 알림이 잘 오는 것을 볼 수 있다.

![](/assets/images/fastcampus/part2/chat/chat25.jpg)

다음으로 메시지 값을 보내기 위해 설정에 기존 Cloud Messaging API로 변경한 뒤 FCM의 서버 키를 받아온다.

서버 키를 받아왔으니 POSTMAN을 사용하여 다른 사용자의 메시지도 전달되는지 확인해 보도록 하자. POASTMAN에서 통신 방식을 POST로 바꾸고 body 탭에서 아래와 같은 매개변수를 넣어 api 주소로 값을 보내보도록 하자. 이때 Header 탭에 Authorization 매개변수를 추가하여 `key=~`를 통해 서버 키를 넣어주는 것을 잊지 말자. 

![](/assets/images/fastcampus/part2/chat/chat26.png)

값이 성공적으로 전달 되었다면 알림이 제대로 오는 것을 볼 수 있다.

![](/assets/images/fastcampus/part2/chat/chat27.jpg)

<br>

성공적으로 전송되는 것을 확인했으니 이제 okhttp를 통해 서버에 직접 값을 전달해 보도록 하자.

먼저 OkHttp를 사용하기 위한 라이브러리를 추가하도록 한다.

<b>app:build.gradle</b>

```kotlin
implementation 'com.squareup.okhttp3:okhttp:4.10.0'
```

ChatActivity로 돌아와 다른 사용자의 Fcm 토큰을 받아오기 위해 클래스 변수로 만들어 준다. "sendButton"이 클릭 되었을 때 동작하므로 클릭 리스너의 블록 내부에서 OkHttpClient를 생성해 주고, JSONOjbect를 생성해 Json 형식으로 구성해 주도록 한다. 또한 "requestBody" 변수를 생성해 json 형식과 utf-8 인코딩을 사용함을 명시해주고 "request" 변수를 생성해 post() 메서드를 사용해 빌드해 주도록 한다. 이때 url과 헤더가 추가된다.

위 과정을 기반으로 만들어진 OkHttpClient를 `newCall()` 메서드를 통해 메시지를 발송한다. 확인을 위해 `enqueue()` 메서드 또한 적용해 준다. 이렇게 하면 메시지를 발송하는 과정은 끝이다.

<b>ChatActivity.kt</b>

```kotlin
class ChatActivity : AppCompatActivity() {
    
    ...

    private var otherUserFcmToken: String = ""

    ...

    override fun onCreate(savedInstanceState: Bundle?) {
        
        ...

        binding.sendButton.setOnClickListener {
            
            ...

            val client = OkHttpClient()

            val root = JSONObject()
            val notification = JSONObject()
            notification.put("title", getString(R.string.app_name))
            notification.put("body", message)

            root.put("to", otherUserFcmToken)
            root.put("priority", "high")
            root.put("notification", notification)

            val requestBody =
                root.toString().toRequestBody("application/json; charset=utf-8".toMediaType())
            val request =
                Request.Builder().post(requestBody).url("https://fcm.googleapis.com/fcm/send")
                    .header("Authorization", FCM_SERVER_KEY).build()

            client.newCall(request).enqueue(object : Callback {
                override fun onFailure(call: Call, e: IOException) {}

                override fun onResponse(call: Call, response: Response) {}
            })

            ...
            
        }
    }

    ...

}
```

반대로 메시지를 받기 위해서는 상대방의 Fcm 토큰을 알아야 한다. 이때 "otherUserFcmToken"은 앞서 작성했던 otherUserItem을 받아온 부분을 활용한다. UserItem 데이터 클래스는 토큰이 추가되어 있으므로 해당 블럭에서 상대방의 토큰을 가져올 수 있다.

<b>UserItem.kt</b>

```kotlin
data class UserItem(
    val userId: String? = null,
    val username: String? = null,
    val stateMessage: String? = null,
    val fcmToken: String? = null
)
```

<b>ChatActivity.kt</b>

```kotlin
Firebase.database.reference.child(DB_USERS).child(otherUserId).get()
    .addOnSuccessListener {
        val otherUserItem = it.getValue(UserItem::class.java)
        otherUserFcmToken = otherUserItem?.fcmToken.orEmpty()
        chatAdapter.otherUserItem = otherUserItem

        isInit = true
        getChatData()

    }
```

해당 문장을 메서드로 묶어 사용자의 정보를 가져왔을 때 호출하도록 하고 채팅 목록 또한 상대방 정보를 불러오는 메서드는 완료하고 난 후 호출할 수 있도록 한다. 이는 코드의 가독성을 높이기 위함이다. init 변수는 전송 버튼의 예외 처리를 위해 사용되었다. EditText에 아무런 값도 입력되어 있지 않으면 버튼은 동작하지 않는다.

<b>ChatActivity.kt</b>

```kotlin
class ChatActivity : AppCompatActivity() {
    
    ...

    override fun onCreate(savedInstanceState: Bundle?) {
        
        ...

        Firebase.database.reference.child(DB_USERS).child(myUserId).get()
            .addOnSuccessListener {
                val myUserItem = it.getValue(UserItem::class.java)
                myUserName = myUserItem?.username ?: ""

                getOtherUserData()
            }

        ...

    }

    private fun getOtherUserData() {
        Firebase.database.reference.child(DB_USERS).child(otherUserId).get()
            .addOnSuccessListener {
                val otherUserItem = it.getValue(UserItem::class.java)
                otherUserFcmToken = otherUserItem?.fcmToken.orEmpty()
                chatAdapter.otherUserItem = otherUserItem

                isInit = true
                getChatData()

            }
    }

    private fun getChatData() {
        Firebase.database.reference.child(DB_CHATS).child(chatRoomId)
            .addChildEventListener(object : ChildEventListener {
                override fun onChildAdded(snapshot: DataSnapshot, previousChildName: String?) {
                    val chatItem = snapshot.getValue(ChatItem::class.java)
                    chatItem ?: return

                    chatItemList.add(chatItem)
                    chatAdapter.submitList(chatItemList.toMutableList())

                }

                override fun onChildChanged(snapshot: DataSnapshot, previousChildName: String?) {}
                override fun onChildRemoved(snapshot: DataSnapshot) {}
                override fun onChildMoved(snapshot: DataSnapshot, previousChildName: String?) {}
                override fun onCancelled(error: DatabaseError) {}

            })
    }
}
```

결과를 보면 두 기기에서 알림이 정상적으로 수신되는 것을 확인할 수 있다.

![](/assets/images/fastcampus/part2/chat/chat28.jpg)
![](/assets/images/fastcampus/part2/chat/chat29.jpg)



<br><br>

## 📔전체 코드
<https://github.com/Becomeproo/chat_app>

<br>

## 📔마무리
이번 앱은 유독 이해하기 힘들었다. 세 개의 Firebase 기능을 사용했고 Authnetication 기능을 과정이 단순해 적용하기 쉬웠지만 Realtime Database와 Cloud Messaging 기능을 이해하기는 매우 쉽지 않았다. 무엇보다도 두 가지 기능을 반복해야 함을 느낄 수 있었던 반면 이제 RecyclerView와 데이터 클래스 Adapter 클래스를 연결해 뷰로 표시하는 것은 어느 정도 익숙해진 것 같아 기분이 좋았다.