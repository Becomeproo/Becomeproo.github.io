---
title:  "[Kotlin] Nullable 처리"

categories:
  - Kotlin
tags:
  - [Kotlin, Nullable, Kotlin 정리]

toc: true
toc_sticky: true
 
date: 2023-02-10 11:43:49+0900
last_modified_at: 2023-02-10 11:43:53+0900

---

<br>
<br>
<br>

# 📚 Nullable 처리

<br>

## 📔 널러블 여부

보통 변수에 객체가 할당되지 않거나 null이 할당된 경우에 메서드나 연산자 등을 사용하면 널 포인터 예외가 발생한다. 이를 방지하기 위해 항상 널을 체크한 후에 연산자나 메서드 처리를 수행한다.

<br>

### 📖 널러블 자료형 처리 규칙

* 기존 자료형에 물음표를 붙이면 널을 처리할 수 있는 자료형이 만들어짐
* 널러블 자료형은 항상 널이 불가능한 자료형보다 상위 자료형이다. 그래서 널이 불가능한 자료형의 변수를 널러블 자료형의 변수에 할당할 수 있다. 하지만 반대로 널러블 자료형은 널이 불가능한 자료형의 변수에 할당할 수 없다
* 널에 대한 체크는 컴파일 타임에 확정하지만, 실제 실행할 때는 처리하지 않는다. 그래서 널에 대한 처리는 전부 컴파일 처리할 때 예외를 발생시킨다

<br>

### 📖 널러블 처리 연산자

* 안전 연산자(?.): 널 값이 들어오면 그다음 메서드를 처리하지 않는다. 반면 실제 객체가 들어올 경우에는 안전하게 메서드를 실행한다
* 엘비스 연산자(?:): 널 값이면 다음에 들어오는 값으로 변환한다. 안전 연산자와 엘비스 연산자를 같이 사용하면 조건문을 사용하는 것보다 편리하다
* 널 단언 연산자(!!): 널 값이 들어올 수 없다고 확신할 경우에만 사용한다. 널 값이 들어오면 예외를 발생시킨다


```kotlin
val ss = "100"
println(ss!!.length)

val ss1: String? = null
try {
    ss1!!.length // 널 값이 아니어야 함. 널 값이 들어오면 예외 처리
} catch (e: Exception) {
    println("예외: ${e.message}")
}

val d: String? = "문자열"
println(d?.length) // 널인 경우 널 처리. 그렇지 않다면 뒤의 메서드 실행

val e: String? = null
println(e?.length)

val f: String? = null
println(e?.length ?: 0) // 널 값이 라면 뒤의 0을 디폴트 값으로 처리

var g = if (f != null) f?.length else 0
println(g)
```

    3
    예외: null
    3
    null
    0
    0


<br>

### 📖 널러블 자료형 지정

변수나 함수 반환을 처리할 때 널이 가능한 자료형인 널러블 자료형으로 변환해서 널이 들어오는 것을 명기할 수 있다.


```kotlin
import kotlin.reflect.full.isSubclassOf

val a1: Any? = null // nullable 타입 지정
val a2: Any = Any()
val a3: Any? = Any()

var s1: String? = null
var s2: String = "문자열"

//s2 = s1  <- 상위 타입에 하위 타입 처리 불가
s1 = s2 // 상위 타입에 하위 타입 처리 가능
println(s1)
s2 = s1 as String // 명확한 타입으로 저장
println(s2)

var s3: String = "문자열"
var s4: String? = "문자열2"

println((s3::class))
println((s4!!::class))
println((s3::class).isSubclassOf(s4!!::class))
println((s4!!::class).isSubclassOf(s3::class))
```

    문자열
    문자열
    class kotlin.String
    class kotlin.String
    true
    true


<br>

### 📖 널러블 자료형 점검한 후에 처리

변수에 널러블 자료형이 할당되면 널 값이 들어올 수 있다는 것을 알 수 있다. 들어온 널 값을 예외 없이 처리하려면 널 값을 체크한 다음에 널 값이 아닌 경우만 메서드나 속성 등을 처리한다.


```kotlin
val b: String? = "Kotlin" // 널러블 문자열에 문자열 할당
if (b != null) println("문자열: $b")
else println("널값")

val c: String? = null // 널러블 문자열에 널 할당
if (c == null) println("널값") 
else println("문자열: $c")
```

    문자열: Kotlin
    널값


<br>

### 📖 널러블 연산자로 처리

안전 연산자(?.)를 사용하면 널 값일 경우 null로 처리하고 널이 아니면 그다음에 오는 속성이나 메서드를 실행한다.


```kotlin
fun <T: Any> T?.mapCheck(f: (T) -> T): T? = // 널러블 처리
    when(this) {
        null -> null // 널 값
        else -> f(this) // 원래 타입을 처리 
    }
    
val n1: Any? = null // 널러블 타입을 지정하면 널값 할당 가능
val n2 = 100

println(n1?.mapCheck({ it }))
println(n2.mapCheck({ it * 3 }))
```

    null
    300


<br>

### 📖 내장 클래스의 널을 처리하는 메서드

문자열이나 리스트 등은 널러블로 사용할 경우 내부적으로 널 값을 처리하는 메서드가 있다.


```kotlin
val ff: String? = null
println(ff?.getOrNull(0))

val nullableList: List<Int?> = listOf(1, 2, null, 4) // 리스트 내 원소
println(nullableList.filterNotNull()) // 리스트 내 널 원소 제거

val s1: String? = null
println(s1.isNullOrEmpty())
println(s1.isNullOrBlank())

val s2 = " "
println(s2.isNullOrEmpty())
println(s2.isNullOrBlank())

val s3: String = "\t\n"
println(s3.isNullOrEmpty())
println(s3.isNullOrBlank())
```

    null
    [1, 2, 4]
    true
    true
    false
    true
    false
    true

