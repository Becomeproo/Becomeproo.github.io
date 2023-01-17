---
title:  "[Android] ConstraintLayout 정리(5) - Circular Positioning"

categories:
  - Android
tags:
  - [Android, Android 정리, ConstraintLayout]

toc: true
toc_sticky: true
 
date: 2023-01-17 16:01:45+0900
last_modified_at: 2023-01-17 16:01:48+0900

---

<br>
<br>
<br>

> 아래 내용은 [Android Developer 공식 문서](https://developer.android.com/training/constraint-layout?hl=ko)와 [recipes4dev님의 블로그](https://recipes4dev.tistory.com/158), [EMU(Eastern Mediterranean University) 제공 문서](https://staff.emu.edu.tr/mobinabeheshti/Documents/courses/ITEC399/lecture%205%20-%20399.pdf)] 내용을 참고하여 작성한 자습 기록입니다.

<br>
<br>

# 📚ConstraintLayout 정리(5) - Circular Positioning

## 📔Circular Positioning
ConstraintLayout은 상, 하, 좌, 우 위치 지정 외에도 좀 더 유연한 배치를 위해 원형 위치 지정을 제공한다.

![](/assets/images/android/constraintlayout/circle1.png)

Circular Positioning, 원형 위치 지정이란 중심점이 되는 뷰를 기준으로 특정 각도 및 거리에 따라 또 다른 뷰를 배치할 수 있도록 하는 제약이다. 

### 📖Circular Positioning의 속성
Circular Positioning의 속성은 아래와 같이 세 가지이다.

|**속성명**|설명|
|:---:|:---|
|`layout_constraintCircle`|대상 뷰 위젯 지정|
|`layout_constraintCircleRadius`|대상 뷰 위젯의 중심과 생성된 뷰 위젯의 중심 사이의 거리|
|`layout_constraintCircleAngle`|생성된 뷰 위젯이 배치될 각도 (0 ~ 360)|

<br>

코드를 보면 다음과 같다. `button1`을 중심점이 되는 뷰로 잡고(layout_constraintCircle),해당 뷰와의 거리는 300만큼(layout_constraintCircleRadius), 각도는 30의 위치만큼(layout_constraintCircleAngle) 배치되도록 했다. 해당 과정에서 상, 하, 좌, 우로 제약을 같이 걸었지만, Circular Positioning의 값이 우선으로 적용된 것을 볼 수 있다.

```kotlin
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">

    <Button
        android:id="@+id/button1"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="button1"
        app:layout_constraintTop_toTopOf="parent"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent" />

    <Button
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:id="@+id/button2"
        android:text="button2"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintCircle="@id/button1"
        app:layout_constraintCircleRadius="300dp"
        app:layout_constraintCircleAngle="30" />

</androidx.constraintlayout.widget.ConstraintLayout>
```

![](/assets/images/android/constraintlayout/circle2.png)