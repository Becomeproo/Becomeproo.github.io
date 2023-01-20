---
title:  "[Android] ConstraintLayout 정리(8) - Chains"

categories:
  - Android
tags:
  - [Android, Android 정리, ConstraintLayout]

toc: true
toc_sticky: true
 
date: 2023-01-20 17:35:50+0900
last_modified_at: 2023-01-20 17:35:50+0900

---

<br>
<br>
<br>

> 아래 내용은 [Android Developer 공식 문서](https://developer.android.com/training/constraint-layout?hl=ko)와 [EMU(Eastern Mediterranean University) 제공 문서](https://staff.emu.edu.tr/mobinabeheshti/Documents/courses/ITEC399/lecture%205%20-%20399.pdf)] 내용을 참고하여 작성한 자습 기록입니다.

<br>
<br>

# 📚ConstraintLayout 정리(8) - Chains

체인은 단일 축(수평 또는 수직)에서 그룹과 같이 동작할 수 있도록 도움을 준다. 다른 축은 독립적으로 제약을 지정할 수 있다.

<br>
<br>

## 📔 체인 관계 형성

![](/assets/images/android/constraintlayout/chain1.png)

두 위젯 간의 체인 관계는 두 뷰의 상호 제약을 통해 연결된 경우 형성된다. 코드로 보면 다음과 같다.

먼저 상호 연결되지 않은 두 개의 버튼이 있다. `A`는 시작 사이드 쪽으로 제약이 지정되어있고, `B`는 끝 사이드 쪽으로 제약이 지정되어 있다. 따라서 두 버튼은 양쪽 끝에 배치되어 있는 것을 볼 수 있다.

```kotlin
    <Button
        android:id="@+id/A"
        android:layout_width="150dp"
        android:layout_height="100dp"
        android:text="A"
        android:textSize="30sp"
        app:layout_constraintTop_toTopOf="parent"
        app:layout_constraintStart_toStartOf="parent" />

    <Button
        android:id="@+id/B"
        android:layout_width="150dp"
        android:layout_height="100dp"
        android:text="B"
        android:textSize="30sp"
        app:layout_constraintTop_toTopOf="parent"
        app:layout_constraintEnd_toEndOf="parent" />
```

![](/assets/images/android/constraintlayout/chain2.png)

다음으로 두 버튼이 서로를 향해 제약을 걸도록 지정해 주었다. 이미지를 보면, 두 버튼이 서로를 향해 제약을 건 모양이 마치 체인을 걸어 둔 모양과 비슷하다. 이로써 체인 관계가 형성이 되었다. 이에 따라 두 버튼 사이에 간격을 둔 하나의 그룹으로 묶여 중앙에 정렬된 것을 확인할 수 있다.
```kotlin
    <Button
        android:id="@+id/A"
        android:layout_width="150dp"
        android:layout_height="100dp"
        android:text="A"
        android:textSize="30sp"
        app:layout_constraintEnd_toStartOf="@id/B"
        app:layout_constraintTop_toTopOf="parent"
        app:layout_constraintStart_toStartOf="parent" />

    <Button
        android:id="@+id/B"
        android:layout_width="150dp"
        android:layout_height="100dp"
        android:text="B"
        android:textSize="30sp"
        app:layout_constraintStart_toEndOf="@id/A"
        app:layout_constraintTop_toTopOf="parent"
        app:layout_constraintEnd_toEndOf="parent" />
```

![](/assets/images/android/constraintlayout/chain3.png)

<br>
<br>

## 📔 체인 제어

체인 관계 형성은 두 개의 뷰 사이에서만 이루어지는 게 아니라 다수의 뷰들 사이에서도 이루어질 수 있다.

체인은 체인 관계가 형성된 뷰들 중에서 첫 번째 요소의 속성에 의해 제어된다.(체인의 '머리' 부분)
![](/assets/images/android/constraintlayout/chain4.png)

수평 형태의 체인 관계에서 제어 역할을 하는 위젯은 가장 왼쪽에 위치한 위젯이다. 그리고 수직 형태의 체인 관계에서는 최상단에 위치한 위젯이 제어 역할을 한다. 

<br>
<br>

## 📔 체인 스타일
머리 부분의 뷰에 `layout_constraintHorizontal_chainStyle`또는 `layout_constraintVertical_chainStyle`의 속성을 지정하면, 체인 관계에 속하는 뷰들의 정렬 방식이 지정된 값에 따라 변경된다.

Chain style의 속성값들은 다음과 같다.

|**속성명**|설명|
|:---:|:---|
|`spread`|뷰들이 분산됨(기본값)|
|`weighted chain`|뷰들이 `spread` 속성 값을 가진 상태에서, 특정 뷰가 `Match_Constraint(0dp)`의 값을 가진 경우, 이 뷰는 나머지의 공간을 차지하게 됨|
|`spread_inside`|뷰들이 분산되지만 양쪽의 뷰들은 각각 끝에 배치|
|`packed`|뷰들이 한 곳에 밀집됨, `constraintHorizontal_bias` 또는 `constraintVertical_bias`와 같은 값이 해당 속성 값이 적용된 뷰의 배치에 영향을 줌|

![](/assets/images/android/constraintlayout/chain_style.png)

<br>
<br>

## 📔 Weighted chains
체인의 기본값은 사용 가능한 공간에 뷰들을 균등하게 분산시킨다. 만약 하나 또는 더 많은 뷰가 `Match_Constraint`를 사용하고 있을 때, 해당 값을 적용한 뷰들은 사용 가능한 빈 공간을 사용한다. `layout_constraintHorizontal_weight`와 `layout_constraintVertical_weight` 속성은 `Match_Constraint` 값을 적용한 뷰들간 배치될 공간을 지정한다.

코드와 함께 이해해 보자. 체인 관계에 있는 버튼 세 개가 있다. `A`는 너비와 높이가 모두 `wrap_content`로 지정되어 있고, 나머지 `B`와 `C`는 모두 `Match_Constraint(0dp)`로 지정되어 있다. 따라서 `A`만 위젯의 내용에 맞게 공간을 차지하고, `B`와 `C`는 `A`가 차지하고 남은 공간을 공유한다.

```kotlin

    <Button
        android:id="@+id/A"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="A"
        android:textSize="30sp"
        app:layout_constraintEnd_toStartOf="@id/B"
        app:layout_constraintTop_toTopOf="parent"
        app:layout_constraintStart_toStartOf="parent" />

    <Button
        android:id="@+id/B"
        android:layout_width="0dp"
        android:layout_height="0dp"
        android:text="B"
        android:textSize="30sp"
        app:layout_constraintStart_toEndOf="@id/A"
        app:layout_constraintTop_toTopOf="parent"
        app:layout_constraintBottom_toBottomOf="@id/A"
        app:layout_constraintEnd_toStartOf="@id/C" />

    <Button
        android:id="@+id/C"
        android:layout_width="0dp"
        android:layout_height="0dp"
        android:text="C"
        android:textSize="30sp"
        app:layout_constraintStart_toEndOf="@id/B"
        app:layout_constraintTop_toTopOf="parent"
        app:layout_constraintBottom_toBottomOf="@id/A"
        app:layout_constraintEnd_toEndOf="parent" />

```

![](/assets/images/android/constraintlayout/weight_chain1.png)

여기서 `B`의 공간이 `C`의 공간보다 2배 더 많이 차지하도록 하고 싶을 경우, `B`와 `C`에 각각 `layout_constraintHorizontal_weight` 속성을 만들어 `B`에는 2를, `C`에는 1을 적용해 준다. 이렇게 되면 `B`가 `C`보다 2배의 공간을 더 차지한 것을 볼 수 있다.

```kotlin
    <Button
        android:id="@+id/B"
        android:layout_width="0dp"
        android:layout_height="0dp"
        android:text="B"
        android:textSize="30sp"
        android:backgroundTint="@color/teal_700"
        app:layout_constraintHorizontal_weight="2"
        app:layout_constraintStart_toEndOf="@id/A"
        app:layout_constraintTop_toTopOf="parent"
        app:layout_constraintBottom_toBottomOf="@id/A"
        app:layout_constraintEnd_toStartOf="@id/C" />

    <Button
        android:id="@+id/C"
        android:layout_width="0dp"
        android:layout_height="0dp"
        android:text="C"
        android:textSize="30sp"
        app:layout_constraintHorizontal_weight="1"
        app:layout_constraintStart_toEndOf="@id/B"
        app:layout_constraintTop_toTopOf="parent"
        app:layout_constraintBottom_toBottomOf="@id/A"
        app:layout_constraintEnd_toEndOf="parent" />

```

![](/assets/images/android/constraintlayout/weight_chain2.png)

<br>
<br>

## 📔 체인에 여백 값 지정하기

체인 관계의 뷰에 여백을 지정하는 경우, 여백의 값은 합쳐진다. 예를 들어, 수평 체인 관계의 뷰가 두 개 존재한다고 했을 때, 한 뷰가 끝 쪽으로 10dp의 여백을 정의하고, 다른 하나의 뷰는 시작 쪽의 여백으로 5dp를 정의하는 경우, 두 뷰 사이의 여백은 15dp이다.

![](/assets/images/android/constraintlayout/chain_margin.png)
