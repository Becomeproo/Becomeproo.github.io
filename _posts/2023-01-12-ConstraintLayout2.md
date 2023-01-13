---
title:  "[Android] ConstraintLayout 정리(2) - Relative Positioning
"

categories:
  - Android
tags:
  - [Android, 안드로이드 정리, ConstraintLayout]

toc: true
toc_sticky: true
 
date: 2023-01-12
last_modified_at: 2023-01-12
---

<br>
<br>
<br>

> 아래 내용은 [Android Developer 공식문서](https://developer.android.com/training/constraint-layout?hl=ko)와 [recipes4dev님의 블로그](https://recipes4dev.tistory.com/158), [EMU(Eastern Mediterranean University) 제공 문서](https://staff.emu.edu.tr/mobinabeheshti/Documents/courses/ITEC399/lecture%205%20-%20399.pdf)] 내용을 참고하여 작성한 자습 기록입니다.

<br>
<br>

# 📚ConstraintLayout 정리(2) - Relative Positioning(상대 위치 지정)

<br>

## 📔Relative Positioning(상대 위치 지정)이란

Relative Positioning(상대 위치 지정)은 ConstraintLayout에서 레이아웃을 만드는 기본 구성 요소 중 하나로, 뷰 위젯의 상대적인 위치에 따라 뷰의 표시 영역을 지정하는 방법이다. 

Relative Positioning 방법은
1. 레이아웃이 중첩되는 성능 이슈를 줄여줌
2. 화면 구성의 복잡성을 낮춤
3. UI 화면 구성에 소요되는 시간을 단축

과 같은 장점을 갖고 있다.

<br>

## 📔Relative Positioning 사용
Relative Positioning 방법은 ConstraintLayout의 `자식 뷰 위젯 간 상대 위치` 또는 `자식 뷰와 레이아웃 간 상대 위치`에 대해 수평이나 수직으로 제약 조건을 지정하여 뷰의 위치를 결정한다.

수평과 수직의 지정은 다음과 같은 속성을 사용한다.
* 수평: left, right, start, end
* 수직: top, bottom, text baseline

기본적인 지정 개념은 뷰 위젯의 한쪽 면을 다른 뷰 위젯의 한쪽 면에 지정한다. 따라서 <br>
**`layout_constraint[지정된 위젯의 면]_to[대상 위젯의 면]Of`**
<br>
와 같은 이름 규칙을 가진다. 이는 <u>"지정된 위젯의 면"을 "대상 위젯의 면"에 맞춘다</u> 를 의미한다.

쉬운 이해를 위해 그림으로 설명해 보자.

다음과 같이 "A"와 "B"라는 뷰 위젯이 존재한다고 가정해 보자. "B" 위젯을 "A" 위젯의 오른쪽으로 배치시키고자 할 때는 다음과 같은 제약을 지정한다.


![relative-positioning1](/assets/images/android/constraintlayout/relative_positioning1.png)

```kotlin
android:id="@+id/B"
layout_constraintLeft_toRightOf="@id/A" // A는 생성되어 있다고 가정
```

위와 같은 제약을 지정한 후의 결과는 원하던 대로 "B"가 "A"의 오른쪽으로 배치된 것을 볼 수 있다. 여기서 주의할 점은 **ConstraintLayout은 수평과 수직 모두 각각 하나 이상 지정해 줘야 한다는 점이다.** 때문에 `layout_constraintLeft_toRightOf="@id/A"`외에도 `layout_constraintTop_toTopOf="@id/A"`를 지정해 주었다. "B"의 상단을 "A"의 상단과 맞추겠다는 의미이다.

![relative-positioning1](/assets/images/android/constraintlayout/relative_positioning2.png)

<br>

## Relative Positioning(상대 위치 지정)을 위한 속성

Relative Positioning을 위한 속성은 다음과 같다.

![relative-positioning](/assets/images/android/constraintlayout/relative_positioning3.png)

|**속성명**|설명|
|:---:|:---|
|`layout_constraintLeft_toLeftOf`|뷰의 왼쪽 사이드(Side)를 대상 뷰의 왼쪽 사이드(Side)에 맞춤|
|`layout_constraintLeft_toRightOf`|뷰의 왼쪽 사이드(Side)를 대상 뷰의 오른쪽 사이드(Side)에 맞춤|
|`layout_constraintRight_toLeftOf`|뷰의 오른쪽 사이드(Side)를 대상 뷰의 왼쪽 사이드(Side)에 맞춤|
|`layout_constraintRight_toRightOf`|뷰의 오른쪽 사이드(Side)를 대상 뷰의 오른쪽 사이드(Side)에 맞춤|
|`layout_constraintTop_toTopOf`|뷰의 상단 사이드(Side)를 대상 뷰의 상단 사이드(Side)에 맞춤|
|`layout_constraintTop_toBottomOf`|뷰의 상단 사이드(Side)를 대상 뷰의 하단 사이드(Side)에 맞춤|
|`layout_constraintBottom_toTopOf`|뷰의 하단 사이드(Side)를 대상 뷰의 상단 사이드(Side)에 맞춤|
|`layout_constraintBottom_toBottomOf`|뷰의 하단 사이드(Side)를 대상 뷰의 하단 사이드(Side)에 맞춤|
|`layout_constraintBaseline_toBaselineOf`|뷰의 텍스트 baseline을 대상 뷰의 텍스트 baseline에 맞춤|
|`layout_constraintStart_toEndOf`|뷰의 시작 사이드(Side)를 대상 뷰의 끝 사이드(Side)에 맞춤|
|`layout_constraintStart_toStartOf`|뷰의 시작 사이드(Side)를 대상 뷰의 시작 사이드(Side)에 맞춤|
|`layout_constraintEnd_toStartOf`|뷰의 끝 사이드(Side)를 대상 뷰의 시작 사이드(Side)에 맞춤|
|`layout_constraintEnd_toEndOf`|뷰의 끝 사이드(Side)를 대상 뷰의 끝 사이드(Side)에 맞춤|

모든 속성값에는 대상 뷰의 ID, 또는 `parent`를 사용한다.