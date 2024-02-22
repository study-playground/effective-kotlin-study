### 일반적인 알고리즘을 구현할 때 제네릭을 사용하라

* 아큐먼트로 함수에 값을 전달할 수 있는 것처럼, 타입 아규먼트를 사용하면 함수에 타입을 전달할 수 있다.
* 타입 아규먼트를 사용하는 함수를 제네릭 함수라고 부른다.
* 타입 파라미터는 컴파일러에 타입과 관련된 정보를 제공하여 컴파일러가 타입을 조금이라도 더 정확하게 추측할 수 있게 해준다.

### 제네릭 제한

* 타입 파라미터의 중요한 기능 중 하나는 구체적인 타입의 서브타입만 사용하게 타입을 제한하는 것이다.

```kotlin
fun <T : Comparable<T>> Iterable<T>.sorted(): List<T> {
    /*..*/
}

fun <T, C : MutableCollection<in T>> Iterable<T>.toCollection(destination: C): C {
    /*..*/
}

fun <T : String> subTypeLimit(list: List<T>) {
    println(list)
}

fun main() {
    val list1 = listOf<String>("TEST1", "TEST2")
    val list2 = listOf<Int>(1,2)
    subTypeLimit(list1)
    subTypeLimit(list2) // 컴파일 에러 !!
}
```

* 타입에 제한이 걸리므로, 내부에서 해당 타입이 제공하는 메서드를 사용할 수 있다.
* 예를들어, Any 로 제한하면 nullable 이 아님을 나타낸다.