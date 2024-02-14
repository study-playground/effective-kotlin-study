### 변수 타입이 명확하지 않은 경우 확실하게 지정하라
* 코틀린은 개발자가 타입을 지정하지 않아도, 타입을 지정해서 넣어주는 타입 추론 시스템이 있다.

```kotlin
fun main() {
    val num = 10
    val name = "Marcin"
    val ids = listOf(12, 112, 554)
}
```

* 가독성을 위해 코드를 설계할 때는 읽는 사람에게 중요한 정보를 숨겨서는 안된다.
* 가독성 향상을 위해 타입이 명확하지 않으면 표기하는게 좋다.

```kotlin
val data: UserData = getSomeData()
```