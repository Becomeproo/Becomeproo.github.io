---
title:  "[Android] Context"

categories:
  - Android
tags:
  - [Android, Android 정리, Context]

toc: true
toc_sticky: true
 
date: 2023-01-28 15:43:27+0900
last_modified_at: 2023-01-28 15:43:30+0900

---

<br>
<br>
<br>

> 아래 내용은 [Android Developer 공식 문서](https://developer.android.com/reference/android/content/Context) 내용을 참고하여 작성한 자습 기록입니다.

<br>
<br>

# 📚Android 정리 - Context

Context, 안드로이드를 다루면서 가장 많이 사용하지만 정작 정확하게 설명해 볼 수 있냐고 물어본다면 자신이 없다. 오늘은 뭘 정리해야 좋을까라는 생각을 시작하기도 무섭게 Context가 떠올랐다. 긴 말이 필요 없다. 정리해 보자.

## 📔 Context

`Context` 클래스는 안드로이드의 많고 많은 컴포넌트들의 근간이 되는 요소이다. `Context`는 다양한 리소스 및 서비스에 대한 액세스를 제공하며 다양한 API 호출에서 매개변수로 자주 사용된다. 주로 사용되는 용도들을 보면 아래와 같다.

## 📔 Context의 사용

|**사용**|설명|
|:---:|:---|
|리소스 접근|`Context` 클래스는 `getResources()` 메서드를 사용하여 layouts, strings, drawables 과 같은 리소스들에 접근할 수 있도록 해준다|
|액티비티 실행|`Context` 클래스를 사용하면 `startActivity()` 메서드를 사용하여 새로운 액티비티를 실행할 수 있다|
|Shared Preferences 접근|`Context` 클래스를 사용하면 `getSharedPreferences()` 메서드를 사용하여 공유 저장소에 접근할 수 있다|
|내부 저장소 접근|`Context` 클래스의 `getFilesDir()` 메서드를 사용하여 장치의 내부 저장소에 접근할 수 있다|
|외부 저장소 접근|`Context` 클래스의 `getExternalFilesDir()` 메서드를 사용하여 장치의 외부 저장소에 접근할 수 있다|
|BradcastRecevier 등록 및 해지|`Context.registerReceiver()`와 `Context.unregisterReceiver()` 메서드를 사용하여 동적으로 브로드캐스트 리시버를 등록하거나 해지할 수 있다|
|Content Provider 접근|`Context` 클래스를 사용하면 `getContentResolver()` 메서드를 사용하여 content provider에 접근할 수 있다|
|시스템 서비스 가져오기|`Context` 클래스를 사용하면 `getSystemService()` 메서드를 사용하여 LayoutInflater 서비스, ActivityManager 서비스, NotificationManager 서비스와 같은 다양한 시스템 서비스에 접근할 수 있다|
|응용 프로그램 리소스 접근|`Context` 클래스를 사용하면 `getDir()` 메서드를 사용하여 개인 데이터, 파일 및 DB와 같은 응용 프로그램의 리소스에 액세스할 수 있다|

<br>
<br>

## 📔 Context의 종류

위에서 보았듯이 `Context`는 다양한 용도로 사용되기 때문에 이에 맞는 `Context`의 종류 또한 다양하다. 주요 `Context`들을 보면 아래와 같다.

|**Context**|설명|
|:---:|:---|
|`ApplicationContext`|앱의 전체 수명 동안 사용할 수 있는 전역 컨텍스트로, 앱이 실행될 때 생성되며 앱 내의 액티비티, 서비스 또는 이외 컴포넌트에서 액세스할 수 있다. 해당 Context는 특정 액티비티와 연관되지 않아 액티비티의 수명 주기에 영향을 받지 않는다. 이 Context는 `getApplicationContext()` 메서드를 사용하여 액세스할 수 있다|
|`ActivityContext`|특정 액티비티에 연결된 Context로, 액티비티의 실행과 함께 생성되며 해당 액티비티 내에서 접근할 수 있다. 이 Context는 액티비티 생명 주기에 따라 영향을 받는다. 예를 들어, 액티비티가 `Paused(정지됨)` 상태일 경우, Context 또한 정지된다. 액티비티 클래스 내에서 `this` 키워드 또는 `getApplicationContext()` 메서드를 사용하여 해당 Context에 접근할 수 있다|
|`ServiceContext`|특정 서비스에 연결된 Context로, 서비스의 실행과 함께 생성된다. `this` 키워드와 서비스 클래스 내에서 접근할 수 있다|
|`BroadcastReceiverContext`|브로드캐스트 리시버와 연관된 Context로, 브로드캐스트 리시버가 등록되었을 때 생성된다. 리시버 클래스 내에서 `this` 키워드를 통해 접근할 수 있다|
|`ContentProviderContext`|Content Provider와 연관된 Context로, Content Provider와 동시에 생성된다. Content Provider 클래스 내 `this` 키워드를 통해 접근할 수 있다|
|`FragmentContext`|특정 프래그먼트와 연결되며 프래그먼트가 액티비티에 연결될 때 생성된다. 프래그먼트 클래스 내부의 `getContext()` 메서드를 사용하여 접근할 수 있다|

위와 같이 각 클래스에는 해당 클래스의 특정 인스턴스와 연결된 고유한 Context 개체가 있다.

<br>

## 📔 ApplicationContext와 ActivityContext의 사용

ApplicationContext와 ActivityContext는 가장 많이 보는 Context들이다. 하지만 Context를 인자로 지정할 때마다 둘 중 어떤 Context를 사용해야 하는지에 대해서는 구별하기 힘들었다. 잘못된 사용으로 인해 메모리 릭 또는 다른 오류들을 유발할 수도 있기 때문에 각각의 Context가 어떤 상황에서 사용되는지 알아보자.

### 📖 ApplicationContext 사용
현재 실행되고 있는 액티비티에 관계없이 앱의 전체 수명 동안 사용할 수 있는 Context가 필요한 경우네느 ApplicationContext를 사용한다. 예를 들어, 백그라운드 서비스나 여러 액티비티 간에 지속되어야 하는 싱글톤 클래스를 만드는 경우 ApplicationContext를 사용하면 된다.

아래는 ApplicationContext가 사용된 예시 코드이다.

* background service 실행

    ```kotlin
    val serviceIntent: Intent = Intent(applicationContext, MyService::class.java)
    startService(serviceIntent)
    ```
* BroadcastReceiver 등록

    ```kotlin
    val intentFilter = IntentFilter("com.example.broadcast")
    registerReceiver(MyReceiver(), intentFilter)
    ```

<br>

### 📖 ActivityContext 사용
특정 액티비티와 관련된 Context가 필요한 경우 ActivityContext를 사용한다. 예를 들어, 다이얼로그를 만들거나 새 액티비티를 실행하는 경우 현재 액티비티의 ActivityContext를 사용해야 한다.

아래는 ActivityContext가 사용된 예시 코드이다.

* 새 액티비티 실행

    ```kotlin
    val intent = Intent(this, SecondActivity::class.java)
    startActivity(intent)
    ```
* 다이얼로그 생성

    ```kotlin
    val myDialog = AlertDialog.Builder(this)
    ```