### 제네릭 타입과 variance 한정자를 활용하라

### invariant (불공변성)
* 타입 파라미터는 기본적으로 invariant(불공변성)이다. ```ex. class Webtoon<T>```
* invariant 는 타입들이 서로 연관성이 없다는 뜻 이다.
* 예를 들어 ```Webtoon<Int>```, ```Webtoon<Number>```, ```Webtoon<Any>```, ```Webtoon<Nothing>``` 은 어떠한 연관성도 갖지 않는다.

```kotlin
class Webtoon<T>

fun main() {
    val anyWebtoon: Webtoon<Any> = Webtoon<Int> // ERROR
}
```

### covariant (공변성)
* out variance 한정자를 붙이면, 이는 타입 파라미터를 covariant (공변성)으로 만든다.
* 이는 A 가 B 의 서브타입일 때, Series<A> 가 Series<B> 의 서브타입이라는 의미이다.
* 기본 자바의 상속구조와 비슷하다.

```kotlin
class Series<out T>
open class Comic
class TrueBeauty : Comic()

fun main() {
    val comic: Series<Comic> = Series<TrueBeauty>()
    val trueBeauty: Series<TrueBeauty> = Series<Comic>() // ERROR
}
```

### contravariant (반변성)
* in variance 한정자를 붙이면, 이는 타입 파라미터를 contravariant (반변성)으로 만든다.
* 이는 A 가 B 의 서브타입일 때, Webtoon<A> 가 Webtoon<B> 의 슈퍼타입이라는 것을 나타낸다.

```kotlin
class Webtoon<in T>
open class Comic
class TowerOfGod : Comic()

fun main() {
    val comic: Webtoon<Comic> = Webtoon<TowerOfGod>() // ERROR
    val towerOfGod: Webtoon<TowerOfGod> = Webtoon<Comic>()
}
```

### 함수 타입
* 함수 타입은 파라미터 유형과 리턴 타입에 따라서 어떤 관계를 갖는다.
* 코틀린 함수 타입의 모든 파라미터 타입은 contravariant 이다.
* 코틀린 함수 타입의 모든 리턴 타입은 covariant 이다.
* 함수 타입을 사용할 때는 자동으로 variance 한정자가 사용된다.

![image](https://github.com/hongyeongjune/kotlin-playground/assets/39120763/f1d9063b-92cd-4b72-b32c-99cdff361f94)

```kotlin
fun main() {
    val intToNumber: (Int) -> Number = { it.toDouble() }
    val numberToAny: (Number) -> Any = { it.toString() }
    val numberToNumber: (Number) -> Number = { it }
    val numberToInt: (Number) -> Int = { it.toInt() }
    val anyToNumber: (Any) -> Number = { it.hashCode() }
    val anyToAny: (Any) -> Any = { it }

    process(intToNumber)
    process(numberToAny)
    process(numberToNumber)
    process(numberToInt)
    process(anyToNumber)
    process(anyToAny)
}

fun process(transition: (Int) -> Any) {
    println(transition(10))
}
```

### 자바 배열 covariant

* 자바의 배열은 covariant 인데, 이는 큰 문제가 발생할 수 있다.

```java
class Main {
    public static void main(String[]args){
        Integer[] numbers = {1,2,3,4};
        Object[] objects = numbers;
        objects[2] = "A"; // Runtime ERROR
    }
}
```

* numbers 를 Object[] 로 캐스팅해도 구조 내부에서 사용되고있는 실질적인 타입이 바뀌는 것은 아니다. (여전히 Integer)
* 따라서 이는 런타임 오류가 발생할 수 있다.
* 코틀린은 이를 해결하기 위해 Array(IntArray, CharArray) 등 을 invariant 로 만들었다.

### variance 한정자와 안전성

```kotlin
class SoccerTeam<out T> {
    private var team: T? = null
    fun get(): T = team ?: error("team not set")
    fun set(team: T) { // ERROR 발생
        this.team = team
    }
}
```

* ```covariant 타입 파라미터(out 한정자)가 in 한정자 위치(예를 들어 타입 파라미터)에 있다면, covariant 업캐스팅을 연결해서, 우리가 원하는 타입을 아무것이나 전달할 수 있습니다.``` -> 무슨 뜻인지 도저히 모르겠음

* 내가 이해한 것으로는 위 코드에서 set 이 불가능한 이유 (write 가 안되는 이유는 다음과 같다.)
* 아래 코드를 예시로 보자

```kotlin
open class EPL
class Liverpool: EPL()
class Tottenham: EPL()

fun copy(from: MutableList<out EPL>, to: MutableList<EPL>) {
    from[0] = Liverpool() // Type mismatch ERROR
    
    for (index in from.indices) {
        to[index] = from[index] // read 가능
    }
}
```

* 우선 공변성을 가지면 read-only 상태가 된다.
* read 가 가능한 이유는 out EPL 타입은 부모가 EPL 이고 타입이 EPL, Liverpool, Tottenham 중 하나일 것이기 이다. -> 이는 모두 EPL 에 할당할 수 있으므로 문제가 안된다.
* 하지만 예를 들어서 copy() 메서드에 from 으로 MutableList<Tottenham> 을 넘겼다고 가정해보자.
* 이럴 경우 from[0] 은 Tottenham 일 텐데 나는 강제로 Liverpool 을 할당해주고있다.
* 즉, 코틀린 컴파일러가 어떤 타입이 들어올지 알 수 없기 때문에 공변성을 적용하면 무조건 read-only 상태로 변하게된다.
* 책에서도 공변성 상태에서 read-only 상태가 된다고 설명하는 것 같은데, 글은 이해가 잘 안된다.

* 책에서 나타내는 또 다른 예시로 Response 를 설명한다.
* ```Response<T> 라면 T 의 모든 서브타입이 허용됩니다.``` 라는 말이 이해가 되지 않는다.
* 다음 코드는 되지 않는다.

```kotlin
class Response<T>

fun main() {
    val resp: Response<Any> = Response<Int> // ERROR
}
```

