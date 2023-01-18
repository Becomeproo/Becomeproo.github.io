---
title:  "[Android] ConstraintLayout 정리(6) - Visibility behavior"

categories:
  - Android
tags:
  - [Android, Android 정리, ConstraintLayout]

toc: true
toc_sticky: true
 
date: 2023-01-18 12:32:28+0900
last_modified_at: 2023-01-18 12:32:33+0900

---

<br>
<br>
<br>

> 아래 내용은 [Android Developer 공식 문서](https://developer.android.com/training/constraint-layout?hl=ko)와 [recipes4dev님의 블로그](https://recipes4dev.tistory.com/158), [EMU(Eastern Mediterranean University) 제공 문서](https://staff.emu.edu.tr/mobinabeheshti/Documents/courses/ITEC399/lecture%205%20-%20399.pdf)] 내용을 참고하여 작성한 자습 기록입니다.

<br>
<br>

# 📚ConstraintLayout 정리(6) - Visibility behavior

이번에는 대상 뷰의 Visibility 속성에 따른 뷰의 위치 변화에 대해서 알아보고자 한다.

View의 Visibility 속성은 3가지 값을 갖고 있다.
* View.VISIBLE
* View.INVISIBLE
* View.GONE

각각의 상태에 따라 뷰가 레이아웃에 표시되는 방법이 모두 다르다. 코드와 함께 보도록 하자.

<br>

## 📔 대상 뷰가 View.VISIBLE 상태인 경우

<br>

대상 뷰의 `android:visibility` 속성 값이 'VISIBLE'일 때에는 일반적으로 뷰가 보이는 방식과 같다. 코드와 함께 확인해 보자.

코드 상에는 버튼 `button1`과 `button2`가 있다. `button1`의 `android:visibility` 속성 값이 `visible`인 경우, View.VISIBLE이 기본 값이므로 해당 속성을 사용하지 않고 코드를 작성했을 때와 차이가 없다. UI 또한 `button1`과 `button2`가 온전히 잘 보이는 것을 볼 수 있다.

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
        android:layout_width="120dp"
        android:layout_height="70dp"
        android:layout_marginStart="30dp"
        android:visibility="visible"
        android:text="button1"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent" />

    <Button
        android:id="@+id/button2"
        android:layout_width="120dp"
        android:layout_height="70dp"
        android:layout_marginStart="30dp"
        android:text="button2"
        app:layout_constraintStart_toEndOf="@id/button1"
        app:layout_constraintTop_toTopOf="@id/button1" />

</androidx.constraintlayout.widget.ConstraintLayout>
```

![](/assets/images/android/constraintlayout/visibility1.png)

<br>

## 📔 대상 뷰가 View.INVISIBLE 상태인 경우

다음으로 INVISIBLE 상태의 경우, 위의 코드에서 `android:visibility` 속성 값만 `INVISIBLE`로 변경해 주었다. 결과를 보면, `button1`이 사라졌지만 `button2`의 위치는 위와 동일한 위치에 배치되어 있는 것을 볼 수 있다. 즉, 해당 상태의 경우 `button1`이 사라지기만 할 뿐 `button1`이 차지하고 있던 공간, `button1`과 `button2`의 제약, 여백 지정에는 변함이 없다.

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
        android:layout_width="120dp"
        android:layout_height="70dp"
        android:layout_marginStart="30dp"
        android:visibility="invisible"
        android:text="button1"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent" />

    <Button
        android:id="@+id/button2"
        android:layout_width="120dp"
        android:layout_height="70dp"
        android:layout_marginStart="30dp"
        android:text="button2"
        app:layout_constraintStart_toEndOf="@id/button1"
        app:layout_constraintTop_toTopOf="@id/button1" />

</androidx.constraintlayout.widget.ConstraintLayout>
```

![](/assets/images/android/constraintlayout/visibility2.png)

<br>

## 📔 대상 뷰가 View.GONE 상태인 경우

<br>

마지막으로 View.GONE 상태의 경우 View.VISIBLE, View.INVISIBLE의 결과와 또 다른 결과가 보이는데, `button1`의 경우 ui 상에서 잠시 사라진 상태가 되고, 앞서 View.INVISIBLE 상태와 달리 `button2`와의 제약, 여백 등도 모두 잠시 사라진 상태가 된다. 이에 따라 `button2`가 `button1`의 위치를 대체하고 해당 위치에서 자신의 여백 값이 적용된 것을 볼 수 있다.

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
        android:layout_width="120dp"
        android:layout_height="70dp"
        android:layout_marginStart="30dp"
        android:visibility="gone"
        android:text="button1"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent" />

    <Button
        android:id="@+id/button2"
        android:layout_width="120dp"
        android:layout_height="70dp"
        android:layout_marginStart="30dp"
        android:text="button2"
        app:layout_constraintStart_toEndOf="@id/button1"
        app:layout_constraintTop_toTopOf="@id/button1" />

</androidx.constraintlayout.widget.ConstraintLayout>
```

![](/assets/images/android/constraintlayout/visibility3.png)

만약 `button1`의 `android:visibility` 속성의 값을 `gone`으로 처리하고 `button2`이 `button1`의 위치를 대체하게 되면서 `button1`이 가지고 있던 레이아웃으로의 여백 값이 적용되길 원한다면, 저번 시간에 살펴보았던 [[뷰의 android:visibility 속성 값이 gone 상태일 때, Margin 사용]](https://becomeproo.github.io/android/ConstraintLayout3/#%EB%B7%B0%EC%9D%98-androidvisibility-%EC%86%8D%EC%84%B1-%EA%B0%92%EC%9D%B4-gone-%EC%83%81%ED%83%9C%EC%9D%BC-%EB%95%8C-margin-%EC%82%AC%EC%9A%A9) 이 부분을 참고하면 될 것이다.