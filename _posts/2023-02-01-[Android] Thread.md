---
title:  "[Android] Thread"

categories:
  - Android
tags:
  - [Android, Android 정리, Thread]

toc: true
toc_sticky: true
 
date: 2023-02-01 20:28:49+0900
last_modified_at: 2023-02-01 20:28:53+0900

---

<br>
<br>
<br>

> 아래 내용은 [Android Developer 공식 문서](https://developer.android.com/guide/components/processes-and-threads?hl=ko) 내용을 참고하여 작성한 자습 기록입니다.

<br>
<br>

# 📚Android 정리 - Thread

## 📔 메인 스레드와 Worker Thread

스레드는 안드로이드 운영체제의 기본적인 부분이며 UI의 상태를 원활하고 유연하게 유지하는 데 중요한 역할을 한다.

앱이 실행되었을 때, 시스템은 앱을 실행하는 메인 스레드를 생성한다. 메인 스레드는 화면을 업데이트하고, 사용자 입력 값에 응답하고, 애니메이션을 동작시키는 등 UI 구현 및 사용자와 상호작용하는 데 있어서 중요한 역할을 한다. 이러한 이유로, 메인 스레드는 UI 스레드라고도 불린다.

메인 스레드를 사용하는 데 있어서 주의할 사항이 있는데, 메인 스레드에서는 리소스 또는 메모리를 많이 소모하는 작업은 지양해야 한다. 네트워크 액세스나 데이터베이스 쿼리 등의 시간 소요가 긴 작업들을 수행할 때에는 전체 UI가 차단된다. 메인 스레드가 차단되면, UI를 그리는 작업을 포함해서 아무런 이벤트도 처리될 수 없다. 또한 만약 UI 스레드가 차단된 상태가 약 5초 이상 지속될 경우, ANR(Application Not Responding)이라는 다이얼로그가 표시된다. 극단적으로, 해당 상황을 겪은 사용자가 앱을 그냥 실행하지 않고 지워버리는 상황을 초래할 수도 있다. 따라서 긴 시간을 요구하는 작업들은 스레드를 구분하여 따로 실행시켜줄 필요가 있다. 

또한 Android UI 도구 키트는 스레드로부터 안전하지 않다. 따라서 UI를 메인 스레드 이외의 스레드에서는 조작해선 안된다. 이와 관련해서 Android 단일 스레드 모델은 두 가지 규칙을 가진다.
1. UI 스레드를 차단하지 않는다
2. UI 스레드 외부에서 Android UI 도구 키트에 접근해선 안된다.


위의 상황을 해결하기 위해 존재하는 스레드가 Worker Thread이다. Worker Thread를 통해 네트워크 또는 DB와 관련된 무거운 작업들은 따로 스레드를 생성하여 작업한다.

<br>

두 개의 스레드에서 처리되는 방식은 다음과 같다.

![](/assets/images/android/thread/thread.drawio.png)

메인 스레드는 하나의 Looper를 가지고 있다. Looper는 스레드와 연결된 message queue에서 메시지를 확인하고 발송하면서 지속적으로 반복되는 객체이다. Worker Thread가 메인 스레드로 작업을 공유하고자 할 때, Handler를 사용하여 메인 스레드의 message queue에 메시지를 보낸다. Looper는 큐에서 메시지를 검색하고 Handler에 처리를 위한 적절한 지시를 한다. 이에 따라 Handler는 메시지의 정보를 기반으로 UI 업데이트와 같은 작업을 수행할 수 있다. Looper는 스레드가 종료되지 않는 한, 계속해서 메시지를 확인하고 검색한다.

이와 같이 루퍼는 메인 스레드와 Worker Thread 사이에서 중간 역할을 수행한다. 

## 📔 스레드 사용

스레드는 4가지의 방법으로 구현 가능하다.

### 📖 Thread 클래스 상속을 통한 구현

```kotlin
class ThreadExample : Thread() {
    override fun run() {
        println("HelloWorld")
    }
}

fun main() {
    val thread = ThreadExample()
    thread.start()
}
```

### 📖 Runnable 인터페이스 상속을 통한 구현

```kotlin
class RunnableExample : Runnable {
    override fun run() {
        println("HelloRunnable")
    }
}

fun main() {
    val thread = Thread(RunnableExample())
    thread.start()
}
```

### 📖 익명 객체를 통한 구현

```kotlin

fun main() {
    val thread = object : Thread() {
        override fun run() {
            println("Hello Object Runnable")
        }
    }
    
    thread.start()
}
```

### 📖 람다식을 통한 구현

```kotlin

fun main() {
    val thread = Thread {
        println("Hello lambda")
    }

    thread.start()
}
```

<br>

## 📔 Worker Thread로부터 전달된 UI 처리

메인 스레드 외의 스레드로부터 UI 처리에 관한 메시지를 요청하는 방법은 4가지가 존재했으나 가장 애용되었던 `AsyncTask`는
1. 오직 한 번만 실행되어 재사용이 불가능
2. 종료를 직접 해주지 않으면 종료가 되지 않아 메모리 릭이 발생
3. 항상 UI 스레드에서 호출해야 함
등과 같은 이유로 Deprecated 되었다.

따라서, 다음과 같이 3가지 방법이 존재한다.

### 📖 runOnUiThread()

`runOnUiThread` 메서드는 액티비티 클래스의 정적 메서드이다. 백그라운드에서 수행된 UI 처리 작업을 메인 스레드에서 실행시킨다.
```kotlin
class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)


        Thread {
            // 이곳에 네트워크 작업과 같은 백그라운드 작업 수행
            println("백그라운드 작업 수행")

            runOnUiThread {
                // 이곳에 백그라운드 작업 기반의 UI 처리 수행 및 UI 업데이트
            }
        }.start()
    }
}
```

### 📖 post()

`post()` 메서드는 뷰 클래스의 메서드이다. 해당 메서드 또한 마찬가지로 백그라운드에서 수행된 UI 처리 작업을 메인 스레드에서 실행시킨다.

```kotlin
class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)


        val textView = findViewById<TextView>(R.id.textView)

        Thread {
            // 이곳에서 백그라운드 작업 처리
            println("백그라운드 작업 수행")

            textView.post {
                // 이곳에 백그라운드 작업 기반의 UI 처리 수행 및 UI 업데이트
            }
        }.start()
    }
}
```

### 📖 Handler

위에서 언급했듯이 핸들러는 스레드에서 메시지 객체를 전송하고 처리한다. 핸들러는 백그라운드 스레드에서 메시지 객체를 메인 스레드의 Message Queue에 전송해 UI 처리하고 업데이트한다.

```kotlin
class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        val handler = Handler(Looper.getMainLooper())

        Thread {
            // 이곳에서 백그라운드 작업 처리
            println("백그라운드 작업 수행")

            handler.post {
                // 이곳에 백그라운드 작업 기반의 UI 처리 수행 및 UI 업데이트
            }
        }.start()
    }
}
```