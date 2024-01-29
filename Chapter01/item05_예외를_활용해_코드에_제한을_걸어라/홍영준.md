### 요약
* 제한을 훨씬 더 쉽게 확인할 수 있다.
* 애플리케이션을 더 안정적으로 지킬 수 있다.
* 코드를 잘못 쓰는 상황을 막을 수 있다.
* 스마트 캐스팅을 활용할 수 있다.

### 예외를 활용해 코드에 제한을 걸어라
* require 블록 : Argument 를 제한
* check 블록 : state 와 관련된 동작을 제한
* assert 블록 : 테스트 시 TRUE 인지 확인할 수 있다. (테스트 모드에서만 동작)
  * assert 종류 : assertEquals, assertNotEquals,  assertTrue, assertFalse, assertThrows, assertNotNull, assertNull
* return 또는 throw 와 함께 활용하는 Elvis 연산자 (?:)

### 장점
* 제한을 걸면 문서를 읽지않은 개발자도 문제를 확인할 수 있다.
* 문제가 있을 경우에 함수가 예상하지 못한 동작을 하지 않고 예외를 던진다.
* 코드가 어느정도 자체적으로 검사가된다.
* 스마트 캐스트 기능을 활용할 수 있게 되므로, 캐스트를 적게 할 수 있다.

### Argument
* 함수를 정의할 때 타입 시스템을 활용해서 Argument 에 제한을 거는 코드로 사용
* require 함수를 사용 -> require 함수는 제한을 확인하고, 제한을 만족하지 못할 경우 예외를 던진다.
* IllegalArgumentException 발생

```kotlin
fun main() {
    factorial(-2)
}

fun factorial(num: Int): Long {
    require(num >= 0) {
        "Cannot calculate factorial of $num because it is smaller than 0"
    }
    return if (num <= 1) 1 else factorial(num -1) * num
}
```

### State
* 어떤 구체적인 조건을 만족할 때만 함수를 사용할 수 있게 해야 할 때 사용
* IllegalStateException 발생
* check 함수 사용
* 일반적으로 require 블록 뒤에 배치

```kotlin
fun isNull(text: String?) {
    checkNotNull(text)
}
```

### Assert
* Assert 계열의 함수는 코드가 예상대로 동작하는지 확인하므로 테스트라고 할 수 있다.
* 현재는 코틀린/JVM 에서만 활성화되며, -ea JVM 옵션을 활성화해야 확인할 수 있다.
* 다만, 프로덕션 환경에서는 오류가 발생하지 않는다.
* 만약 코드가 정말 심각한 오류고, 심각한 결과를 초래할 수 있으면 check 를 사용하는 것이 좋다.
* Assert 계열의 함수는 코드를 자체 점검하며, 더 효율적으로 테스트할 수 있게 해준다.
* 특정 상황이 아닌 모든 상황에 대한 테스트를 할 수 있다.
* 실행 시점을 확인할 수 있다.
* 실제 코드가 더 빠른 시점에 실패하게 만든다. 따라서 예상치 못한 동작이 언제 실행되었는지 빠르게 찾을 수 있다.

### nullability 와 스마트 캐스팅
* require 와 check 블록으로 어떤 조건을 확인해서 true 가 나왔다면, 해당 조건은 그 이후로도 true 일 거라고 가정한다. -> 이를 스마트 캐스팅이라 한다.

```kotlin
data class Webtoon(
    val subject: Any,
)

fun example4(webtoon: Webtoon) {
     require(webtoon.subject is String)
    // 스마트 캐스팅 -> require 함수가 없다면 type mismatch 발생
    val subject: String = webtoon.subject
}
```

```kotlin
data class Webtoon(
    val id: Int?,
    
)

fun example5(webtoon: Webtoon) {
    require(webtoon.id != null)
    // null 아님
    val id: Int = webtoon.id
}
```

* nullablility 를 목적으로, 오른쪽에 throw 또는 return 을 두고 Elvis 연산자를 활용할 수 있다.
* 이 코드는 읽기 쉽고, 유연하게 사용할 수 있다.
* null 일 때 여러 차리가 필요하면 run 을 활용할 수도 있다.

```kotlin

data class Webtoon(
    val id: Int?,
)

fun example6(webtoon: Webtoon) {
    val id = webtoon.id ?: throw IllegalArgumentException("is null")
}

fun example7(webtoon: Webtoon) {
    val id = webtoon.id ?: run {
        println("Id is null")
        return
    }
}
```