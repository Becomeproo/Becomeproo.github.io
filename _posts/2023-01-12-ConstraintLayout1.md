---
title:  "[Android] ConstraintLayout 정리(1)"

categories:
  - Android
tags:
  - [Android, Android 정리, ConstraintLayout]

toc: true
toc_sticky: true
 
date: 2023-01-12
last_modified_at: 2023-01-12
---

<br>
<br>
<br>

> 아래 내용은 [Android Developer 공식문서](https://developer.android.com/training/constraint-layout?hl=ko)와 [recipes4dev님의 블로그](https://recipes4dev.tistory.com/158) 내용을 기반으로 작성한 자습 기록입니다.

<br>
<br>


# 📚ConstraintLayout 정리

## 📔ConstraintLayout 개요

저번에 LinearLayout에 이어 이번에는 ConstraintLayout에 대해서 정리해 보고자 한다.
ConstraintLayout은 제약조건을 추가하여 view를 배치할 수 있도록 해주는 레이아웃으로, ConstraintLayout을 사용하면 유연하고 세부적인 레이아웃을 만들 수 있다. 

ConstraintLayout을 더 쉽게, 그리고 더 잘 사용하기 위해 사전적 정의를 먼저 찾아보도록 하자.
constraint란 "제약, 조건" 등의 의미를 가지고 있다. 말 그대로 `TextView`, `Button`과 같은 View들에 제약을 걸어 각각의 View의 위치와 크기를 결정한다는 것을 느낄 수 있다.

<br>

## 📔ConstraintLayout 추가

ConstraintLayout을 사용하기 위해서는 다음과 같은 종속이 필요한데, 기본적으로 추가되어 있다.

### 📖build.gradle(module)
```kotlin
repositories {
    google()
}

...

dependencies {
    implementation("androidx.constraintlayout:constraintlayout:2.2.0-alpha05")
    // To use constraintlayout in compose
    implementation("androidx.constraintlayout:constraintlayout-compose:1.1.0-alpha05")
}
```

<br>

## 📔ConstraintLayout 기본 사용
ConstraintLayout은 앞서 언급했듯이 제약을 걸어 위치와 크기를 결정한다. 이해를 돕기 위해 아래 코드를 보자.

```kotlin
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:orientation="vertical"
    android:weightSum="50">

    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="textView"
        android:textSize="24sp"
        android:textStyle="bold"
        android:textColor="@color/black"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toEndOf="parent" />
    
</androidx.constraintlayout.widget.ConstraintLayout>
```

![contraint1](/assets/images/android/constraintlayout/constraint1.png)

간단한 텍스트 뷰를 하나 만들어 주었고 끝부분과 하단 부분에 제약을 주었다. 여기서 `parent`는 `ConstraintLayout`을 의미한다. 이는 즉, 텍스트 뷰의 위치를 `ConstraintLayout`의 끝과 하단에 맞추겠다는 의미이다. 이미지에도 텍스트 뷰가 그에 맞게 위치해 있는 것을 볼 수 있다.

이러한 방법으로 view에 제약을 지정함으로써 위치를 설정한다.

<br>

## 📔ConstraintLayout 제약

안드로이드는 view의 위치와 크기를 유연하게 지정할 수 있도록 다양한 종류의 제약을 제공한다.
제약의 종류는 다음과 같다.

### 📖ConstLayout 제약의 종류

 * Relative Positioning
 * Margins
 * Centering positioning
 * Circular positioning
 * Visibility behavior
 * Dimension constraints
 * Chains
 * Virtual Helpers objects
 * Optimizer

 각각의 종류에 대해서 계속해서 나누어서 정리해 볼 예정이다.