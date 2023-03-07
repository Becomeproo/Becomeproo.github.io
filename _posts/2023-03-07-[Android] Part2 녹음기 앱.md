---
title:  "[Android] Part2-녹음기 앱"

categories:
  - Android
tags:
  - [Android, Kotlin, 안드로이드 강의]

toc: true
toc_sticky: true
 
date: 2023-03-07 10:15:40+0900
last_modified_at: 2023-03-07 10:15:48+0900
---

<br>
<br>
<br>

> 아래 내용은 패스트 캠퍼스 [35개 프로젝트로 배우는 Android 앱 개발 feat. Jetpack Compose]의 강의 내용을 기반으로 작성한 자습 기록입니다.

<br><br>

# 웹뷰 앱

## ℹ️앱 설명

소리를 녹음 및 재생할 수 있는 녹음기 앱

<br>

## ✅구현 기능

* CustomView를 활용하여 소리 시각화
* MediaRecorder와 MediaPlayer를 활용하여 녹음 및 재생 기능 구현

<br>

## ✅사용되는 기능

* Android
    * MediaPlayer
    * MediaRecorder
    * Permission Request
    * Canvas
    * Handler
    * CustomView


---

<br><br>

## 📔 녹음기 UI 구현

우선 앱의 UI는 비교적 간단하다. 녹음 시간을 표시하기 위한 TextView를 상단에 위치시키고, 하단에는 각각 재생, 녹음, 정지 기능을 수행할 버튼의 용도로 ImageView를 3개 위치시켰다. 마지막으로 시각화된 소리를 표시할 커스텀 뷰를 배치할 것이므로 이후 구현을 위해 View를 먼저 배치해 주었다.

<b>activity_main.xml</b>

```kotlin
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">

    <TextView
        android:id="@+id/timerTextView"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="00:00.00"
        android:textSize="40sp"
        app:layout_constraintBottom_toTopOf="@id/amplitudeView"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent" />

    <View
        android:id="@+id/amplitudeView"
        android:layout_width="0dp"
        android:layout_height="300dp"
        android:background="@color/gray"
        app:layout_constraintBottom_toTopOf="@id/recordButton"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent" />

    <ImageView
        android:id="@+id/recordButton"
        android:layout_width="50dp"
        android:layout_height="50dp"
        android:layout_marginBottom="50dp"
        android:src="@drawable/ic_baseline_fiber_manual_record_24"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:tint="@color/red" />

    <ImageView
        android:id="@+id/playButton"
        android:layout_width="50dp"
        android:layout_height="50dp"
        android:src="@drawable/ic_baseline_play_arrow_24"
        app:layout_constraintBottom_toBottomOf="@id/recordButton"
        app:layout_constraintEnd_toStartOf="@id/recordButton"
        app:layout_constraintStart_toStartOf="parent" />

    <ImageView
        android:id="@+id/stopButton"
        android:layout_width="50dp"
        android:layout_height="50dp"
        android:src="@drawable/ic_baseline_stop_24"
        app:layout_constraintBottom_toBottomOf="@id/recordButton"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toEndOf="@id/recordButton" />

</androidx.constraintlayout.widget.ConstraintLayout>
```

![](/assets/images/fastcampus/part2/recorder/recorder1.png)

<br><br>

## 📔 오디오 권한 설정 구현

녹음기에 사용되는 오디오 권한은 안드로이드에서 위험한 권한으로 분류된다. 따라서 오디오 권한을 사용자에게 요청해 권한 허용을 받아 사용할 수 있도록 구현해야 한다. 권한 설정은 이전의 앱들에서도 미리 다루었으므로 쉽게 할 수 있을 것이다. 

<b>MainActivity.kt</b>

