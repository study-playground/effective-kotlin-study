### 요약
* 연산자 오버로딩은 그 이름이 의미에 맞게 사용하는 것이 좋다.
* 연산자 의미가 명확하지 않다면, 연산자 오버로딩은 사용하지 않는 것이 좋다.
* 이름이 있는 일반 함수를 사용하는 것을 권장하며, 꼭 연산자 같은 형태로 쓰고 싶다면 infix 확장 함수를 활용하라

### 연산자 오버로드를 할 때는 의미에 맞게 사용하라

* 팩토리얼은 수학에서 ! 기호로 표기한다.
* 코틀린에서 연산자 오버로딩 기능을 사용하여 다음과 같이 ! 에 대항하는 ```.not``` 를 오버로드할 수 있다.
* 오해의 소지가 있으므로 함수 이름이 ```not``` 이므로 논리 연산에 사용해야지, 팩토리얼에 사용하면 안된다.

```kotlin
fun Int.factorial(): Int = (1..this).product()

fun Iterable<Int>.product(): Int = fold(1) { acc, i -> acc * i }

operator fun Int.not() = factorial()

fun main() {
    // 720 출력
    print(!6)
}
```

### 연산자 오버로드를 실제로 하는 예시
```kotlin
data class Point(
    val x: Int,
    val y: Int,
) {
    operator fun plus(right: Point): Point = Point(x + right.x, y + right.y)
}

fun main() {
    val point1 = Point(10, 30)
    val point2 = Point(30, 50)

    // Point(x=40, y=80) 출력
    println(point1 + point2)
}
```

### 분명하지 않은 경우
* 하지만 관례를 충족하지지 아닌지 확실하지 않을 때가 문제이다.
* 예를 들어, * 연산자의 의미를 세 번 반복하는 새로운 함수를 만들어낸다고 생각할 수 있다.

```kotlin
operator fun Int.times(operation: () -> Unit): () -> Unit = { repeat(this) { operation() } }

fun main() {
    val hello = 3 * { print("Hello") }
    hello()
}
```

* 사실 의미가 명확하지 않을 때는 ```infix``` 키워드를 사용하는 것이 좋다.

```kotlin
operator fun Int.timesRepeated(operation: () -> Unit): () -> Unit = { repeat(this) { operation() } }

fun main() {
    val hello = 3 timesRepeated { print("Hello") }
    hello()
}
```