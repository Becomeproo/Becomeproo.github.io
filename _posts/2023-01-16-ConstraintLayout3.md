---
title:  "[Android] ConstraintLayout 정리(3) - Margin"

categories:
  - Android
tags:
  - [Android, Android 정리, ConstraintLayout]

toc: true
toc_sticky: true
 
date: 2023-01-16
last_modified_at: 2023-01-16
---

<br>
<br>
<br>

> 아래 내용은 [Android Developer 공식 문서](https://developer.android.com/training/constraint-layout?hl=ko)와 [recipes4dev님의 블로그](https://recipes4dev.tistory.com/158), [EMU(Eastern Mediterranean University) 제공 문서](https://staff.emu.edu.tr/mobinabeheshti/Documents/courses/ITEC399/lecture%205%20-%20399.pdf)] 내용을 참고하여 작성한 자습 기록입니다.

<br>
<br>

# 📚ConstraintLayout 정리(3) - Margin

## 📔여백(Margin) 지정

![](/assets/images/android/constraintlayout/margin3.png)

margin은 뷰의 여백을 지정할 때 사용하는 속성이다. `android:layout_margin`을 사용해 속성값을 지정하는 데, ConstraintLayout 뿐만 아니라 모든 레이아웃에서 사용할 수 있는 공통 속성이다. 여백의 값은 **<u>양수 값이나 0으로 지정할 수 있으며, 음수 값은 지정할 수 없다.</u>**

ConstraintLayout에서 margin을 사용하기 위해서는 제약이 지정되어 있어야 뷰 간의 여백을 표시할 수 있다. 아래 코드를 통해서 쉽게 이해해 보자.

아래와 같이 제약이 없는 상황에서 여백 값을 설정하면 적용이 되지 않는다.

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
        android:layout_width="130dp"
        android:layout_height="70dp"
        android:text="Button1"
        android:layout_marginStart="30dp" />

</androidx.constraintlayout.widget.ConstraintLayout>
```

![](/assets/images/android/constraintlayout/margin2.png)

반면에, 제약을 추가하면 여백이 생기는 것을 볼 수 있다.

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
        android:layout_width="130dp"
        android:layout_height="70dp"
        android:text="Button1"
        android:layout_marginStart="30dp"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent" />

    <Button
        android:id="@+id/button2"
        android:layout_width="130dp"
        android:layout_height="70dp"
        android:text="Button2"
        android:layout_marginStart="30dp"
        app:layout_constraintTop_toTopOf="@id/button1"
        app:layout_constraintStart_toEndOf="@id/button1" />

</androidx.constraintlayout.widget.ConstraintLayout>
```

![](/assets/images/android/constraintlayout/margin1.png)

<br>

---

<br>

## 📔뷰의 `android:visibility` 속성 값이 `gone` 상태일 때, Margin 사용

제약으로 지정되어 있는 뷰의 `android:visibility` 속성 값이 잠시 사라져 있는 상태일 때, 즉 `gone` 상태일 때 margin 값은 어떻게 될까. 확인해 보자.

앞서 사용했던 코드에서 특별히 변경된 건 없이 `button1`의 visibility 값을 gone으로 지정해 주었고 시작 쪽으로 여백 값을 50dp로 적용했다. 그 상태에서 `button1` 방향으로 제약이 지정되어 있고, 여백 값이 시작 쪽에 30dp로 지정되어 있는 `button2`의 결과는 `button1`이 사라지면서 생긴 공간으로 `button2`의 위치가 이동하였고, 해당 위치에서 여백의 값이 적용된 것을 볼 수 있었다.

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
        android:layout_width="130dp"
        android:layout_height="70dp"
        android:text="Button1"
        android:visibility="gone"
        android:layout_marginStart="50dp"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent" />

    <Button
        android:id="@+id/button2"
        android:layout_width="130dp"
        android:layout_height="70dp"
        android:text="Button2"
        android:layout_marginStart="30dp"
        app:layout_constraintTop_toTopOf="@id/button1"
        app:layout_constraintStart_toEndOf="@id/button1" />

</androidx.constraintlayout.widget.ConstraintLayout>
```

![](/assets/images/android/constraintlayout/margin4.png)

![](/assets/images/android/constraintlayout/margin5.png)

만약, `button2`의 여백 값이 `button1`이 사라지기 전에는 30dp를, 사라진 후에는 `button1`이 가지고 있던 50dp의 값을 적용하길 원한다면, 해당 상태를 지정할 수 있는 별도의 속성을 사용하면 된다.

앞의 예시와 같은 코드를 사용해 보자. `button2`에 `layout_goneMarginStart`의 속성 값을 50dp로 지정해 주면, 이전과 달리 `button1`이 사라진 자리에 위치하면서 여백 값 50dp가 적용된 것을 볼 수 있다.

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
        android:layout_width="130dp"
        android:layout_height="70dp"
        android:text="Button1"
        android:visibility="gone"
        android:layout_marginStart="50dp"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent" />

    <Button
        android:id="@+id/button2"
        android:layout_width="130dp"
        android:layout_height="70dp"
        android:text="Button2"
        app:layout_goneMarginStart="50dp"
        android:layout_marginStart="30dp"
        app:layout_constraintTop_toTopOf="@id/button1"
        app:layout_constraintStart_toEndOf="@id/button1" />

</androidx.constraintlayout.widget.ConstraintLayout>
```

![](/assets/images/android/constraintlayout/margin6.png)

### 📖gone 상태 뷰 대상 여백 값 지정 속성
View.GONE 상대 뷰 대상 속성 값들은 기존 `layout_marginXXX` 속성에서 앞에 `gone`만 붙여주면 된다.

|**속성명**|설명|
|:---:|:---|
|`layout_goneMarginLeft`|뷰의 왼쪽 사이드(Side)를 대상 뷰가 View.GONE 상태일 때 여백|
|`layout_goneMarginTop`|뷰의 상단 사이드(Side)를 대상 뷰가 View.GONE 상태일 때 여백|
|`layout_goneMarginRight`|뷰의 오른쪽 사이드(Side)를 대상 뷰가 View.GONE 상태일 때 여백|
|`layout_goneMarginBottom`|뷰의 하단 사이드(Side)를 대상 뷰가 View.GONE 상태일 때 여백|
|`layout_goneMarginStart`|뷰의 시작 사이드(Side)를 대상 뷰가 View.GONE 상태일 때 여백|
|`layout_goneMarginEnd`|뷰의 끝 사이드(Side)를 대상 뷰가 View.GONE 상태일 때 여백|