```kotlin
override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    binding = ActivityMainBinding.inflate(layoutInflater)
    setContentView(binding.root)

    binding.recordButton.setOnClickListener {
        when {
            ContextCompat.checkSelfPermission(
                this,
                AUDIO_REQUEST_PERMISSION
            ) == PackageManager.PERMISSION_GRANTED -> {

            }
            ActivityCompat.shouldShowRequestPermissionRationale(this, AUDIO_REQUEST_PERMISSION) -> {
                showPermissionRationalDialog()
            }
            else -> {
                ActivityCompat.requestPermissions(this, arrayOf(AUDIO_REQUEST_PERMISSION), AUDIO_REQUEST_PERMISSION_CODE)
            }
        }
    }
}

private fun showPermissionRationalDialog() {
    AlertDialog.Builder(this).apply {
        setMessage("녹음 기능을 수행하기 위한 권한이 필요합니다.")
        setPositiveButton("권한 허용") { _, _ ->
            ActivityCompat.requestPermissions(this@MainActivity, arrayOf(AUDIO_REQUEST_PERMISSION), AUDIO_REQUEST_PERMISSION_CODE)
        }
        setNegativeButton("거부", null)
    }.show()
}
```

![](/assets/images/fastcampus/part2/recorder/recorder2.jpg)
![](/assets/images/fastcampus/part2/recorder/recorder3.jpg)


위 내용까지는 기존 앱 권한 설정 과정과 다르지 않다. 다만, 이번에는 추가적으로 권한 허용에 대해 계속해서 거부하는 사용자에 대해서 권한 알림을 요청하는 것은 권장되지 않으므로 앱에 해당 권한을 사용해야 하는 이유에 대해 충분한 설명을 한 번 더 하고 앱 설정에서 사용자가 직접 권한을 허용할 수 있는 방법으로 유도해야 한다. 기존의 권한 설정에 부가적으로 시스템 권한 설정에 대한 응답을 재정의하는 `onRequestPermissionsResult()` 메서드에서 해당 기능을 추가하면 된다.

<b>MainActivity.kt</b>

```kotlin
override fun onRequestPermissionsResult(
    requestCode: Int,
    permissions: Array<out String>,
    grantResults: IntArray
) {
    super.onRequestPermissionsResult(requestCode, permissions, grantResults)

    val audioRecordPermissionGranted = requestCode == AUDIO_REQUEST_PERMISSION_CODE && grantResults.firstOrNull() == PackageManager.PERMISSION_GRANTED

    if (audioRecordPermissionGranted) {

    } else {
        if (shouldShowRequestPermissionRationale(AUDIO_REQUEST_PERMISSION)) {
            showPermissionRationalDialog()
        } else {
            showPermissionDisallowDialog()
        }
    }
}
```

위의 `showPermissionDisallowDialog()` 메서드는 권한 설정의 필요성에 대한 설명을 보았음에도 시스템 UI에서 한 번 더 거절한 경우 앱 설정에서 직접 권한 설정을 하도록 유도하기 위해 부가적으로 설명하는 메서드이다.

```kotlin
private fun showPermissionDisallowDialog() {
    AlertDialog.Builder(this).apply {
        setMessage("앱을 실행하기 위해서 권한 설정을 해주셔야 합니다. 앱 설정에서 권한을 허용해주세요.")
        setPositiveButton("앱 설정") { _, _ ->
            navigateToAppSetting()
        }
        setNegativeButton("취소", null)
    }.show()
}
```

![](/assets/images/fastcampus/part2/recorder/recorder4.jpg)


사용자가 앱 설정으로 버튼을 클릭하였다면, 앱의 권한 설정을 지정하기 위한 설정 화면으로 이동한다. 이 과정에서 Intent가 사용되고 data의 값으로 앱의 패키지 이름을 필요로 한다.

앱 설정에서 권한을 허용 또는 이전 과정에서 이미 권한을 허용했다면 오디오 기능을 사용할 수 있다. 다음으로 오디오 기능을 구현해 보자.

```kotlin
private fun navigateToAppSetting() {
    Intent(Settings.ACTION_APPLICATION_DETAILS_SETTINGS).apply {
        data = Uri.fromParts("package", packageName, null)
        startActivity(this)
    }
}
```

![](/assets/images/fastcampus/part2/recorder/recorder5.jpg)

<br><br>

## 📔 녹음 기능 구현

녹음 기능을 구현하기 위해 "recordButton"을 클릭했을 때 권한을 확인하는 코드들을 `checkAudioPermissionAndRecord()` 메서드로 지정해 주었다.

