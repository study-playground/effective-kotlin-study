### API 의 필수적이지 않는 부분을 확장 함수로 추출하라

* 클래스의 메서드를 정의할 때는 메서드를 멤버로 정의할 것인지, 확장 함수로 정의할 것인지 결정해야 한다.

```kotlin
import java.time.LocalDateTime

class Person() {
    fun makeEvent(dateTime: LocalDateTime): Event = //...
}

fun Person.makeEvent(dateTime: LocalDateTime): Event = //...
```

* 두 방법은 호출하는 방식도 비슷하고, 리플렉션으로 레퍼런싱 하는 것도 비슷하다.
* 두 방식 중 어떤 방식이 우월하다고 할 수 없다. 장단점이 모두 있고 상황에 따라 다르게 쓰면 된다.

* 일반 멤버와 확장의 가장 큰 차이점은 확장은 따로 가져와서 사용해야 한다는 것 이다. -> 일반적으로 확장은 다른 패키지에 위치
* 확장은 우리가 직접 멤버를 추가할 수 없는 경우 데이터와 행위를 분리하도록 설계된 프로젝트에서 사용한다.
* 필드가 있는 프로퍼티는 클래스에 있어야 하지만, 메서드는 public API 만 활용하면 어디에 있어도 상관없기 때문이다.
* 임포트해서 사용한다는 특징 때문에 확장은 같은 타입에 같은 이름으로 여러개를 만들 수 있다. -> 하지만, 같은 이름으로 다른 동작을 하는 확장은 위험할 수 있다.

* 확장은 가상이 아니다. 즉, 파생 클래스에서 오버라이드할 수 없다. -> 확장 함수는 컴파일 시점에 정적으로 선택된다.
* 따라서 상속을 목적으로 설계된 요소는 확장 함수로 만들면 안된다.

```kotlin
open class C
class D: C()
fun C.foo() = "c"
fun D.foo() = "d"

fun main() {
    val d = D()
    println(d.foo()) //d
    val c: C = d
    println(c.foo()) //c
    
    println(D().foo()) //d
    println((D() as C).foo()) //c
}
```

* 확장 함수는 클래스가 아닌 타입에 정의하는 것이다. 그래서 nullable 또는 구체적인 제네릭 타입에도 확장 함수를 정의할 수 있다.

```kotlin
inline fun CharSequence?.isNullOrBlank(): Boolean {
    contract {
        return(false) implies (this@isNullOrBlank != null)
    }

    return this == null || this.isBlank()
}

public fun Iterable<Int>.sum(): Int {
    var sum: Int = 0
    for (element in this) {
        sum += element
    }
    return sum
}
```

* 확장은 클래스 레퍼런스에서 멤버로 표시되지 않는다.
* 그래서 확장 함수는 어노테이션 프로세서가 따로 처리하지 않는다.

### 정리

* 확장 함수는 우리에게 더 많은 자유와 유연성을 준다.
* 확장 함수는 상속, 어노테이션 처리 등을 지원하지 않고 클래스 내부에 없으므로 약간 혼동을 줄 수 있다.
* API의 필수적인 부분은 멤버로 두는 것이 좋지만, 필수적이지 않다면 확장 함수로 만드는 것이 여러모로 좋다.