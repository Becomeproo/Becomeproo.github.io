---
title:  "[Android] ConstraintLayout 정리(7) - Dimension constrains"

categories:
  - Android
tags:
  - [Android, Android 정리, ConstraintLayout]

toc: true
toc_sticky: true
 
date: 2023-01-18 14:48:26+0900
last_modified_at: 2023-01-18 14:48:30+0900

---

<br>
<br>
<br>

> 아래 내용은 [Android Developer 공식 문서](https://developer.android.com/training/constraint-layout?hl=ko)와 [recipes4dev님의 블로그](https://recipes4dev.tistory.com/158), [EMU(Eastern Mediterranean University) 제공 문서](https://staff.emu.edu.tr/mobinabeheshti/Documents/courses/ITEC399/lecture%205%20-%20399.pdf)] 내용을 참고하여 작성한 자습 기록입니다.

<br>
<br>

# 📚ConstraintLayout 정리(7) - Dimension constrains

이번에는 ConstraintLayout의 Dimension과 관련된 속성에 대해 알아보고자 한다. 어떤 속성들인지 감이 오지 않아 사전적 의미에 대해 알아본 결과, dimension이란 '(공간의) 크기, (높이·너비·길이의) 치수'의 의미를 지닌다. 크기에 대해 정의하는 속성이라고 알고 가면 될 듯하다.

## 📔ConstraintLayout의 최소 크기와 최대 크기

아래와 같은 속성들을 사용해서 ConstraintLayout의 최소 크기와 최대 크기를 지정할 수 있다. 해당 속성들을 지정할 때 주의해야 할 점은, 이 속성들을 사용하기 위해선 적용할 ConstraintLayout의 너비와 높이 속성의 값이 모두 `WRAP_CONTENT`로 지정되어 있어야 한다.

|**속성명**|설명|
|:---:|:---|
|`minWidth`|레이아웃의 최소 너비 지정|
|`minHeight`|레이아웃의 최소 높이 지정|
|`maxWidth`|레이아웃의 최대 너비 지정|
|`maxHeight`|레이아웃의 최대 높이 지정|

## 📔 뷰 위젯들의 크기 및 제약
뷰 위젯의 크기는 `android:layout_width`와 `android:layout_height` 속성에 각각 아래와 같은 3가지 값 중 하나를 적용함으로써 지정할 수 있다.

1. dp와 Dimension 참조값(~/res/values/dimens.xml)과 같은 특정 크기 지정
2. WRAP_CONTENT 사용
3. 0dp(MATCH_PARENT) 사용

중요한 점으로 `MATCH_PARENT` 속성은 ConstraintLayout에서 권장되지 않는다. 이보다는 0dp로 지정한 뒤 각각의 제약을 지정하는 것을 권장한다.

뷰의 크기가 `MATCH_CONSTRAINT`로 설정되어 있을 경우, 즉 뷰가 ConstraintLayout의 사이드에 제약이 걸려있을 경우 가능한 모든 면적을 차지하게 된다. 이때 아래의 속성들을 사용하여 뷰의 크기를 지정할 수 있다.

|**속성명**|설명|
|:---:|:---|
|`layout_constraintWidth_min`|뷰의 최소 너비 지정|
|`layout_constraintHeight_min`|뷰의 최소 높이 지정|
|`layout_constraintWidth_max`|뷰의 최대 너비 지정|
|`layout_constraintHeight_max`|뷰의 최대 높이 지정|
|`layout_constraintWidth_percent`|뷰의 너비를 퍼센트로 지정(0 ~ 1)|
|`layout_constraintHeight_percent`|뷰의 높이를 퍼센트로 지정(0 ~ 1)|

<br>

퍼센트로 크기를 지정하기 위해서는 다음과 같은 조건을 준수해야 한다.
* 뷰의 크기가 `MATCH_CONSTRAINT(0dp)`로 설정되어 있어야 한다.
* `app:layout_constraintWidth_default="percent"` 또는 `app:layout_constraintHeight_default="percent"`를 사용하여 기본값을 퍼센트로 지정해둬야 한다.
* 해당 속성들은 0과 1 사이의 값을 지정하여 사용한다.

## 📔 Ratio(비율) 속성을 사용하여 크기 지정

뷰 위젯의 크기를 비율을 통해 지정하는 방법도 있다. 이를 위해서는 너비 또는 높이 값이 0dp(`MATCH_CONSTRAINT`)로 지정되어 있어야 한다. 비율을 사용하여 크기를 지정하려면 `app:layout_constraintDimensionRatio` 속성을 사용한다.

비율은 두 가지 방법을 통해 조정할 수 있다.
* 실수를 사용하여 너비와 높이 간의 비율을 조정하여 크기 지정
* "너비:높이" 와 같은 비율을 사용하여 크기 지정

너비와 높이의 값이 모두 0dp(`MATCH_CONSTRAINT`)인 상태에서도 비율을 지정할 수 있다. 이해를 위해 아래 코드를 보자. 너비와 높이가 모두 0dp이고, 상하좌우 제약이 걸려있는 버튼이다. 버튼에 `app:layout_constraintDimensionRatio="w, 1:3"` 속성값이 지정되어 있는 것을 볼 수 있다. 해당 문장의 뜻은 `layout_height`값을 제약에 맞추어 3만큼 먼저 설정한 뒤 `layout_width`의 길이를 1만큼 표시하겠다는 의미이다. 따라서 너비는 높이의 3분의 1만큼의 길이를 가진다.

```kotlin
<Button
    android:layout_width="0dp"
    android:layout_height="0dp"
    app:layout_constraintDimensionRatio="w, 1:3"
    app:layout_constraintTop_toTopOf="parent"
    app:layout_constraintBottom_toBottomOf="parent"
    app:layout_constraintStart_toStartOf="parent"
    app:layout_constraintEnd_toEndOf="parent" />
```

![](/assets/images/android/constraintlayout/ratio1.png)

이와 반대로 비율의 값을 `h, 1:3`으로 지정하게 되면, 이는 `layout_width`의 값을 제약에 맞춰 1로 먼저 지정한 뒤 `layout_height`값을 이와 비례하여 3만큼 지정하겠다는 의미이다. 해당 코드를 결과로 보면 뷰가 화면을 벗어난 것을 볼 수 있다.

```kotlin
<Button
    android:layout_width="0dp"
    android:layout_height="0dp"
    app:layout_constraintDimensionRatio="h, 1:3"
    app:layout_constraintTop_toTopOf="parent"
    app:layout_constraintBottom_toBottomOf="parent"
    app:layout_constraintStart_toStartOf="parent"
    app:layout_constraintEnd_toEndOf="parent" />
```

![](/assets/images/android/constraintlayout/ratio2.png)