`startRecording()` 메서드를 만들어 녹음 기능을 활성화한다. 이를 위해 MediaRecorder를 사용하는 데, MediaRecorder는 앱에서 전체적으로 사용되므로 클래스 변수로 지정해 준다. 추가적으로 이후 MediaRecorder와 MediaPlayer에서 사용될 파일 경로를 사용하기 위해 "fileName"이라는 이름으로 문자열 변수를 지정해 주었다. 해당 변수는 액티비티 생성과 동시에 초기화된다. 파일 형식은 절대 경로와 함께 파일 유형으로 .3gp를 지정한다.

<b>MainActivity.kt</b>

```kotlin
class MainActivity : AppCompatActivity() {

    ...

    private var recorder: MediaRecorder? = null
    private var fileName = ""

    
    override fun onCreate(savedInstanceState: Bundle?) {
        
        ...

        fileName = "${externalCacheDir?.absolutePath}/audiorecordtest.3gp"

        ...

    }
}
```

다시 `startRecording()` 메서드로 돌아와서 MediaRecorder를 생성한다. 생성과 동시에 4가지 설정을 같이 해주었다. `setAudioSource()`를 통해 녹음할 오디오 소스가 마이크임을 명시해 주었고, `setOutputFormat()`은 출력 파일 형식을 지정해 주며 3GP를 설정해 주었다. `setOutputFile()` 메서드를 지정하기 위해 앞서 만들어 두었던 fileName을 지정해 준다. 다음으로 `setAudioEncoder()`를 사용해 오디오 데이터를 인코딩하는 데 사용할 오디오 인코더로 AMR_NB를 설정해 주었다. 

위 설정을 통해 MediaRecorder를 실행할 준비를 마쳤다는 것을 명시해 주기 위해 `prepare()` 메서드를 사용한다. 해당 메서드는 만에 하나 실패할 수도 있기 때문에 try~catch 문에 작성해 준다.

준비하는 과정도 무리 없이 통과했다면 `start()`를 통해 MediaRecorder를 사용한다.

<b>MainActivity.kt</b>

```kotlin
private fun startRecording() {
    recorder = MediaRecorder().apply {
        setAudioSource(MediaRecorder.AudioSource.MIC)
        setOutputFormat(MediaRecorder.OutputFormat.THREE_GPP)
        setOutputFile(fileName)
        setAudioEncoder(AudioEncoder.AMR_NB)

        try {
            prepare()
        } catch (e: IOException) {
            Log.e("APP", "mediaRecorder failed $e")
        }

        start()
    }
}
```

추가적으로 오디오 버튼을 눌러 녹음을 실행했을 때 버튼 이미지가 정지 모양으로 바뀌도록 구현해 보도록 하고 재생 버튼을 비활성화 시키는 효과를 추가해 주었다.

```kotlin
private fun startRecording() {
    
    ...

    binding.recordButton.setImageDrawable(
        ContextCompat.getDrawable(this, R.drawable.ic_baseline_stop_24)
    )
    binding.recordButton.imageTintList = ColorStateList.valueOf(Color.BLACK)
    binding.playButton.apply {
        isEnabled = false
        alpha = 0.3f
    }
}
```

위 과정에서 녹음 버튼은 두 가지 기능을 가지고 있다. 녹음을 시작하는 기능과 정지하는 두 가지 기능을 가지고 있고 해당 앱은 크게 '일반', '재생', '녹음' 이렇게 세 가지 상태로 나눌 수 있다. 따라서 각각의 상태를 구분하기 쉽도록 enum class 생성하도록 한다. 클래스 내부에 enum class `State`를 생성해 주었다. 세 가지 상태의 이름은 각각 'RELEASE', 'RECORDING', 'PLAYING' 이다.

```kotlin
private enum class State {
    RELEASE, RECORDING, PLAYING
}
```

enum class를 생성하였으니 구분을 좀 더 쉽게 해주도록 하자. `onRecord()`라는 메서드를 생성하여 녹음 기능을 좀 더 세분화하여 구분해 주도록 한다. 인자의 값으로 true가 들어왔을 때에는 녹음 기능을 시작한다는 의미로 앞서 작성한 `startRecording()`을 호출하도록 하고 false 값이 들어왔을 경우 녹음 기능 정지를 의미하므로 `stopRecording()` 메서드를 만들어 준다.

```kotlin
private fun onRecord(start: Boolean) = if (start) startRecording() else stopRecording()
```

