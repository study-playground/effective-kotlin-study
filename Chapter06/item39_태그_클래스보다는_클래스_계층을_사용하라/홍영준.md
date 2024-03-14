### 태그 클래스보다는 클래스 계층을 사용하라

* 상수 모드를 가진 클래스에서 상수 모드를 태그라고 부르며, 태그를 가진 클래스를 태그 클래스라고 부른다.
* 즉, 태그는 해당 클래스가 어떤 타입인지에 대한 정보를 담고 있는 멤버 변수를 의미한다.

```kotlin
class ValueMatcher<T> private constructor(
    private val value: T? = null,
    private val matcher: Matcher
) {
    fun match(value: T?) = when(matcher) {
        Matcher.EQUAL -> value == this.value
        Matcher.NOT_EQUAL -> value != this.value
        Matcher.LIST_EMPTY -> value is List<*> && value.isEmpty()
        Matcher.LIST_NOT_EMPTY -> value is List<*> && value.isNotEmpty()
    }

    enum class Matcher {
        EQUAL,
        NOT_EQUAL,
        LIST_EMPTY,
        LIST_NOT_EMPTY
    }

    // 정적 팩터리 메서드로 객체 생성
    companion object {
        fun <T> equal(value: T) = ValueMatcher<T>(value = value, matcher = Matcher.EQUAL)
        fun <T> notEqual(value: T) = ValueMatcher<T>(value = value, matcher = Matcher.NOT_EQUAL)
        fun <T> emptyList() =
            ValueMatcher<T>(matcher = Matcher.LIST_EMPTY)
        fun <T> notEmptyList() =
            ValueMatcher<T>(matcher = Matcher.LIST_NOT_EMPTY)
    }
}
```

```kotlin
class Figure private constructor(
    private val shape: Shape,
    private val length: Double? = null,
    private val width: Double? = null,
    private val radius: Double? = null,
) {
    enum class Shape {
        RECTANGLE,
        CIRCLE
    }

    fun area(): Double {
        return when (shape) {
            Shape.RECTANGLE -> this.length!! * this.width!!
            Shape.CIRCLE -> Math.PI * (radius!! * radius)
        }
    }

    companion object {
        fun createCircle(radius: Double): Figure {
            return Figure(
                shape = Shape.CIRCLE,
                radius = radius
            )
        }

        fun createRectangle(length: Double, width: Double): Figure {
            return Figure(
                shape = Shape.RECTANGLE,
                length = length,
                width = width,
            )
        }
    }
}
```

* 이렇게 사용했을 경우 여러 문제점이 있다.

* 한 클래스에 여러 모드를 처리하기 위한 상용구가 추가된다.
* 어떤 목적으로 사용해야 하므로 프로퍼티가 일관적이지 않게 사용될 수 있다. -> 위 예시에서 LIST_EMPTY 는 value 가 사용되지 않고, RECTANGLE 은 radius 가 사용되지 않는다.
* 요소가 여러 목적을 가지고, 요소를 여러 방법으로 설정할 수 있는 경우에는 상태의 일관성과 정확성을 지키기 어렵다.
* 팩터리 메서드를 사용해야하는 경우가 많다. 그렇지 않으면 객체가 제대로 생성되었는지 확인하는 것 자체가 어렵다.

* 코틀린은 일반적으로 태그 클래스보단 sealed 클래스를 더 많이 사용한다.
* 한 클래스에 여러 모드를 만드는 방법 대신, 각각의 모드를 여러 클래스로 만들고 타입 시스템과 다형성을 사용한다.

```kotlin
sealed class ValueMatcher<T> {
    abstract fun match(value: T): Boolean

    class Equal<T>(val value: T) : ValueMatcher<T>() {
        override fun match(value: T): Boolean = value == this.value
    }

    class NotEqual<T>(val value: T) : ValueMatcher<T>() {
        override fun match(value: T): Boolean = value != this.value
    }

    class EmptyList<T> : ValueMatcher<T>() {
        override fun match(value: T): Boolean = value is List<*> && value.isEmpty()
    }

    class NotEmptyList<T> : ValueMatcher<T>() {
        override fun match(value: T): Boolean = value is List<*> && value.isNotEmpty()
    }
}
```

```kotlin
sealed class Figure {
    abstract fun area(): Double

    class Rectangle(
        val length: Double,
        val width: Double,
    ) : Figure() {
        override fun area(): Double {
            return length * width
        }
    }
    
    class Circle(
        val radius: Double,
    ) : Figure() {
        override fun area(): Double {
            return Math.PI * (radius * radius)
        }
    }
}
```

* 이렇게 구현하면 책임이 분산되고, 필요한 데이터만 들어가게된다.
* Sealed 한정자를 사용하면, 외부에서 추가 서브클래스를 만들 수 없으므로, 타입이 추가되지 않을 것이 보장된다.
* 따라서, When 을 사용할 때 else 브랜치를 따로 만들 필요가 없다.

### 태그 클래스와 상태 패턴의 차이

* 태그 클래스, 상태 패턴을 혼동하면 안 된다. 
* 상태 패턴은 객체 내부가 변화할 때 객체 동작이 변하는 소프트웨어 디자인 패턴이다. 
* 상태 패턴은 프론트엔드 컨트롤러, 프레젠터, 뷰모델을 설계할 때 많이 사용된다. 
* 아침 운동을 위한 앱을 만들 때 각각의 운동 전에 준비 시간이 있다고 한다. 또한 운동을 모두 하고 나면 완료했다는 화면을 띄우려고 한다.
* 상태 패턴을 쓴다면 서로 다른 상태를 나타내는 클래스 계층 구조를 만들게 된다. 그리고 현재 상태를 나타내기 위한 읽고 쓸 수 있는 프로퍼티도 만들게 된다.

```kotlin
sealed class WorkoutState

class PrepareState(val exercise: Exercise): WorkoutState()

class ExerciseState(val exercise: Exercise): WorkoutState()

object DoneState: WorkoutState()

fun List<Exercise>.toStates(): List<WorkoutState> = flatMap { exercise ->
    listOf(PrepareState(exercise), ExerciseState(exercise))
} + DoneState

class WorkoutPresenter(/*...*/) {
    private var state: WorkoutState = states.first()
}
```

* 상태는 더 많은 책임을 가진 클래스다.
* 상태는 바꿀 수 있다.
* 구체 상태(concrete state)는 객체를 활용해서 표현하는 게 일반적이며 태그 클래스보단 sealed class 계층으로 만든다. 
* 또한 이를 불변 객체로 만들고 바꿔야 할 때마다 state 프로퍼티를 변경하게 만든다. 
* 그리고 뷰에서 이런 state 의 변화를 observe 한다.