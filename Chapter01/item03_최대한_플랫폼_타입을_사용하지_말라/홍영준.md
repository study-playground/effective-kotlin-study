### 요약
* 다른 프로그래밍 언어에서 와서 nullable 여부를 알 수 없는 타입을 플랫폼 타입이라고 부른다.
* 이러한 플랫폼 타입을 사용하는 코드는 해당 부분만 위험할 뿐만 아니라, 이를 활용하는 곳까지 영향을 줄 수 있는 위험한 코드이다.
* 따라서, 플랫폼 타입 코드를 빠르게 제거하는 것이 좋다.
* 연결되어 있는 자바 생성자, 메서드, 필드에 nullable 여부를 지정하는 어노테이션을 적극 활용하자.

### 최대한 플랫폼 타입을 사용하지 말라
* 코틀린의 주요 기능 중 하나인 널 안정성 덕분에 NPE 를 자주보지는 않는다.
* 하지만, null-safety 메커니즘이 없는 자바, C 등의 프로그래밍 언어와 코틀린을 연결해서 사용할 때는 NPE 가 발생할 수 있다.
* 코틀린은 자바 등의 다른 프로그래밍 언어에서 넘어온 타입들을 특수하게 다루는데, 이러한 타입을 **플랫폼 타입**이라고 부른다.
* 플랫폼 타입은 타입 이름 뒤에 ! 기호를 붙여서 표기한다.
* 플랫폼 타입을 사용하는 경우 nullable 한 값을 다루기 때문에 안전하지 않기 때문에 가능하다면 최대한 빨리 플랫폼 타입을 활용하는 부분을 제거하는 것이 좋다.


```java
public class JavaClass {
    public String getValue() {
        return null;
    }
}

```

```kotlin
fun example1() {
    // 여기서 NPE 발생
    val value: String = JavaClass().value

    println(value.length)
}

fun example2() {
    // 플랫폼 타입
    val value = JavaClass().value

    // 여기서 NPE 발생
    println(value.length)
}
```

* 위의 example1(), example2() 모두 null 을 리턴한다고 가정하지 않으면 NPE 가 발생한다.
* example1() 은 자바에서 값을 가져오는 위치에서 NPE 가 발생하기 때문에 쉽게 수정할 수 있다.
* example2() 는 값을 활용할 때 NPE 가 발생한다. 즉, 코드가 복잡해지면 오류를 찾는 데 굉장히 오랜 시간이 걸릴 수 있다.