`stopRecording()` 메서드는 생성한 MediaRecorder의 녹음 중인 상태를 정지시키고 해제하면 된다. 또한 위해서 추가적으로 설정해 준 재생 버튼을 다시 활성화시켜주고 녹음 버튼의 이미지 또한 원상태로 돌려놓는다.

```kotlin
private fun stopRecording() {
    recorder?.apply {
        stop()
        release()
    }
    recorder = null

    binding.recordButton.setImageDrawable(
        ContextCompat.getDrawable(this, R.drawable.ic_baseline_fiber_manual_record_24)
    )
    binding.recordButton.imageTintList = ColorStateList.valueOf(getColor(R.color.red))
    binding.playButton.apply {
        isEnabled = true
        alpha = 1f
    }
}
```

또한 "recordButton"을 클릭했을 때 동작을 when 분기문으로 나누어 처리하도록 한다. 'RELEASE' 상태에서 녹음 버튼을 클릭했다면 녹음 시작을, 'RECORDING' 상태에서 클릭했다면 녹음 정지를 각각 실행함을 명시한다. 또한 `checkAudioPermissionAndRecord()`와 `onRequestPermissionResult()` 메서드에서 허용된 상태일 때의 구현할 동작으로 onRecord(true)를 넣어 녹음을 시작하도록 지정해 준다.

```kotlin
binding.recordButton.setOnClickListener {
    when (state) {
        State.RELEASE -> checkAudioPermissionAndRecord()
        State.RECORDING -> onRecord(false)
        State.PLAYING -> {}
    }
}

    private fun checkAudioPermissionAndRecord() {
        when {
            ContextCompat.checkSelfPermission(
                this,
                AUDIO_REQUEST_PERMISSION
            ) == PackageManager.PERMISSION_GRANTED -> {
                onRecord(true)
            }
            ActivityCompat.shouldShowRequestPermissionRationale(
                this,
                AUDIO_REQUEST_PERMISSION
            ) -> {
                showPermissionRationalDialog()
            }
            else -> {
                ActivityCompat.requestPermissions(
                    this,
                    arrayOf(AUDIO_REQUEST_PERMISSION),
                    AUDIO_REQUEST_PERMISSION_CODE
                )
            }
        }
    }

        override fun onRequestPermissionsResult(
        requestCode: Int,
        permissions: Array<out String>,
        grantResults: IntArray
    ) {
        super.onRequestPermissionsResult(requestCode, permissions, grantResults)

        val audioRecordPermissionGranted =
            requestCode == AUDIO_REQUEST_PERMISSION_CODE && grantResults.firstOrNull() == PackageManager.PERMISSION_GRANTED

        if (audioRecordPermissionGranted) {
            onRecord(true)
        } else {
            if (shouldShowRequestPermissionRationale(AUDIO_REQUEST_PERMISSION)) {
                showPermissionRationalDialog()
            } else {
                showPermissionDisallowDialog()
            }
        }
    }
```

<br><br>

## 📔 녹음 기능 구현

녹음된 파일을 출력해 보도록 하자. 녹음 기능은 MediaRecorder를 사용해서 구현했듯이 재생 기능은 MediaPlayer를 사용해서 구현한다.

우선 클래스 변수로 MediaPlayer를 정의해 준다.

<b>MainActivity.kt</b>
```kotlin
private var player: MediaPlayer? = null
```

그 다음 "recordButton"을 클릭했을 때와 마찬가지로 클릭 리스너를 통해 각각의 State에 대응해 동작을 구현시켜 준다. "playButton"은 '일반 상태', 즉 `State.RELEASE` 상태에서만 동작하므로 `onPlay()` 메서드로 구현될 것이기 때문에 해당 메서드에 true값을 넣어준다.

"stopButton"은 '재생' 상태, `State.PLAYING`일 때 정지 기능을 수행하기 위해 `onPlay()` 메서드에 false 값을 넣어준다.

```kotlin
override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    binding = ActivityMainBinding.inflate(layoutInflater)
    setContentView(binding.root)

    ...

    binding.playButton.setOnClickListener {
        when (state) {
            State.RELEASE -> onPlay(true)
            else -> {}
        }
    }

    binding.stopButton.setOnClickListener {
        when (state) {
            State.PLAYING -> onPlay(false)
            else -> {}
        }
    }
}
```

