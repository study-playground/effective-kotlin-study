### 멤버 확장 함수의 사용을 피하라

* 어떤 클래스에 대한 확장 함수를 정의할 때, 이를 멤버로 추가하는 것은 좋지 않다.
* 확장 함수는 첫 번째 아규먼트로 리시버를 받는 단순한 일반 함수로 컴파일된다.

```kotlin
fun String.isPhoneNumber(): Boolean {
    return length == 7 && all { it.isDigit() }
}

// 컴파일
fun isPhoneNumber('$this': String): Boolean {
    return '$this'.length == 7 && '$this'.all { it.isDigit() }
}
```

* 이렇게 단순하게 변경되므로 확장 함수를 클래스 멤버로 정의할 수 있고, 인터페이스 내부에 정의할 수도 있다.

```kotlin
interface PhoneBook {
	fun String.isPhoneNumber(): Boolean
}

class Fizz: PhoneBook {
	override fun String.isPhoneNumber(): Boolean = length == 7 && all { it.isDigit() }
}
```

* 확장 함수를 멤버로 정의하는 것은 굉장히 좋지 않다.

#### 가시성을 제한할 수 있다고 생각할 수 있지만 확장만으로 가시성을 제한하지 못한다. -> 이해가 잘안됨

* 이는 단순하게 확장 함수를 사용하는 형태를 어렵게 만들 뿐이다.

```kotlin
class PhoneBookCorrect {
    private fun String.isPhoneNumber() = length == 7 && all { it.isDigit() }
}

fun main() {
    PhoneBookCorrect().apply { "1234567890".isPhoneNumber() } // 컴파일 에러 가시성 제한됨
}

private fun String.isPhoneNumber() = length == 7 && all { it.isDigit() } // 가시성 제한 
```

#### 레퍼런스를 지원하지 않는다.

```kotlin
// bad habbit
class PhoneBookIncorrect {
    fun String.isPhoneNumber() = length == 7 && all { it.isDigit() }
}

fun String.isPhoneNumber() = length == 7 && all { it.isDigit() }

val ref = String::isPhoneNumber
val str = "1234567890"
val strRef = str::isPhoneNumber

val refObject = PhoneBookIncorrect::isPhoneNumber // 오류
val book = PhoneBookIncorrect()
val bookRef = book::isPhoneNumber // 오류
```

#### 암묵적 접근을 할 때, 두 리시버 중에 어떤 리시버가 선택될지 혼동된다.

```kotlin
class A {
    val a = 10
}

class B {
    val a = 20
    val b = 30

    fun A.test() = a + b // 40? 50?
}

fun main() {
    B().apply { println(A().test()) } // 정답 : 40
}
```

#### 확장 함수가 외부에 있는 다른 클래스를 리시버로 받을 때, 해당 함수가 어떤 동작을 하는지 명확하지 않다.

```kotlin
class A {
    var state = 10
}

class B {
    var state = 20
    
    fun A.update() = state + 10  // A 와 B 중 에서 어떤 것을 업데이트 할까요?
}

fun main() { 
    B().apply { println(A().update()) } // 정답 : 20 (A를 업데이트 한다)
}
```

* 멤버 확장 함수를 사용하는 것이 의미가 있는 경우엔 사용해도 좋지만, 일반적으로는 그 단점을 인지하고 사용하지 않는 것이 좋다.