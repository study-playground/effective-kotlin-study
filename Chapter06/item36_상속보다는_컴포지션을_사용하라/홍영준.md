### 상속을 사용하는 경우
* 상위 클래스와 하위 클래스가 is-a 관계인 경우
* ```A cat is an animal```
* ```A car is a vechicle```
* 즉, 원본 버전과 동일하나 좀 더 구체화한, 좀 더 확장한 버전으로 볼 수 있다.

### 컴포지션을 사용하는 경우
* 하나의 객체가 또 다른 객체를 has 하는 경우
* ```A car has a battery```
* ```A person has a heart```
* ex. Spring Data Jpa 에서 연관관계를 표현할 때

### 간단한 행위 재사용

```kotlin
class ProfileLoader {
    fun load() {
        // 프로그레스 바를 보여줌
        // 프로파일을 읽어 들임
        // 프로그레스 바를 숨김
    }
}

class ImageLoader {
    fun load() {
        // 프로그레스 바를 보여줌
        // 이미지를 읽어 들임
        // 프로그레스 바를 숨김
    }
}
```

* 대부분의 개발자는 위 코드 상황에서 슈퍼클래스를 만들어 공통 행위를 추출한다.

```kotlin
abstract class LoaderWithProgress {
    fun load() {
        // 프로그레스 바를 보여줌
        innerLoad() // 재정의
        // 프로그레스 바를 숨김
    }

    abstract fun innerLoad()
}

class ProfileLoader : LoaderWithProgress() {
    override fun innerLoad() {
        // 프로파일을 읽어들임
    }
}

class ImageLoader : LoaderWithProgress() {
    override fun innerLoad() {
        // 이미지를 읽어들임
    }
}
```

* 이러한 코드는 문제 없이 동작하지만 몇 가지 단점이 있다.
* 단점
  * 상속은 하나의 클래스만을 대상으로 할 수 있다. 상속을 사용해서 행위를 추출하다보면, 많은 함수를 갖는 거대한 BaseXXX 클래스가 생성되고 복잡한 계층구조를 가지게 된다.
  * 상속은 클래스의 모든 것을 가져오게된다. 따라서 불필요한 함수를 갖는 클래스가 만들어질 수 있다.
  * 상속은 이해하기가 어렵다. 개발자가 메서드를 읽고, 동작 방식을 알기 위해 슈퍼클래스를 여러 번 확인하는건 좋지 않다.

* 이럴 때 컴포지션을 사용하면 좋다. -> 객체를 프로퍼티로 갖고, 함수를 호출하는 형태로 재사용하는 것을 의미한다.
* 또한 컴포지션을 활용하면 하나의 클래스 내부애서 여러 기능을 재사용할 수 있다. 예를 들어, 프로파일을 읽고 경고창을 출력한다면 아래와 같은 형태로 구현할 수 있다.

```kotlin
class Progress {
    fun showProgress() { /*프로그레스 바를 보여줌*/ }
    fun hideProgress() { /*프로그레스 바를 숨김*/ }
}

class ProfileLoader {
    val progress = Progress()
    val finishedAlert = FinishedAlert()
    
    fun load() {
        progress.showProgress() // 프로그레스 바를 보여줌
        // 프로파일을 읽어 들임
        progress.hideProgress() // 프로그레스 바를 숨김
        finishedAlert.show() // 경고창 출력
    }
}
```

* 하나 이상의 클래스를 상속할 수는 없다. -> 상속으로 위 코드를 구현하면 하나의 슈퍼 클래스에 모든 기능을 배치해야하고, 이 때문에 클래스에 복잡한 계층 구조가 만들어질 수 있다.
* 따라서 슈퍼 클래스에 모든 기능을 넣으면 서브 클래스가 필요하지 않은 기능을 가질 수 있다. (필요한 것만 가져올 수는 없다.)

### 모든 것을 가져올 수 밖에 없는 상속

* 상속은 슈퍼클래스의 메서드, 제약, 행위 등 모든 것을 가져오므로 일부분을 재사용하기 위한 목적으로는 적합하지 않다.
* 즉, 일부분만 재사용하고 싶다면 컴포지션을 사용하는 것이 좋다.

### 캡슐화를 깨는 상속