두 가지 상태를 분류해서 `startPlaying()`과 `stopPlaying()` 메서드로 동작하도록 하는 `onPlay()` 메서드의 내용은 다음과 같다.

```kotlin
private fun onPlay(start: Boolean) = if (start) startPlaying() else stopPlaying()
```

녹음된 파일 재생을 위해 먼저 `startPlaying()` 메서드를 구현해 보자. 가장 "state" 변수의 값을 재생을 시작했으므로 `State.PLAYING`으로 변환해 주고 MediaPlayer를 생성한다. MediaPlyaer에서 `setDataSource()` 메서드를 사용해 fileName을 인자로 넘겨 재생될 파일을 설정한다. 그다음 `prepare()` 메서드로 준비됨을 명시한다. 해당 메서드들은 예외 처리를 위해 try~catch문으로 감싸주었다.

재생이 완료되었다면 자동으로 `stopPlaying()` 메서드를 실행시킬 필요가 있다. 이를 위한 콜백 메서드인 `setOnCompletionListener()`를 호출해 주도록 한다. 해당 메서드 내부에 `stopPlaying()`을 호출한다.

또한 재생 중인 상태에서는 녹음 기능을 비활성화해주도록 한다.

```kotlin
private fun startPlaying() {
    player = MediaPlayer().apply {
        try {
            setDataSource(fileName)
            prepare()
        } catch (e: IOException) {
            Log.e("APP", "MediaPlayer failed $e")
        }
        start()
    }

    player?.setOnCompletionListener {
        stopPlaying()
    }

    binding.recordButton.apply {
        isEnabled = false
        alpha = 0.3f
    }

    state = State.PLAYING
}
```

`stopPlaying()` 메서드에서는 재생 중인 파일을 정지하도록 구현한다. MediaPlayer를 중지시키고 해제해 주도록 하며 "recordButton"을 다시 활성화시켜 주도록 한다. 마지막으로 "state"의 값을 `State.RELEASE`로 바꿔준다.

```kotlin
private fun stopPlaying() {
    player?.release()
    player = null

    binding.recordButton.apply {
        isEnabled = true
        alpha = 1f
    }

    state = State.RELEASE
}
```

결과를 보면, 녹음 기능을 먼저 수행한 뒤 재생 버튼을 누르면 녹음된 파일이 재생되는 것과 재생이 완료되면 자동으로 정지 상태로 변환되거나 재생 중일 때 정지 버튼을 눌러 정지 상태로 변환되는 것을 볼 수 있다.

<br><br>

## 📔 녹음 파형 구현

저장된 녹음 파일의 소리를 파형으로 시각화 시켜보도록 하자. 이를 위해서는 커스텀 뷰를 구현해야 하기 때문에 View를 상속받는 WaveformView라는 이름의 클래스를 생성해 준다. View 생성자에 필요한 세 개의 매개변수를 인자로 넣어주었다.

<b>WaveformView.kt</b>

```kotlin
class WaveformView @JvmOverloads constructor (
    context: Context,
    attrs: AttributeSet? = null,
    defStyleAttr: Int = 0
) : View(context, attrs, defStyleAttr) {}
```

파형을 시각화하기 위해 사각형을 그려줄 것이다. "rectF"라는 이름의 RectF형 변수를 하나 만들어 준다. RectF는 좌표를 기준으로 모양을 설정한다. 기본값은 (0, 0)이다. 이와 더불어 채색을 위해 Paint형의 "redPaint" 변수를 만들어 주었다. 

```kotlin
private val rectF = RectF().apply {
    top = 30f
    left = 20f
    bottom = top + 60f
    right = left + 30f
}

private val redPaint = Paint().apply {
    color = Color.RED
}
```

위의 설정을 토대로 `onDraw()` 메서드를 재정의하여 Canvas의 `drawRect(rectF, redPaint)`
를 실행시키면 아래와 같이 빨간색의 사각형이 WaveformView 위에 그려진 것을 볼 수 있다.

```kotlin
override fun onDraw(canvas: Canvas?) {
    super.onDraw(canvas)

    canvas?.drawRect(rectF, redPaint)
}
```

![](/assets/images/fastcampus/part2/recorder/recorder6.png)

