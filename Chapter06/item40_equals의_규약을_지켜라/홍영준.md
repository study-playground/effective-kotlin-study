### equals 의 규약을 지켜라

* 코틀린의 Any 에는 다음과 같이 잘 설정된 규약들을 가진 메서드들이 있다.
  * equals
  * hashCode
  * toString
* 이러한 메서드 규약은 주석과 문서에 잘 설명되어있다.
* Any 클래스를 상속받는 모든 메서드는 이러한 규약을 잘 지켜주는 것이 좋다.

### 동등성

* 구조적 동등성 (structural equality): equals 메서드와 이를 기반으로 만들어진 == 연산자로 확인하는 동등성 -> a 가 nullable 이 아니라면 ```a.equals(b)``` nullable 이라면 ```a?.equals(b) ?: (b === null)``` 으로 반환
* 레퍼런스적 동등성 (referential equality): === 연산자로 확인하는 동등성, 두 연산자가 같은 객체를 가리키면 true 를 리턴
* equals 는 모든 클래스의 슈퍼 클래스인 Any 에 구현되어 있으므로, 모든 객체에서 사용할 수 있다.
* 다만 연산자를 사용해서 다른 타입의 두 객체를 비교할 수는 없다. (상속관계를 갖는 경우에는 가능)

### equals 가 필요한 이유

* Any 클래스에 구현되어 있는 equals 메서드는 디폴트로 === 처럼 두 인스턴스가 완전히 같은 객체인지 비교한다.

```kotlin
class Name(val name: String)

val name1 = Name("Hong")
val name2 = Name("Hong")
val name1Ref = name1

name1 == name2 // false
name1 == name1Ref // true
```

* 만약 두 객체가 기본 생성자의 프로퍼티가 같을 경우, 같은 객체로 봐야하는 형태가 있을 수 있다. -> data 한정자로 붙여서 클래스를 정의하면 자동으로 이와 같은 동등성으로 동작한다.

```kotlin
data class Name(val name: String)
val name1 = Name("Hong")
val name2 = Name("Hong")

name1 == name2 // true -> 데이터가 같기 때문
```

* 데이터 클래스의 동등성은 모든 프로퍼티가 아니라 일부 프로퍼티만 비교해야할 때도 유용하다. -> 이게 동등성이랑 무슨 연관이 있지?
* 아래 클래스처럼 작성할 경우 기본 생성자에 선언되지 않는 프로퍼티는 copy 로 복사되지 않는다. -> 이게 동등성이랑 무슨 연관이 있지?

```kotlin
data class Person(
    val name: String,
    val age: Int,
) {
    val cellPhone: String? = null
}
```

* 일반적으로 data 한정자 덕분에 코틀린에서는 equals 를 직접 구현할 필요가 없다.
* 다만, 일부 프로퍼티만 같은지 확인하려면 직접 구현이 필요하다.

### equals 의 규약

* 1.4.31 기준 equals 요구사항은 다음과 같다.

* 반사적 동작 : x 가 null 이 아니라면, x.equals(x) 는 true 리턴
* 대칭적 동작 : x 와 y 가 null 이 아닌 값이라면, x.equals(y) 는 y.equals(x) 와 같은 결과를 출력해야한다.
* 연속적 동작 : x, y, z 가 null 이 아닌 값이고, x.equals(y) 와 y.equals(z) 가 true 라면, x.equals(z) 도 true 여야한다.
* 일관적 동작 : x 와 y 가 null 이 아닌 값이라면, x.equals(y) 는 여러번 실행해도 같은 결과를 리턴해야한다.
* null 과 관련된 동작 : x 가 null 이 아닌 값이라면, x.equals(null) 은 항상 false 다.
* 추가로, equals, toString, hashCode 는 빠르게 동작해야하낟.

### URL 과 관련된 equals 문제
* equals 를 굉장히 잘못 설계한 예로 ```java.net.URL``` 이 있다.
* 객체 2개를 비교하면 동일한 IP 주소로 해석되면 true, 아니면 false 이다.
* 여기서 문제는 네트워크 상태에 따라 결과가 달라진다는 점 이다.

```kotlin
import java.net.URL

fun main() {
    val enWiki = URL("https://en.wikipeida.org/")
    val wiki = URL("https://wikipeida.org/")
    println(enWiki == wiki)
}
```

* 일반적인 상황에선 두 주소가 같은 IP 주소라서 true 이지만, 인터넷 연결이 끊어지면 false 를 리턴한다.

* 문제점
  * 동작이 일관적이지 않다.
  * 네트워크 처리를 하므로 속도가 느리다.
  * 동작에 문제가 있다. 동일한 IP 주소를 갖는다고 동일한 컨텐츠일 수 없다. 가상 호스팅을 사용하면 관련 없는 사이트가 같은 IP 주소를 공유할 수도있다.
* 안드로이드 4.0 버전부터 이러한 내용이 수정되었고, java.net.URI 를 사용해서 이런 문제를 해결할 수 있다.

### equals 구현하기

* 특별한 이유가 없는 이상, 직접 equals 를 구현하는 것은 좋지 않다.
* 기본적으로 제공되는 것을 쓰거나 데이터 클래스를 만들어서 사용하는 것이 좋다.
* 직접 구현해야 한다면, 요구사항이 맞는지 확인하고 final 클래스로 만드는 것이 좋다.
* 만약 상속을 한다면, 서브클래스에서 equals 가 작동하는 방식을 변경하면 안된다. 상속을 지원하면서 완벽한 사용자 정의 equals 를 만드는 것은 거의 불가능에 가깝다.