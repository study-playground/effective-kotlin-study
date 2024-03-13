### compareTo 의 규약을 지켜라

* compareTo 메서드는 Any 클래스에 있는 메서드가 아니다.
* 이는 수학적인 부등식으로 변환되는 연산자이다.
* compareTo 는 Comparable<T> 인터페이스에도 들어있다.

```kotlin
obj1 > obj2 // obj1.compareTo(obj2) > 0 으로 변경
obj1 < obj2 // obj1.compareTo(obj2) < 0 으로 변경
obj1 >= obj2 // obj1.compareTo(obj2) >= 0 으로 변경
obj1 <= obj2 // obj1.compareTo(obj2) <= 0 으로 변경
```

* 비대칭적 동작 : a >= b 이고, b >= a 라면, a == b 여야 한다. 즉 비교와 동등성 비교에 어떠한 관계가 있어야 하며, 서로 일관성 있어야 한다.
* 연속적 동작 : a >= b 이고, b >= c 라면, a >= c 여야 한다.
* 코넥스적 동작 : 두 요소는 어떤 확실한 관계를 갖고 있어야 한다. 즉, a >= b 또는, b >= a 중에 적어도 하나는 항상 true 여야 한다.

### compareTo 를 따로 정의해야할까 ?

* 코틀린에서 compareTo 를 따로 정의하는 상황은 거의 없다.
* 일반적으로 어떤 프로퍼티 하나를 기반으로 순서를 지정하는 것으로 충분하기 때문이다.

```kotlin
data class Person(
    val name: String,
    val surname: String,
)

fun main() {
    val persons = listOf<Person>()

    // 원하는 키로 컬렉션 정렬
    persons.sortedBy { it.surname }

    // compareBy 를 활용하여 surname 기준 정렬 후 같으면 name 기준으로 정렬
    persons.sortedWith(compareBy({ it.surname }, { it.name }))
}
```

* 문자열은 알파벳과 숫자 등의 순서가 있다. 따라서 내부적으로 Comparable<String> 을 구현하고 있다.
* 텍스트는 일반적으로 알파벳과 숫자 순서로 정렬해야 하는 경우가 많으므로 굉장히 유용하다.
* 단점으로는 직관적이지 않은 부등호 기호를 기반으로 두 문자열을 비교하면 이해하는데 약간 시간이 걸릴 수 있다. // "Kotlin" > "Java" 이렇게 하는 것은 좋지 않다.
* 객체가 자연스러운 순서를 갖는 지 확실하지 않다면 비교기를 활용하는 것이 좋디. 

```kotlin
data class Person(
    val name: String,
    val surname: String,
) {
    companion object {
        val DEFAULT_ORDER = compareBy(Person::surname, Person::name)
    }
}

fun main() {
    val persons = listOf<Person>()
    // compareBy 를 활용하여 surname 기준 정렬 후 같으면 name 기준으로 정렬
    persons.sortedWith(Person.DEFAULT_ORDER)
}
```

### compareTo 구현하기

* compareTo 를 구현할 때 유용하게 활용할 수 있는 톱레벨 함수가 있다.
* 두 값을 단순하게 비교하기만 한다면, compareValues 함수를 활용할 수 있다.

```kotlin
data class Person(
    val name: String,
    val surname: String,
) : Comparable<Person> {
    override fun compareTo(other: Person): Int {
        compareValues(surname, other.surname)
    }

    // 더 많은 값을 비교할 때 사용
    override fun compareTo(other: Person): Int {
        compareValuesBy(this, other, { it.surname }, { it.name })
    }
}
```