위 사각형을 MediaRecorder에서 입력되는 소리의 크기에 맞게 출력해야 하므로 `addAmplitude()` 메서드에 Float 타입의 maxAmplitude 값을 가져와 사각형의 길이를 정하는데 사용하도록 한다.
메서드의 마지막에 `invalidate()` 메서드를 호출하여 사각형을 다시 그리도록 한다.

```kotlin
fun addAmplitude(maxAmplitude: Float) {
    rectF.top = 0f
    rectF.bottom = maxAmplitude
    rectF.left = 0f
    rectF.right = 20f

    invalidate()
}
```

이로써 입력되는 값을 시각화하기 위한 사각형을 그릴 준비는 마쳤다. 이제 MainActivity에서 사각형을 그릴 수 있도록 소리의 입력값을 보내주면 된다. 그전에 한 가지 고려해야 할 사항이 있다. 입력값을 보낼 수 있는 `startRecording()` 함수 내에서 그대로 WaveformView의 `addAmplitude()` 메서드를 호출해 전달해 주면 값이 한 번만 넘어간다. MediaRecorder를 위한 스레드는 이미 따로 동작하고 있지만 해당 메서드는 그렇지 않다. 따라서 주기적으로 값을 전달해 줄 스레드를 명시적으로 선언해 줘야 한다. 이를 위해 Timer 클래스를 새로 하나 생성해 준다.

Timer 클래스의 변수로 Handler 하나와 Runnable 인터페이스를 하나 생성해 준다. Runnable 인터페이스 내부의 `run()` 메서드에서는 추후 녹음 시간을 나타내는 변수인 "duration" 값의 증가와 handler 실행 메서드를 넣어준다. 또한 `run()` 메서드의 실행과 정지를 위한 `start()`와 `stop()` 메서드도 클래스 내에 추가해 주었다.

<b>Timer.kt</b>

```kotlin
class Timer() {
    private var duration: Long = 0L
    private val handler = Handler(Looper.getMainLooper())
    private val runnable: Runnable = object: Runnable {
        override fun run() {
            duration += 100L
            handler.postDelayed(this, 100L)
        }
    }

    fun start() {
        handler.postDelayed(runnable, 100L)
    }

    fun stop() {
        handler.removeCallbacks(runnable)
    }
}
```

다음으로 인터페이스를 사용해 MainActivity에서의 동작을 Thread에서 실행시킬 수 있도록 리스너를 등록한 뒤 연결한다.

```kotlin
class Timer(listener: OnTimerTickListener) {
    private var duration: Long = 0L
    private val handler = Handler(Looper.getMainLooper())
    private val runnable: Runnable = object: Runnable {
        override fun run() {
            duration += 40L
            handler.postDelayed(this, 40L)
            listener.onTick(duration)
        }
    }


    fun start() {
        handler.postDelayed(runnable, 40L)
    }

    fun stop() {
        handler.removeCallbacks(runnable)
    }
}

interface OnTimerTickListener {
    fun onTick(duration: Long)
}
```

이제 MainActivity에서 인터페이스를 상속받은 뒤 메서드를 재정의해 입력된 소리의 값을 인자로 넘겨주고 `startRecording()`과 `stopRecording()` 메서드에서 각각 타이머의 시작과 정지를 명시해 준다.

<b>MainActivity.kt</b>

```kotlin
class MainActivity : AppCompatActivity(), OnTimerTickListener {

    private lateinit var timer: Timer

    ...

    override fun onCreate(savedInstanceState: Bundle?) {
        
        ...

        timer = Timer(this)
        
        ...
    }

    private fun startRecording() {
        
        ...

        timer.start()

        ...

    }

    private fun stopRecording() {
        
        ...

        timer.stop()

        ...

    }

    ...

    override fun onTick(duration: Long) {
        binding.waveformView.addAmplitude(recorder?.maxAmplitude?.toFloat() ?: 0f)
    }

    ...

}
```

결과를 보면 소리가 입력되는 값을 시각화해서 나타내는 것을 볼 수 있다.

![](/assets/images/fastcampus/part2/recorder/recorder7.jpg)


