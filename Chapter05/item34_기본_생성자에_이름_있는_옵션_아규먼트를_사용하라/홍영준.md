### 점층적 생성자 패턴

* 점층적 생성자 패턴은 아래와 같이 만들 수 있다.

```kotlin
class Pizza {
    val size: String
    val cheese: Int
    val olives: Int
    val bacon: Int

    constructor(size: String, cheese: Int, olives: Int, bacon: Int) {
        this.size = size
        this.cheese = cheese
        this.olives = olives
        this.bacon = bacon
    }

    constructor(size: String, cheese: Int, olives: Int): this(size, cheese, olives, 0)

    constructor(size: String, cheese: Int): this(size, cheese, 0)

    constructor(size: String): this(size, 0)
}
```

* 코틀린은 default argument 로 더 좋은 코드를 만들 수 있다.

```kotlin
class Pizza(
    val size: String,
    val cheese: Int = 0,
    val olives: Int = 0,
    val bacon: Int = 0,
)
```

* 이처럼 default argument 는 코드를 단순하고 깔끔하게 만들어 줄 뿐만 아니라, 점층적 생성자보다 훨씬 다양한 기능을 제공한다.
* 장점
  * 파라미터를의 값을 원하는 대로 지정할 수 있다.
  * 아규먼트를 원하는 순서로 지정할 수 있다.
  * 명시적으로 이름을 붙여서 아규먼트를 지정하므로 의미가 훨씬 명확하다.

### 빌더 패턴

* 자바의 빌더 패턴보다 이름있는 파라미터를 사용하는 것이 좋은 이유를 간단히 정리하면 다음과 같다.

* 더 짧다 -> default argument 가 있는 생성자 또는 팩터리 메서드가 빌더 패턴보다 구현하기 쉽다.
* 더 명확하다. -> 객체가 어떻게 생성되는지 확인하고 싶을 때, 빌더 패턴은 여러 메서드들을 확인해야 한다. 하지만 default argument 가 있는 코드는 생성자 주변만 확인하면 된다.
* 더 사용하기 쉽다. -> 기본 생성자는 기본적으로 언어에 내장된 개념이다. 하지만 builder 패턴은 언어 위에 추가로 구현한 개념이라 이해가 필요하다.
* 동시성과 관련이 없다. -> 코틀린의 함수 파라미터는 항상 immutable 이다. 반면 대부분의 빌더 패턴에서 프로퍼티는 mutable 이다. 따라서 thread-safe 하지 않다.

* 빌더 패턴은 값의 의미를 묶어서 지정할 수 있거나 특정 값을 누적하는 형태로 사용할 수 있다.

```kotlin
// 값에 의미 부여
val dialog = AlertDialog.Builder(context)
    .setMessage(R.string.fire_missiles)
    .setPositiveButton(R.string.fire, { d, id -> /*로직 수행*/ })
    .setNegativeButton(R.string.cancel, { d, id -> /*로직 수행*/ })
    .create()

// 특정 값 누적
val router = Router.Builder()
    .addRoute(/*..*/)
    .addRoute(/*..*/)
    .build()
```

```kotlin
// 빌더패턴 없이 사용
val dialog = AlertDialog(
    context,
    message = R.string.fire_missiles,
    positiveButtonDescription = ButtonDescription(R.string.fire, { d, id -> /*로직 수행*/ }),
    negativeButtonDescription = ButtonDescription(R.string.cancel, { d, id -> /*로직 수행*/ })
)

val router = Router(
    routes = listOf(
        Route(/*..*/),
        Route(/*..*/),
    )
)
```

* 위 코드는 코틀린 DSL 빌더로 편하게 만들 수 있다.

```kotlin
val dialog = context.alert(R.string.fire_missiles) {
    positiveButton(R.string.fire) {
        /*..*/
    }
    negativeButton {
        /*..*/
    }
}

val router = router {
    /*..*/
    /*..*/
}
```

* 이렇게 DSL 빌더를 사용하는 것이 전통적인 빌더 패턴보다 훨씬 유연하고 명확하고 코틀린은 이와 같은 형태의 코드를 많이 사용한다.

