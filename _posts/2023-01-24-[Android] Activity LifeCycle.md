---
title:  "[Android] Activity Lifecycle"

categories:
  - Android
tags:
  - [Android, Android 정리, Activity]

toc: true
toc_sticky: true
 
date: 2023-01-24 15:18:11+0900
last_modified_at: 2023-01-24 15:18:11+0900

---

<br>
<br>
<br>

> 아래 내용은 [Android Developer 공식 문서](https://developer.android.com/guide/components/activities/activity-lifecycle?hl=ko)와 [해로님 블로그](https://velog.io/@haero_kim/Activity-Lifecycle-%EC%99%84%EB%B2%BD-%EC%A0%95%EB%B3%B5%ED%95%98%EA%B8%B0) 내용을 참고하여 작성한 자습 기록입니다.

<br>
<br>

# 📚Android 정리 - Activity Lifecycle

이번에는 액티비티의 Lifecycle, 즉 액티비티의 생명주기에 관하여 다루고자 한다. 생명주기를 이해해야 하는 이유는 간단히 말하면 더 안정적이고 완성도 있는 앱을 개발할 수 있기 때문이다. 내가 만든 앱이 적지적기에 기능하길 원한다면 생명주기에 대해서 확실히 알고 갈 필요가 있다. 따라서 액티비티 생명주기에 관해 자세히 정리해 보자.

<br>

## 📔 Activity 생명주기
<br>

생명체들이 탄생하여 제 명을 다할 때까지 살아가듯이, 액티비티 또한 생성되고 소멸되는 과정을 거친다. 이 과정에서 액티비티는 각각의 상태로 변화한다. 앱을 실행할 때, 앱 실행 중 전화를 받는다거나 메시지를 보낼 때, 실행 중에 재난 문자와 같은 다이얼로그 알림이 뜨는 등 다양한 상황이 발생할 수 있는데, 액티비티는 이에 맞추어 상태가 변화한다.

위와 같은 상황들이 발생했을 때 생명주기에 대해 알지 못한다면, 진행 중이던 데이터를 저장하거나 처리하지 못하게 된다. 따라서 액티비티는 각각의 상태가 변화할 때마다 특정 동작을 수행할 수 있도록 여러 콜백 메서드를 제공한다. 예를 들어, 다음과 같은 상황들에 생명주기 콜백 메서드를 사용할 수 있다.

* 사용자가 앱을 사용하는 중 전화가 오거나, 다른 앱으로 전환할 때 비정상 종료되는 경우
* 사용자가 앱을 사용하지 않는 경우, 시스템 리소스가 소비되는 경우
* 사용자가 앱에서 나갔다가 다시 돌아왔을 때 사용자의 진행 상태가 저장되지 않는 경우
* 화면이 가로 방향 또는 세로 방향으로 전환될 때, 비정상 종료되거나 사용자의 진행 상태가 저장되지 않는 경우

### 📖 Activity 생명주기의 상태

액티비티는 각각의 상태에 맞는 유연한 앱을 위해 6가지 핵심 콜백 메서드를 제공한다. 핵심 콜백 메서드로는 `onCreate()`, `onStart()`, `onResume()`, `onPause()`, `onStop()`, `onDestroy()` 등이 있다. 시스템은 액티비티가 새로운 상태로 전환될 때마다 각각의 콜백 메서드를 호출한다. 아래는 Activity 생명주기를 시각적으로 나타낸 것이다.

<br>

![](/assets/images/android/lifecycle/lifecycle1.png)

## 📔 Lifecycle callbacks

각각의 콜백 메서드에 대해서 알아보자.

<br>

### 📖 onCreate()
`onCreate()`는 반드시 구현해야 하는 콜백 메서드로, 시스템이 액티비티를 처음 생성할 때 실행되는 메서드이다. 액티비티가 생성됨에 따라, 액티비티는 `Created(생성됨)` 상태에 진입하게 된다. `onCreate()`에서 액티비티의 전체 생명 주기 동안 한 번만 실행할 수 있는 초기화 및 시작 로직을 수행한다. 데이터를 특정 리스트에 바인딩 한다거나 뷰 모델을 연결시키는 일 등이 이중 한 예이다. 

`onCreate()` 메서드는 파라미터로 `savedInstanceState`를 수신하는데, 이는 액티비티의 이전 상태가 저장된 `Bundle` 객체이다(만약 액티비티가 처음 생성되었다면, `Bundle` 객체의 값은 null).

다음은 Android 공식 문서에 명시되어 있는 `onCreate()`에서 사용자 인터페이스 선언(XML 파일에 정의), 멤버 변수 정의, 일부 UI 구성과 같이 액티비티의 기본 설정을 보여주는 예이다. 해당 예시에서 `setContentView()`를 사용해 XML 파일인 `R.layout.main_activty`를 인자로 넘겨주는 것을 볼 수 있는데, `setContentView()`는 `onCreate()`에 종속적인 메서드이므로 반드시 해당 콜백 안에 구현해야 한다.

```kotlin

lateinit var textView: TextView

// 액티비티 인스턴스에 대한 일시적인 상태
var gameState: String? = null

override fun onCreate(savedInstanceState: Bundle?) {
    // 상위 클래스의 onCreate()를 호출하여 뷰 계층 구조와 같은 액티비티 생성을 완료
    super.onCreate(savedInstanceState)

    // 인스턴스에 저장된 값 복구
    gameState = savedInstanceState?.getString(GAME_STATE_KEY)

    // 현재 액티비티에 대한 UI 설정
    // 레이아웃 파일은 res/layout/main_activity.xml 파일을 사용
    setContentView(R.layout.main_activity)

    // TextView를 초기화 함에 따라 추후에 사용
    textView = findViewById(R.id.text_view)
}


//해당 콜백은 이전에 onSaveInstanceState()를 사용하여 저장된 인스턴스가 있는 경우에만 호출된다.
//몇몇 상태는 onCreate()에서 복원하지만, 이 외의 값들은 onStart()가 완료된 이후에 선택적으로 해당 콜백을 통해 복원될 수 있다.

// onCreate()의 savedInstanceState Bundle 객체와 동일하다.
override fun onRestoreInstanceState(savedInstanceState: Bundle?) {
    textView.text = savedInstanceState?.getString(TEXT_VIEW_KEY)
}

// 액티비티가 일시적으로 소멸되었을 때 호출되며 여기에 값을 저장할 수 있다.
override fun onSaveInstanceState(outState: Bundle?) {
    outState?.run {
        putString(GAME_STATE_KEY, gameState)
        putString(TEXT_VIEW_KEY, textView.text.toString())
    }
    // 뷰 계층 구조를 저장하기 위해 슈퍼클래스를 호출
    super.onSaveInstanceState(outState)
}
```

`onCreate()` 콜백의 실행이 종료된 후, 액티비티는 `Started(시작됨)` 상태가 되고, 이후 바로 `onStart()`와 `onResume()` 콜백을 연달아 호출한다.

<br>

### 📖 onStart()
액티비티가 `Started(시작됨)` 상태에 진입하게 되었을 때, `onStart()` 콜백을 호출한다. 해당 메서드는 사용자가 액티비티를 볼 수 있게 하며, 포그라운드에 진입한 액티비티와 상호작용할 수 있도록 준비한다.

`onStart()` 메서드는 매우 빠르게 완료되며, 앞서 언급했듯이 바로 `Resumed(재개됨)` 상태에 진입하게 된다. 그리고 시스템은 `onResume()` 콜백을 호출한다.

### 📖 onResume()

액티비티가 `Resumed(재개됨)` 상태에 진입하게 되면 포그라운드에 액티비티가 표시되고 앱이 사용자와 상호작용할 수 있게 된다. 앱은 전화가 오거나, 사용자가 다른 액티비티로 이동하거나 화면이 꺼지지 않는 등 앱 화면이 전환되지 않는 이상 계속해서 `Resumed` 상태에 머물게 된다.

해당 상태에서, 즉 사용자가 액티비티를 보는 포그라운드에서 실행해야 하는 모든 기능이 활성화될 수 있게 한다.

만일 방해 이벤트가 발생하게 되면, 액티비티는 `Paused(정지됨)` 상태에 진입하게 되고, 시스템은 `onPause()` 콜백을 호출한다. 이후 사용자가 `Paused(정지됨)` 상태에서 다시 `Resumed(재개됨)` 상태로 돌아오게 되면, 다시 말해 실행 중인 앱에서 포그라운드가 방해 이벤트로 인해 다른 액티비티 등으로 전환되었다가 돌아와 포그라운드에 앱이 다시 보이면, 시스템은 다시 한번 `onResume()` 콜백을 호출한다. 이러한 이유로, `onResumed()` 콜백에는 `onPause()` 상태에 진입한 동안 풀어졌던 컴포넌트들을 다시 초기화하고, 이외 `Resumed(재개됨)` 상태에 진입할 때마다 수행하게 되는 다른 초기화 작업들을 구현하면 된다.

특정 상태에서 액티비티를 초기화 했다면, 그에 해당하는 생명 주기 콜백을 사용해서 리소스를 해제하도록 권장된다. 예를 들어, `Started(시작됨)` 상태에서 초기화를 진행했다면, `Stopped(중단됨)` 상태에서 이를 해제하거나 종료해야하고, `Resumed(재개됨)` 상태에서 초기화를 진행했다면, `Paused(정지됨)` 상태에서 해제하도록 하자.

<br>

### 📖 onPause()

사용자가 액티비티를 잠시 떠났을 때, 시스템은 `onPause()` 콜백을 호출한다. 여기서 잠시 떠났다는 말의 의미는 액티비티가 소멸되었다는 의미가 아니라, 액티비티가 더 이상 포그라운드에 존재하지 않음을 의미한다. 멀티 윈도우 모드에서는 두 개의 앱이 한 화면에 동시에 표시되기 때문에 포그라운드에 위치해 있음에도 다른 액티비티에 포커스를 하고 있으므로 `onPause()`가 호출된다. onPause() 메서드에는 액티비티가 `Paused(정지됨)` 상태일 때 계속 실행되어서는 안되지만 다시 시작할 작업을 일시적으로 정지하는 작업을 구현한다. 

액티비티가 `Paused(정지됨)` 상태에 진입하는 경우는 다음과 같다.
* 가장 흔한 예로, 일부 이벤트가 앱 실행을 방해했을 경우
* 안드로이드 7.0(API 24) 이상에서는 멀티 윈도우 모드로 인해 다수의 앱이 실행된다. 언제든지 해당 앱들 중 하나를 포그라운드로 포커스 할 수 있기 때문에 시스템은 이 외의 앱들을 정지시킨다.
* 다이얼로그와 같이 실행 중인 앱을 반투명한 상태로 만들면서, 그 위에 보다 작은 화면이 보이는 경우, 정지됨 상태로 유지

이 외에도 배터리 수명에 영향을 미칠 수 있는 리소스, 센서 등을 해제함으로써 자원을 효율적으로 사용하고자 할 때 `onPause()` 콜백을 사용할 수 있다.

onPause()는 매우 짧고 잠깐 실해되기 때문에 무언가를 저장을 하기엔 시간이 부족할 수 있다. 따라서 네트워크 호출, DB 작업과 같이 앱 데이터 또는 사용자 데이터를 저장할 때에는 작업이 완료되기 전에 해당 콜백이 먼저 종료될 수 있기 때문에 `onPause()`를 사용해서는 안 된다. 이를 위해 부하가 큰 작업들은 `onStop()`에서 처리해 주도록 한다.

액티비티는 다시 시작되거나 사용자가 화면을 완전히 볼 수 없게 되지 않는 이상 계속 `Paused(정지됨)` 상태에 머물게 된다. 이때, 액티비티가 재개되면 메모리에 남아있던 액티비티 인스턴스를 다시 불러와 onResume() 메서드를 호출한다.

만약 액티비티가 화면에 완전히 보이지 않게 되면, 시스템은 `onStop()`을 호출한다.

### 📖 onStop()
액티비티가 더 이상 사용자가 보는 화면에 보이지 않게 되면, 액티비티는 `Stopped(중단됨)` 상태에 진입하고, 시스템은 `onStop()` 콜백을 호출한다. 다시 말해, 새롭게 실행된 액티비티가 전체 화면을 차지하게 되거나 액티비티의 실행이 완료되어 종료되는 시점에 `onStop()`이 호출된다.

`onStop()` 콜백에서는 앱이 사용자에게 보이지 않는 동안 필요치 않은 리소스를 해제하거나 조정해야 한다. 애니메이션을 일시 중지하거나, GPS 사용 시, 배터리를 위해 위치 정확도를 '세밀한 위치 측정'에서 '대략적인 위치 측정'으로 업데이트할 수 있다.

`onPause()` 대신 `onStop()`을 사용하면 예를 들어 멀티윈도우 모드에서, 사용자가 다른 앱을 포커스하고 있을 때에도 UI 작업을 계속해서 진행할 수 있다. 다른 앱으로 포커스를 전환하게 되면 `onPause()`가 호출되기 때문에 `onPause()` 내에 UI 중지 작업을 구현하게 되면, 멀티윈도우에서 액티비티가 화면에 표시되고 있음에도 UI가 멈춰버리기 때문이다. `onStop()`에서는 해당 구현 내용이 멈추지 않기 때문에 UI 중지 작업은 `onStop()`에 구현하도록 한다.

또한, CPU를 많이 소모하는 작업을 종료하고자 할 때 `onStop()`을 사용한다. 예를 들어, 정보를 DB에 저장할 적절한 시기를 모르겠다면, 해당 상태에 저장해 주면 된다. 다음 코드를 보자.

```kotlin
override fun onStop() {
    // 슈퍼클래스의 메서드 호출
    super.onStop()

    // 액티비티가 중지 상태이기 때문에 현재 노트 초안을 저장
    // 또한 노트의 진행 내용이 손실되어서는 안됨
    val values = ContentValues().apply {
        put(NotePad.Notes.COLUMN_NAME_NOTE, getCurrentNoteText())
        put(NotePad.Notes.COLUMN_NAME_TITLE, getCurrentNoteTitle())
    }

    // AsyncQueryHandler 또는 비슷한 기능으로 백그라운드에서 업데이트 수행
    asyncQueryHandler.startUpdate(
            token,     // int token to correlate calls
            null,      // cookie, not used here
            uri,       // The URI for the note to update.
            values,    // The map of column names and new values to apply to them.
            null,      // No SELECT criteria are used.
            null       // No WHERE columns are used.
    )
}
```

액티비티가 `Stopped(중단됨)` 상태에 진입하더라도, 액티비티 객체는 메모리 안에 머문다. 이 객체가 모든 상태 및 멤버를 관리하지만 윈도우 매니저와 연결되어 있지는 않다. 액티비티가 다시 시작되면, 액티비티는 정보를 재호출한다. 이 과정에서 `Resumed(재개됨)` 상태에서 구현된 컴포넌트들은 다시 초기화하지 않는다. 또한 시스템은 레이아웃에 있는 각 View 객체의 현재 상태도 기록한다. 따라서 사용자가 EditText 위젯에 텍스트를 입력하면 해당 내용이 저장되기 때문에 이를 저장 및 복원할 필요가 없다. 즉, 시스템이 더 우선순위가 높은 프로세스를 위해 메모리를 확보해야 하는 경우 해당 액티비티를 죽이게 되지만, 이 과정에 Bundle View 객체의 상태를 그대로 저장해두고, 사용자가 다시 돌아오게 되면 이 상태를 다시 복원한다.

`Stopped(중단됨)` 상태에서 두 가지 경우로 인해 상태가 전환될 수 있는데, 만약 사용자가 다시 액티비티로 돌아오면, `onRestart() -> onStart() -> onResume()`이 호출되며 `Resumed(재개됨)` 상태로 돌아와 다시 사용자가 상호작용을 진행하게 된다. 또 다른 경우는 사용자에 의해 또는 시스템에 의해 액티비티가 완전히 종료되면, 시스템은 `onDestroy()`를 호출한다.

<br>

### 📖 onDestroy()

`onDestroy()`는 액티비티가 완전히 소멸되기 전에 호출된다. 시스템이 해당 콜백을 호출하게 되는 경우는 다음과 같다.

* 사용자가 앱을 완전히 종료하거나 `finish()`가 호출된 경우
* 화면이 가로 또는 세로로 전환되거나 멀티 윈도우 모드로 인한 구성 변경으로 인해 액티비티가 일시적으로 소멸되는 경우

액티비티가 종료되면, `onDestroy()`는 액티비티가 수신하는 마지막 생명주기 콜백 메서드가 된다. 만약 해당 콜백이 화면 구성 변경으로 인해 호출되었다면, 시스템은 즉시 새로운 액티비티 인스턴스를 생성하고 `onCreate()`를 호출한다.

`onDestroy()`가 호출되기까지(`onStop()`과 같이 이전 생명주기에서) 해제되지 않은 리소스가 있다면 모두 해제해 주어야 한다. 메모리 누수의 위험이 있기 때문이다.