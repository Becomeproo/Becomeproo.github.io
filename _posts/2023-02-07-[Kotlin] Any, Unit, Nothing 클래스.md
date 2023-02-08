---
title:  "[Kotlin] Any, Unit, Nothing 클래스"

categories:
  - Kotlin
tags:
  - [Kotlin, Any, Unit, Nothing, 클래스, Kotlin 정리]

toc: true
toc_sticky: true
 
date: 2023-02-07 11:01:13+0900
last_modified_at: 2023-02-07 11:01:19+0900

---

<br>
<br>
<br>

# 📚 Any, Unit, Nothing 클래스

<br>

## 📔 Any

### 📖 공통 메서드 작성

모든 클래스는 기본으로 최상위 클래스 Any 클래스를 자동으로 상속한다. 따라서 Any 클래스에 확장 함수를 지정하면 공통 메서드로 사용된다.


```kotlin
fun Any.dir(): Set<String> {
    val a = this.javaClass.kotlin // 현재 객체의 클래스 저장
    println(a.simpleName)
    var ll = mutableListOf<String>()
    for (i in a.members) { // 클래스의 멤버 순회
        ll.add(i.name) // 이름을 리스트에 저장
    }
    return ll.toSet() // 오버로딩된 메서드 제거
}

println(Any().dir()) // Any 클래스 멤버만 출력
var count = 0
for (i in 'a'.dir()) { // 문자에 대한 멤버 출력
    print(i + ", ")
    count++
    if (count % 5 == 0) println()
}
```

    Any
    [equals, hashCode, toString]
    Char
    compareTo, dec, describeConstable, equals, inc, 
    minus, plus, rangeTo, rangeUntil, toByte, 
    toChar, toDouble, toFloat, toInt, toLong, 
    toShort, toString, hashCode, 

### 📖 Any 클래스의 메서드 오버라이딩 처리


```kotlin
import kotlin.reflect.full.isSubclassOf

class A {
    override fun toString() = "재정의 : A()" // 상위 클래스 메서드 재정의
    override fun hashCode(): Int {
        println("### hashCode 재정의 ###")
        return super.hashCode()
    }
}

val a: Any = A() // 최상위 클래스 자료형에 객체 전달

println(a.toString()) // 재정의한 메서드 출력
println(a.hashCode())
println(a.equals(a)) // 두 객체 비교

println((A::class).isSubclassOf(Any::class)) // 슈퍼 클래스와 서브 클래스 관계 확인
println((A::class).supertypes) // 상속 표시를 하지 않아도 기본 최상위 클래스 상속
```

    재정의 : A()
    ### hashCode 재정의 ###
    357788434
    true
    true
    [kotlin.Any]


<br>

## 📔 Unit

함수를 반환하는 과정에서 반환값이 존재하지 않을 경우, 보통 Unit을 반환 자료형으로 지정한다.


```kotlin
import kotlin.reflect.full.isSubclassOf


println((Unit::class).isSubclassOf(Any::class)) // 서브 클래스 여부 확인
println(Unit.dir()) // 공통 메서드 dir 실행
println((Unit::class).supertypes) // 상위 클래스 확인

fun func(a: Any): Unit {
    println(a.toString() + " => " + a.javaClass.kotlin)
}

println("#### 각 자료형의 클래스 확인 ####")
func(100)
func(100L)
func(100.0)
func(100.0f)
func('c')
func(false)
func("문자열")
func(Exception("예외"))
```

    true
    Unit
    [toString, equals, hashCode]
    [kotlin.Any]
    #### 각 자료형의 클래스 확인 ####
    100 => class kotlin.Int
    100 => class kotlin.Long
    100.0 => class kotlin.Double
    100.0 => class kotlin.Float
    c => class kotlin.Char
    false => class kotlin.Boolean
    문자열 => class kotlin.String
    java.lang.Exception: 예외 => class java.lang.Exception
    

<br>

## 📔 Nothing

Nothing 클래스는 함수를 반환할 때 아무것도 없다는 것을 표시하는 클래스이며, 보통 예외 등을 처리할 때 해당 클래스의 객체가 발생한 것으로 여긴다.


```kotlin
println((Nothing::class).isSubclassOf(Any::class)) // 상속 관계
println((Nothing::class).isSubclassOf(Int::class))

println((Nothing::class).supertypes) // 상위 클래스 확인

fun except(): Nothing { // 반환값이 없음
    throw Exception(" 예외 ") // 예외 발생
}

try { // 예외 처리
    except()
} catch (e: Exception) {
    println(e.message)
}
```

    true
    false
    [kotlin.Any]
     예외 

