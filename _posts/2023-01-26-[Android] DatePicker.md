---
title:  "[Android] DatePicker, DatePickerDialog"

categories:
  - Android
tags:
  - [Android, Android 정리, DatePicker, DatePickerDialog]

toc: true
toc_sticky: true
 
date: 2023-01-26 15:10:16+0900
last_modified_at: 2023-01-26 15:10:20+0900

---

<br>
<br>
<br>

> 아래 내용은 [Android Developer 공식 문서](https://developer.android.com/reference/androidx/leanback/widget/picker/DatePicker) 내용을 참고하여 작성한 자습 기록입니다.

<br>
<br>

# 📚Android 정리 - DatePicker, DatePickerDialog

DatePicker는 날짜를 좀 더 보기 쉽고 편하게 입력하고자 할 때 사용한다. DatePickerDialog 또한 마찬가지로 날짜 값을 받아온다. 오늘은 이 두 가지를 어떻게 사용하는지에 대해 알아보고, 차이점이 무엇인지에 대해 알아보자. 

## 📔 DatePicker

구현이 크게 어렵지 않다. xml 코드에서 `DatePicker` 위젯을 생성하면 다음과 같이 화면에 구현되는 것을 볼 수 있다. 여기서 `android:datePickerMode` 속성 값으로 "spinner" 값을 지정해 주면 `spinner`로 날짜를 선택할 수 있게 되고 그 옆에 달력이 표시된다. 만약 달력이 없는 `spinner`를 사용하고 싶다면 `android:calendarViewShown` 속성 값으로 "false"를 지정해 주면 된다.

```kotlin
<DatePicker
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    app:layout_constraintStart_toStartOf="parent"
    app:layout_constraintEnd_toEndOf="parent"
    app:layout_constraintTop_toTopOf="parent"
    app:layout_constraintBottom_toBottomOf="parent" />
```

```
    <DatePicker
        ...
        android:datePickerMode="spinner"
        ... />
```

![](/assets/images/android/datepicker/datepicker1.png) ![](/assets/images/android/datepicker/datepicker3.png)


`DatePicker`와 관련된 속성값들을 추가해 날짜 선택에 관해 더 구체적인 설정을 할 수 있다.

<br>

### 📖 DatePicker로부터 값 얻어오기

값을 가져오기 앞서 `DatePicker`에서 날짜를 선택하고 버튼을 누르면 텍스트 뷰에 띄워주는 형식으로 다음과 같이 간단한 UI를 구성하였다.

```kotlin
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">

    <TextView
        android:id="@+id/dateTextView"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="00월 0월 0일"
        android:textSize="48sp"
        app:layout_constraintBottom_toTopOf="@id/datePicker"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent" />

    <Button
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:layout_marginHorizontal="16dp"
        android:text="날짜 표시"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toEndOf="@id/datePicker"
        app:layout_constraintStart_toStartOf="@id/datePicker"
        app:layout_constraintTop_toBottomOf="@id/datePicker" />

    <DatePicker
        android:id="@+id/datePicker"
        android:layout_width="350dp"
        android:layout_height="400dp"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toBottomOf="@id/dateTextView" />

</androidx.constraintlayout.widget.ConstraintLayout>
```

![](/assets/images/android/datepicker/datepicker2.png)

클래스 파일에서 이제 값을 가져오는 코드를 구현해 보자. 정말 어렵지 않다. `findViewById`를 사용해 텍스트 뷰와 버튼을 가져오고 버튼을 클릭했을 때, `DatePicker`을 가져와 변수로 지정해 준 뒤, 텍스트 뷰에 `DatePicker`의 "year", "month", "dayOfMonth"의 값을 가져와 표현식으로 지정해 주었다. 여기서 주의해야 할 점은 "month" 값을 지정할 때 +1을 해야 한다는 것이다.

코드의 결과를 보면 버튼을 눌렀을 때, `DatePicker`의 값을 가져와 텍스트 뷰에 표시한 것을 볼 수 있다.

```kotlin

class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        val textView = findViewById<TextView>(R.id.dateTextView)
        val button = findViewById<Button>(R.id.button)

        button.setOnClickListener {
            val datePicker = findViewById<DatePicker>(R.id.datePicker)
            textView.text = "${datePicker.year}년 ${datePicker.month + 1}월 ${datePicker.dayOfMonth}일"
        }
    }
}
```

![](/assets/images/android/datepicker/datepicker4.jpg)

## 📔 DatePickerDialog를 활용하여 날짜 받아오기

이번에는 `DatePickerDialog`를 활용해서 날짜를 받아오도록 해보자. `DatePicker`와의 차이점은 `DatePicker`는 UI에 표시되어 있는 달력을 통해 날짜를 선택했다면, `DatePickerDialog`는 특정 이벤트를 통해 생성된 다이얼로그에서 날짜를 입력해 값을 받는다.

우선 아래와 같이 간단한 xml 파일을 구현해 주었다. 버튼을 눌렀을 때, `DatePickerDialog`가 생성되고 해당 다이얼로그에서 지정한 날짜를 텍스트 뷰에 표시한다. 코드는 아래와 같다.

```kotlin
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">

    <TextView
        android:id="@+id/dateTextView"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="00월 0월 0일"
        android:textSize="48sp"
        app:layout_constraintBottom_toTopOf="@id/button"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent" />

    <Button
        android:id="@+id/button"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:layout_marginHorizontal="16dp"
        android:text="날짜 설정하기"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toEndOf="@id/dateTextView"
        app:layout_constraintStart_toStartOf="@id/dateTextView"
        app:layout_constraintTop_toBottomOf="@id/dateTextView" />

</androidx.constraintlayout.widget.ConstraintLayout>
```

클래스 파일에서 버튼 클릭 리스너를 통해 다이얼로그를 띄우도록 한다. 클릭 리스너에서 `Calendar.getInstance()`를 사용해 캘린더 객체를 변수에 지정해 준다. 이후 연월일에 대한 각각의 값을 `calendar.get()`을 활용하여 `Calendar.YEAR`, `Calendar.MONTH`, `Calendar.DAY_OF_MONTH`으로 값을 가져와 `DatePickerDialog`에 넣어준다. 여기서 생성자에 들어갈 인자로 `OnDateSetListener`를 지정해 주어야 한다. 아래와 같이 텍스트 뷰에 지정된 날짜 정보를 띄워주도록 구현하였다. 해당 리스너는 람다 함수뿐만 아니라 익명 함수, 그리고 해당하는 함수를 만들고 해당 함수의 레퍼런스 값을 넣어 구현해도 된다.

```kotlin
val listener = DatePickerDialog.OnDateSetListener { _, year, month, day ->
            textView.text = "${year}년 ${month + 1}월 ${day}일"
}

/* 익명 함수
val listener = DatePickerDialog.OnDateSetListener(fun(datePicker: DatePicker, year, month, day) {
    textView.text = "${year}년 ${month + 1}월 ${day}일"
})

레퍼런스 참조에 활용될 함수
fun dialogFunc(datePicker: DatePicker, year: Int, month: Int, day: Int) {
    textView.text = "${year}년 ${month + 1}월 ${day}일"
}
*/
```

구현한 리스너를 `DatePicker`에 인자로 넣어준 뒤, 반드시 `.show()`를 실행해 주어야 한다.

```kotlin
button.setOnClickListener {
    val calendar = Calendar.getInstance()
    val year = calendar.get(Calendar.YEAR)
    val month = calendar.get(Calendar.MONTH)
    val day = calendar.get(Calendar.DAY_OF_MONTH)

    DatePickerDialog(this, listener, year, month, day).show()
}
```

이후 코드를 실행하면, 원하는 대로 `DatePickerDialog`의 값을 잘 받아온 것을 확인할 수 있다.

![](/assets/images/android/datepicker/datepicker6.jpg) ![](/assets/images/android/datepicker/datepicker5.jpg)