```kotlin
class CounterSet<T> : HashSet<T>() {
  var elementsAdded: Int = 0
    private set

  override fun add(element: T): Boolean {
    elementsAdded++
    return super.add(element)
  }

  override fun addAll(elements: Collection<T>): Boolean {
    elementsAdded += elements.size
    return super.addAll(elements)
  }
}

fun main() {
  val set = CounterSet<String>()
  set.addAll(listOf("A", "B", "C"))
  println(set.elementsAdded) // 6 출력
}
```

* 위 코드는 6을 출력하게되는데 이유는 HashSet 의 addAll 내부에서 add 를 사용하고있기 때문에 ```elementsAdded++``` 가 3번 실행되기 때문이다.
* 해결방법으로 addAll 을 재정의하지 않으면, 정상적으로 3이 출력된다. -> 그러나, 이는 좋은 해결방법이 아니다. -> 어느날 자바가 HashSet 의 addAll 을 내부적으로 add 를 사용하지않고 최적화한 뒤 업데이트를 했다면, 이는 또다시 장애로 이어질 수 있다.
* 이를 해결하는 가장 좋는 방법은 컴포지션이다.

```kotlin
class CounterSet<T> {
  private val innerSet = HashSet<T>()
  var elementsAdded = 0
    private set

  fun add(element: T) {
    elementsAdded++
    innerSet.add(element)
  }

  fun addAll(elements: Collection<T>) {
    elementsAdded += elements.size
    innerSet.addAll(elements)
  }
}

fun main() {
  val set = CounterSet<String>()
  set.addAll(listOf("A", "B", "C"))
  println(set.elementsAdded) // 3 출력
}
```

* 위 처럼 컴포지션을 사용하면, 상속의 문제를 해결할 수 있지만 다형성이 사라지게된다.
* 이럴때는 코틀린의 위임 패턴을 사용하면된다. -> 위임패턴 : 클래스가 인터페이스를 상속받게 하고, 포함한 객체의 메서드들을 활용해서, 인터페이스에서 정의한 메서드를 구현하는 패턴

```kotlin
class CounterSet<T>(
    private val innerSet: MutableSet<T> = mutableSetOf()
) : MutableSet<T> by innerSet { // 해당 문법을 사용하면 컴파일 시점에 포워드 메서드들이 자동으로 만들어진다.
    var elementsAdded = 0
        private set

    override fun add(element: T): Boolean {
        elementsAdded++
        return innerSet.add(element)
    }

    override fun addAll(elements: Collection<T>): Boolean {
        elementsAdded += elements.size
        return innerSet.addAll(elements)
    }

    // 포워드 메서드
//    override val size: Int
//        get() = innerSet.size
}

fun main() {
    val set = CounterSet<String>()
    set.addAll(listOf("A", "B", "C"))
    println(set.elementsAdded)
}
```

### 오버라이딩 제한하기

* 개발자가 상속용으로 설계되지 않은 클래스를 상속하지 못하게 하려면, final 키워드를 사용하면 된다.
* 만약 상속은 허용하지만, 오버라이드는 못하게 막으려면 open 키워드를 사용하면 된다.

```kotlin
open class Parent {
  fun a() {}
  open fun b() {}
}

class Child : Parent() {
  override fun a() {} // 에러 
  override fun b() { super.b() }
}
```

### 정리
* 컴포지션은 더 안전하다. -> 다른 클래스의 내부적인 구현에 의존하지 않고, 외부에서 관촬되는 동작에만 의존하므로 안전
* 컴포지션은 더 유연하다. -> 상속은 한 클래스만을 대상으로 하지만, 컴포지션은 여러 클래스를 대상으로 할 수 있다. / 상속은 모든 것을, 컴포지션은 필요한 것만 받는다. / 슈퍼 클래스의 동작을 변경하면 서브 클래스의 동작도 큰 영향을 받는다.
* 컴포지션은 더 명시적이다. -> 슈퍼클래스의 메서드를 사용할 때는 리시버를 따로 지정하지 않아도 된다. (ex. this 키워드) 그래서 메서드가 어디에서 왔는지 혼동될 수 있다. 컴포지션을 활용하면 리시버를 명시적으로 활용해야해서 메서드가 어디에 있는 것인지 확실히 알 수 있다.
* 컴포지션은 번거롭다 -> 컴포지션은 객체를 명시적으로 사용해야 하므로, 대상 클래스에 일부 기능을 추가할 떄 이를 포함하는 객체의 코드를 변경해야 한다. (상속보다 코드를 수정할 일이 많아질 수 있다.)
* 상속은 다형성을 활용할 수 있다.