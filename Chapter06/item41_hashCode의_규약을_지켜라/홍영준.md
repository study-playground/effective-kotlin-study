### 해시 테이블

* 해시 테이블은 각 요소에 숫자를 할당하는 함수가 필요하다.
* 이 함수를 해시 함수라고 부르며, 같은 요소라면 항상 같은 숫자를 리턴한다.
* 해시 함수는 빠르고 충돌이 적을 수록 좋다.

텍스트|해시 코드
---|---
"홍영준"|3
"이펙티브 코틀린"|2
"이펙티브 자바"|1
"아이폰"|2

인덱스|해시 테이블이 가리키는 객체
---|---
0|[]
1|["이펙티브 자바"]
2|["이펙티브 코틀린", "아이폰"]
3|["홍영준"]

* 어떤 텍스트가 해시 테이블 안에 있는지 확인할 때는 해시 코드를 계산한다. 
* 0이면 리스트에 없다. 
* 1과 3이면 각각 하나의 텍스트와 비교한다. 
* 2면 2개의 텍스트와 비교한다. 
* 코틀린의 기본 Set 인 LinkedHashSet 와 기본 Map 인 LinkedHashMap 도 이를 사용한다. 
* 코틀린은 해시 코드를 만들 때 hashCode 함수를 사용한다.

### 가변성과 관련된 문제

* 요소가 추가될 때만 해시 코드를 계산하고, 요소가 변경되어도 해시 코드는 계산되지 않으며, 버킷 재배치도 이루어지지 않는다.
* 그래서 기본적인 LinkedHashSet 와 LinkedHashMap 의 키는 한번 추가하면 변경할 수 없다.

```kotlin
data class FullName(
    var name: String,
    var surname: String,
)

fun main() {
    val person = FullName("영준", "김")
    val set = mutableSetOf<FullName>()
    set.add(person)

    person.surname = "홍"

    println(person) // FullName(name=영준, surname=홍)

    println(person in set) // false -> set 에 들어갔을 때와 현재 person 의 hashCode 가 다름

    println(set.first()) // FullName(name=영준, surname=홍)

    println(set.first() == person) // true -> 프로퍼티가 같으므로 true
}
```

### hashCode 의 규약

* 어떤 객체를 변경하지 않았다면, hashCode 는 여러 번 호출해도 결과가 항상 같아야 한다.
* equals 메서드의 실행 결과로 두 객체가 같다고 나온다면, hashCode 메서드의 호출 결과도 같다고 나와야한다.