하지만 보통 녹음기 UI를 보면 오른쪽에서 왼쪽으로 흐르는 듯하게 구현하기 때문에 좀 더 다듬어야 한다. 먼저 WaveformView에서 `addAmplitude()` 메서드를 수정해 보도록 하자. 클래스 변수로 입력값과 구현된 사각형들을 리스트에 담아두기 위해 각각 "ampList"와 "rectList" 변수를 정의해 두었다.

<b>WaveformView.kt</b>

```kotlin
class WaveformView @JvmOverloads constructor(
    context: Context,
    attrs: AttributeSet? = null,
    defStyleAttr: Int = 0
) : View(context, attrs, defStyleAttr) {

    private val ampList = mutableListOf<Float>()
    private val rectList = mutableListOf<RectF>()

    ...

}
```

`addAmplitude()` 메서드에서 매개변수로 받아온 `maxAmplitude` 값을 "ampList" 변수에 추가해 주고, "rectList"는 `clear()`하여 초기화 시켜 준다. 여기서 초기화하는 이유는 계속해서 추가되어 쌓이는 입력값들에 맞게 초기화 후 다시 그려줘야 하기 때문이다. 이후 사각형의 가로와 기기의 화면에 들어갈 수 있는 사각형의 가로의 수를 변수로 지정한 뒤, 최근 입력값의 수에 맞추어 사각형을 다시 그리도록 한다.

```kotlin
fun addAmplitude(maxAmplitude: Float) {

    ampList.add(maxAmplitude)
    rectList.clear()

    val rectWidth = 10f
    val maxRect = (this.width / rectWidth).toInt()

    val amps = ampList.takeLast(maxRect)

    for ((i, amp) in amps.withIndex()) {
        RectF().apply {
            top = 0f
            bottom = amp
            left = i * rectWidth
            right = left + rectWidth

            rectList.add(this)
        }
    }
    invalidate()
}
```

결과를 보면 파형이 우측에서 좌측으로 흘러가는 것처럼 효과가 나타나는 것을 볼 수 있다.

![](/assets/images/fastcampus/part2/recorder/recorder8.jpg)
![](/assets/images/fastcampus/part2/recorder/recorder9.jpg)

<br><br>

## 📔 타이머 시간 구현

녹음 시간을 표시하는 텍스트 뷰를 구현해 보도록 하자.

MainActivity의 `onTick()` 메서드에서 밀리초, 초, 분 단위를 나타내는 변수에 맞게 해당 값을 지정해 준다. 이후 "timerTextView"의 text에 해당 값들을 넣어 주도록 한다.

![](/assets/images/fastcampus/part2/recorder/recorder11.jpg)
![](/assets/images/fastcampus/part2/recorder/recorder10.jpg)


<br><br>

## 📔 재생 기능 파형 시각화 구현

이번에는 녹음된 파일을 재생했을 떄의 시각화 또한 구현해 보자. 

WaveformView에서 녹음 파일 재생을 위해 "tick"이라는 이름의 변수를 정의해 준다.

<b>WaveformView.kt</b>

```kotlin
private var tick = 0
```

그리고 재생을 위해 `replayAmplitude()` 메서드를 생성해 주었다. 메서드의 내용은 `addAmplitude()` 메서드와 크게 다르지 않지만 한 가지 다른 점이 있다. 저장된 파일을 처음부터 재생시키기 때문에 중간에 `take()` 함수를 가져와 인덱스를 0부터 시작해 해당 값을 1씩 올리면서 그리기를 진행시킨다. 여기서 인덱스를 표시하는 값이 앞서 설정한 "tick" 변수이다. 이렇게 하면 파일의 처음부터 끝까지 녹음된 파일의 파형을 재생할 수 있다.

```kotlin
fun replayAmplitude() {
    rectList.clear()

    val maxRect = (this.width / rectWidth).toInt()
    val amps = ampList.take(tick).takeLast(maxRect)

    for ((i, amp) in amps.withIndex()) {
        RectF().apply {
            top = 0f
            bottom = amp
            left = i * rectWidth
            right = left + rectWidth

            rectList.add(this)
        }
    }

    tick++

    invalidate()
}
```

추가적으로 `clearData()`와 `clearWave()` 메서드를 정의하여 각각 녹음 기능을 실행할 때에는 기존의 녹음된 파일을 초기화하여 다시 녹음될 수 있도록, 그리고 재생 기능을 실행할 때 녹음되었거나 재생되었던 파일의 파형을 삭제하여 재생 시 다시 파형을 그릴 수 있도록 설정해 주었다.

