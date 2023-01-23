---
title:  "[Android] ConstraintLayout 정리(10) - Optimizer"

categories:
  - Android
tags:
  - [Android, Android 정리, ConstraintLayout]

toc: true
toc_sticky: true
 
date: 2023-01-23 15:55:40+0900
last_modified_at: 2023-01-23 15:55:40+0900

---

<br>
<br>
<br>

> 아래 내용은 [Android Developer 공식 문서](https://developer.android.com/reference/androidx/constraintlayout/core/widgets/Optimizer#OPTIMIZATION_BARRIER()) 내용을 참고하여 작성한 자습 기록입니다.

<br>
<br>

# 📚ConstraintLayout 정리(10) - Optimizer

ConstraintLayout 1.1에서는 레이아웃의 속도를 높이기 위해 몇 가지 최적화 방법을 제공한다. 최적화는 별도의 단계로 실행된다. 일반적으로 레이아웃에서 속성값을 찾고 이를 적용시킨다.

이를 위해 `app:layout_optimizationLevel` 속성을 사용하는데, 다음과 같이 최적화 레벨을 지정하여 사용할 수 있다.

|**속성값**|설명|
|:---:|:---|
|`none`|사용 안 함|
|`standard`|기본값|
|`direct`|고정된 요소에 연결된 제약 조건과 관련한 최적화|
|`barrier`|barrier와 관련한 최적화|
|`chain`|체인 제약 조건 최적화|
|`dimensions`|크기 측정 최적화|

위와 같은 설정 외에도 다양한 최적화 속성값을 [Android Deveoper 공식 문서](https://developer.android.com/reference/androidx/constraintlayout/core/widgets/Optimizer#OPTIMIZATION_BARRIER())에서 찾아볼 수 있다.