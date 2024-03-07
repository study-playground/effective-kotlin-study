### 생성자 대신 팩토리 함수를 활용하라

* 생성자 역할을 대신 해주는 함수를 ```팩터리 함수```라고 한다.
* 생성자 대신 팩터리 함수를 사용했을 때 장점
  * 함수에 이름을 붙일 수 있다. ```ex. ArrayList.withSize(3)```
  * 생성자와 다르게 함수가 원하는 형태의 타입을 리턴할 수 있다. -> 인터페이스 뒤에 실제 객체의 구현을 숨길 때 유용하게 사용할 수 있다. ex. listOf() 는 List 인터페이스를 리턴하는데, 플랫폼에 따라 코틀린/JVM, 코틀린/JS 등... 각 플랫폼의 빌트인 컬렉션으로 만들어진다.
  * 생성자와 다르게 호출될 때마다 새로운 객체를 만들 필요가 없다. ex. 함수를 사용하면 싱글턴 패턴처럼 객체를 하나만 생성하게 하거나, 최적화를 위한 캐싱 메커니즘도 사용할 수 있다.
  * 팩터리 함수는 아직 존재하지 않은 객체를 리턴할 수 있다. -> ??
  * 객체 외부에 팩터리 함수를 만들면, 그 가시성을 원하는 대로 제어할 수 있다. -> ex. 톱레벨 함수 가시성 제어
  * 팩터리 함수는 inline 으로 만들 수 있으며, 그 파라미터들을 reified 로 만들 수 있다. -> inline, reified 관련해서는 따로 정리
  * 팩터리 함수는 생성자로 만들기 복잡한 객체도 만들 수 있다.
  * 생성자는 즉시 슈퍼클래스 또는 기본 생성자를 호출해야하지만, 팩터리 함수는 원하는 때에 생성자를 호출할 수 있다. 
* 팩터리 함수는 굉장히 강력한 객체 생성 방법이다.
* 기본 생성자나 팩터리 함수 중 꼭 하나를 사용해야 하는 것은 아니다. -> 일반적인 자바로 팩터리 패턴을 구현할 때는 생성자를 private 으로 만들지만, 코틀린은 그렇게하는 경우가 거의 없다. (이름있는 옵션 아규먼트 때문)

### Companion 객체 팩터리 함수

* 팩터리 함수를 정의하는 가장 일반적인 방법

```kotlin
class MyLinkedList<T>(
    val head: T,
    val tail: MyLinkedList<T>?,
) {
    companion object {
        fun <T> of(vararg elements: T): MyLinkedList<T>? {
            /* 로직 수행 */
        }
    }
}
```

```kotlin
// 인터페이스로 팩터리 함수 생성
class MyLinkedList<T>(
  val head: T,
  val tail: MyLinkedList<T>?,
) : MyList<T>

interface MyList<T> {
  companion object {
    fun <T> of(vararg elements: T): MyLinkedList<T>? {
      /* 로직 수행 */
    }
  }
}

fun main() {
  val list = MyList.of(1, 2, 3)
}
```

* 정적 팩터리 함수 종류
  * from : 파라미터를 하나 받고, 같은 타입의 인스턴스 하나를 반환하는 함수
  * of : 파라미터를 여러 개 받고, 이를 통합해서 인스턴스를 만들어 주는 함수
  * valueOf : from 또는 of 와 비슷한 기능을 하면서도, 의미를 조금 더 쉽게 읽을 수 있게 이름을 붙인 함수
  * instance or getInstance : 싱글턴으로 인스턴스 하나를 반환하는 함수
  * createInstance or newInstance : 싱글턴이 적용되지 않아서 함수를 호출할 때마다 새로운 인스턴스를 만들어서 반환
  * getType : 싱글턴이고 팩터리 함수가 다른 클래스에 있을 때 사용하는 이름 ex. ```val fileStore: FileStore = Files.getFileStore(path)```
  * newType : 새로운 인스턴스를 반환하고 팩터리 함수가 다른 클래스에 있을 때 사용하는 이름

* companion 객체는 인터페이스를 구현할 수 있고 클래스를 상속받을 수도 있다. ex. 코틀린 코루틴 라이브러리는 거의 모든 코루틴 컨텍스트의 companion 객체가 컨텍스트 구별 목적으로 CoroutineContext.Key 인터페이스를 구현하고 있다.

```kotlin
/**
 * A [CoroutineContext] that propagates an associated [io.grpc.Context] to coroutines run using
 * that context, regardless of thread.
 */
class GrpcContextElement(private val grpcContext: GrpcContext) : ThreadContextElement<GrpcContext> {
  companion object Key : CoroutineContext.Key<GrpcContextElement> {
    fun current(): GrpcContextElement = GrpcContextElement(GrpcContext.current())
  }

  override val key: CoroutineContext.Key<GrpcContextElement>
    get() = Key

  override fun restoreThreadContext(context: CoroutineContext, oldState: GrpcContext) {
    grpcContext.detach(oldState)
  }

  override fun updateThreadContext(context: CoroutineContext): GrpcContext {
    return grpcContext.attach()
  }
}
```

### 확장 팩터리 함수

* 이미 Companion 객체가 존재할 때, 이 객체의 함수처럼 사용할 수 있는 팩터리 함수를 만들어야 할 때, 확장함수를 활용하면 좋다.

```kotlin
interface Tool {
    companion object { /*..*/ }
}

// 확장 함수 정의
fun Tool.Companion.createBigTool(): BigTool
```

### 톱레벨 팩터리 함수

* 대표적 예로, listOf(), setOf(), mapOf()
* public 톱레벨 함수는 모든 곳에서 사용할 수 있으므로, IDE 가 제공하는 팁을 복잡하게 만드는 단점이 있고, 함수 이름을 클래스 메서드 이름처럼 만들면 혼란을 일으킬 수 있어서 네이미을 신중하게 해야한다.

### 가짜 생성자

* 일반적인 사용의 관점에서 대문자로 시작하는지 아닌지는 생성자와 함수를 구분하는 기준이다.
* 예를 들어, List 와 MutableList 는 인터페이스라서 생성자를 가질 수 없는데, List 를 생성자처럼 사용하는 코드가 존재한다.
* List 는 코틀린 1.1 부터 stdlib 에 포함되었다.

```kotlin
public inline fun <T> List(
    size: Int,
    init: (index: Int) -> T
): List<T> = mutableList(size, init)

public inline fun mutableList(
    size: Int,
    init: (index: Int) -> T
): MutableList<T> {
    val list = ArrayList<T>(size)
    repeat(size) { index -> list.add(init(index)) }
    return list
}
```

* 이러한 코드는 톱레벨 함수처럼 보이고, 생성자처럼 동작한다. 그리고 팩터리 함수와 같은 모든 장점을 갖는다. -> 이를 ```가짜 생성자```라고 부른다.
* 가짜 생성자를 만드는 이유
  * 인터페이스를 위한 생성자를 만들고 싶을 때 
  * reified 타입 아규먼트를 갖게하고 싶을 때 

* 가짜 생성자는 invoke 연산자를 통해서도 만들 수 있는데, 이는 추천하지 않는다.
* 이는 item12 연산자 오버로드를 할 때는 의미에 맞게 해라 에 위배되기 때문

```kotlin
class Tree<T> {
    companion object {
        operator fun <T> invoke(size: Int, generator: (Int) -> T): Tree<T> {
            /*..*/
        }
    }
}
```

* 즉, 가짜 생성자는 톱레벨 함수를 사용하는 것이 좋다.