이후 MainActivity의 `onTick()` 메서드에서 State의 값에 따라 `replayAmplitude()`와 `addAmplitude()` 메서드가 실행될 수 있도록 구분해 준다.

```kotlin
override fun onTick(duration: Long) {

    ...

    if (state == State.PLAYING) {
        binding.waveformView.replayAmplitude()
    } else if (state == State.RECORDING) {
        binding.waveformView.addAmplitude(recorder?.maxAmplitude?.toFloat() ?: 0f)
    }
}
```

또한 위에서 작성했던 `clearData()`와 `clearWave()` 메서드를 각각 `startRecording()`과 `startPlaying()` 메서드의 "timer" 시작 전에 지정해야 초기화와 동시에 새로 기능을 수행할 수 있다.

<b>MainActivity.kt</b>

```kotlin
private fun startRecording() {
   
    ...

    binding.waveformView.clearData()
    timer.start()


    ...

}

...

private fun startPlaying() {
    
    ...

    binding.waveformView.clearWave()
    timer.start()

    ...

}

```

![](/assets/images/fastcampus/part2/recorder/recorder12.jpg)
![](/assets/images/fastcampus/part2/recorder/recorder13.jpg)


<br><br>

## 📔 파형 다듬기

파형의 모양을 좀 더 보기좋게 만들어 보자.

WaveformView에서 `addAmplitude()` 메서드 내부에서 "amplitude"라는 변수의 값을 "ampList"에 추가한다. 이렇게 하면 기존 크지 않은 소리에도 소리가 최대로 표시된 점을 개선하여 표시한다. 이제 RectF를 그리는 코드에서 top과 bottom의 값을 변경시켜 준다. 기기의 높이를 이등분하고 여기에 입력된 소리의 값을 이등분하여 빼거나 더한 값을 각각 top과 bottom에 지정해 주면 WaveformView가 대칭된 형태로 파형을 나타내는 것을 볼 수 있다.

또한 각각의 파형 간에 여백을 주기 위해 "rectWidth"의 값을 15f로 변경해 주고 RectF를 그리는 코드의 right 속성값을 "left - (rectWidth - 5)"를 해주면 파형 간 여백을 만들어 생성되는 것을 볼 수 있다.

<b>WaveformView.kt</b>

```kotlin
class WaveformView @JvmOverloads constructor(
    context: Context,
    attrs: AttributeSet? = null,
    defStyleAttr: Int = 0
) : View(context, attrs, defStyleAttr) {

    ...

    private val rectWidth = 15f
    
    ...

    fun addAmplitude(maxAmplitude: Float) {
        val amplitude = (maxAmplitude / Short.MAX_VALUE) * this.height * 0.8f

        ampList.add(amplitude)
        
        ...

        for ((i, amp) in amps.withIndex()) {
            RectF().apply {
                top = (this@WaveformView.height / 2) - amp / 2
                bottom = (this@WaveformView.height / 2) + amp / 2
                left = i * rectWidth
                right = left + (rectWidth - 5)

                rectList.add(this)
            }
        }
        invalidate()
    }

    fun replayAmplitude() {

        ...

        for ((i, amp) in amps.withIndex()) {
            RectF().apply {
                top = (this@WaveformView.height / 2) - amp / 2
                bottom = (this@WaveformView.height / 2) + amp / 2
                left = i * rectWidth
                right = left + (rectWidth - 5)

                rectList.add(this)
            }
        }

        ...

    }

    ...

}
```

![](/assets/images/fastcampus/part2/recorder/recorder14.jpg)

<br>

## 📔전체 코드
<https://github.com/Becomeproo/audio_recorder_app>

<br>

## 📔마무리
여전히 스레드를 사용하는 것은 어렵지만 저번보다는 훨씬 이해하는데 어려움이 덜 했던 것 같다. 나름 정리한 보람이 있다는 생각이 들었고 커스텀 뷰를 구현하는 데 있어서 아직은 아리송하지만 계속 반복하면서 잘 다루고 싶다는 생각이 들었다. 또한 enum class를 적절히 잘 활용하면 더욱 효율적으로 작업할 수 있다는 것을 알았다.