### 변화로부터 코드를 보호하려면 추상화를 사용하라

* 함수와 클래스 등의 추상화로 실질적인 코드를 숨기면, 사용자가 세부 사항을 알지 못해도 괜찮다는 장점이 있다.
* 그리고 이후에 실질적인 코드를 원하는대로 수정할 수도 있다.

### 상수

* 리터럴은 아무것도 설명하지 않는다.
* AS-IS 의 숫자 7은 이해하는데 시간이 걸릴 수 있지만, 상수로 뺀다면 이해하기 훨씬 쉬워진다.
* 상수로 추출하면 다음과 같은 장점이 있다.
    * 이름을 붙일 수 있다.
    * 나중에 해당 값을 쉽게 변경할 수 있다.

#### AS-IS

```kotlin
fun isPasswordValid(text: String): Boolean {
    if (text.length < 7) return false

    // ... 로직 수행
    return true
}
```

#### TO-BE

```kotlin
const val MIN_PASSWORD_LENGTH = 7

fun isPasswordValid(text: String): Boolean {
    if (text.length < MIN_PASSWORD_LENGTH) return false

    // ... 로직 수행
    return true
}
```

### 함수

* 일반적인 알고리즘을 추출하면, 구체적인 코드를 항상 기억해 두지 않아도 괜찮고, 코드가 변경되어도 추출한 부분만 수정하면 되므로 유지보수성이 향상된다.

* 다만 내부적으로만 사용하더라도, 함수의 이름을 직접 바꾸는 것은 위험할 수 있다.
* 다른 모듈이 이 함수에 의존하고 있다면, 다른 모듈에 큰 문제가 발생할 것 이다.

* 예를들어, 토스트를 추상화하여 사용하고 있는데, 토스트를 스낵바로 변경해서 사용해야 한다고 가정하자.
* snackbar 를 이용해 확장함수로 만들고 기존의 Context.toast() 를 Context.snackbar() 로 일괄 변경한다.
    * 위 내용 처럼 이름을 직접 바꾸는 건 위험할 수 있기 때문에 다른 방법을 고려하는 것이 좋다.
* 토스트 출력이라는 개념과 무관하게 showMessage 라는 더 높은 레벨의 함수로 옮긴다.
* 이때 가장 큰 변화는 이름(eg. showToast() > showMessage()) 인데, 일부 개발자는 이름 변경을 큰 변화가 아니라고 생각할 수도 있지만, 이것은 컴파일러의 관점에서만 유효한 것이다.
* 사람의 관점에서는 이름이 바뀌면 큰 변화가 일어난 것이다.
* 함수는 추상화를 표현하는 수단이며, 함수 시그니처는 이 함수가 어떤 추상화를 표현하고 있는지 알려주므로 의미 있는 이름은 매우 중요하다.
* 구현을 추상화할 수 있는 더 강력한 방법으로는 클래스가 있다.

#### AS-IS

```kotlin
fun Context.toast(
    message: String,
    duration: Int = Toast.LENGTH_LONG,
) {
    Toast.makeText(this, message, duration).show()
}

context.toast("message")
```

#### TO-BE1

```kotlin
fun Context.snackbar(
    message: String,
    duration: Int = Toast.LENGTH_LONG,
) {
    // ...
}
```

#### TO-BE2

```kotlin
fun Context.showMessage(
    message: String,
    duration: MessageLength = MessageLength.LONG
) {
    val toastDuration = when (duration) {
        MessageLength.SHORT -> Toast.LENGTH_SHORT
        MessageLength.LONG -> Toast.LENGTH_LONG
    }
    Toast.makeText(this, message, toastDuration).show()
}

enum class MessageLength { SHORT, LONG }
```

### 클래스

* 클래스가 함수보다 강력한 이유는 상태를 가질 수 있으며, 많은 함수를 가질 수 있다는 점 때문이다.
* 클래스는 훨씬 더 많은 자유를 보장해준다.

* 아래 코드를 보면, context 는 의존성 주입이 될 수 있으며, mock 을 이용해 해당 클래스에 의존하는 다른 클래스의 기능을 테스트할 수 있다.
* 하지만 여전히 한계가 있으며, open 을 사용해 서브클래스를 제공하거나 인터페이스를 사용하는 것이 더 자유를 제공할 수 있다.

```kotlin
class MessageDisplay(val context: Context) {
    fun show(
        message: String,
        duration: MessageLength = MessageLength.SHORT
    ) {
        val toastDuration = when (duration) {
            MessageLength.SHORT -> Toast.LENGTH_SHORT
            MessageLength.LONG -> Toast.LENGTH_LONG
        }
        Toast.makeText(context, message, toastDuration).show()
    }
}

enum class MessageLength { SHORT, LONG }

//사용
val messageDisplay = MessageDisplay(context)
massageDisplay.show("message")
```

### 인터페이스

* 코틀린 표준 라이브러리를 읽어보면, 거의 모든 것이 인터페이스로 표현된다는 것을 확인할 수 있다.
* 라이브러리를 만드는 사람은 내부 클래스의 가시성을 제한하고, 인터페이스를 통해 이를 노출하는 코드를 많이 사용한다.
* 이렇게 하면 사용자가 클래스를 직접 사용하지 못하므로, 라이브러리를 만드는 사람은 인터페이스만 유지한다면, 별도의 걱정 없이 자신이 원하는 형태로 그 구현을 변경할 수 있다.
* 즉, 인터페이스 뒤에 객체를 숨김으로써 실질적인 구현을 추상화하고, 사용자가 추상화된 것에만 의존하게 만들 수 있는 것 이다. -> 결합을 줄일 수 있다.

* 아래 코드는 이제 토스트 출력, 스낵바 출력이 가능하고 안드로이드, iOS, 웹에서 공유해서 사용하는 공통 모듈로 만들 수도 있다.
* 또 다른 장점은 테스트할 때 인터페이스 faking 이 클래스 mocking 보다 간단하므로, 별도의 mocking 라이브러리를 사용하지 않아도 된다.
* 마지막으로 선언과 사용이 분리되어 있으므로, ToastDisplay 등의 실제 클래스를 자유롭게 변경할 수 있다.
* 다만 사용 방법을 변경하려면 인터페이스를 변경하고 이를 구현하는 모든 클래스를 변경해야 한다.

```kotlin
interface MessageDisplay {
    fun show(message: String, duration: MessageLength = MessageLength.LONG)
}

class ToastDisplay(val context: Context) : MessageDisplay {
    override fun show(message: String, duration: MessageLength) {
        val toastDuration = when (duration) {
            MessageLength.SHORT -> Toast.LENGTH_SHORT
            MessageLength.LONG -> Toast.LENGTH_LONG
        }
        Toast.makeText(context, message, toastDuration).show()
    }
}

enum class MessageLength { SHORT, LONG }
```

### ID 만들기 (nextId)

```kotlin
var nextId: Int = 0

// 사용
// 이 코드의 ID 는 무조건 0 부터 시작함
// 이 코드의 스레드는 안전하지 않다.
val newId = nextId++
```

```kotlin
private var nextId: Int = 0
fun getNextId(): Int = nextId++

// 사용
// 이제 ID 타입 변경 등에 대응하지 못한다.
val newId = getNextId()
```

```kotlin
// ID 타입을 쉽게 변경할 수 있게 클래스로 사용
data class Id(val id: Int)

private var nextId: Int = 0
// 더 많은 추상화는 자유를 주지만, 이를 정의하고 사용하고 이해하는 것은 어려워졌다.
fun getNextId(): Id = Id(nextId++)
```

### 추상화가 주는 자유

* 추상화 방법 정리
  * 상수로 추출한다.
  * 동작을 함수로 래핑한다.
  * 함수를 클래스로 래핑한다.
  * 인터페이스 뒤에 클래스를 숨긴다.
  * 보편적인 객체를 특수한 객체로 래핑한다.

* 추상화는 자유를 주지만, 코드를 이해하고 수정하기 어렵게 만든다.

### 추상화의 문제

* 어떤 방식으로 추상화를 하려면, 코드를 읽는 사람이 해당 개념을 배우고, 잘 이해해야한다. 
* 또 다른 방식으로 추상화를 하려면 또 해당 개념을 배우고, 잘 이해해야한다.

* 추상화는 많은 것을 숨길 수 있는 테크닉이다.
* 생각할 것을 어느 정도 숨겨야 개발이 쉬워지는 것도 사실이지만, 너무 많은 것을 숨기면 결과를 이해하는 것 자체가 어려워진다.

### 어떻게 균형을 맞춰야 할까 ?

* 모든 추상화는 자유를 주지만, 코드가 어떻게 돌아가는 것인지 이해하기 어렵게 만든다.
* 극단적인 것은 언제나 좋지 않다. 
* 최상의 답은 언제나 그 사이 어딘가에 있다.

* 정확한 위치는 다음과 같은 요소들에 따라서 달라질 수 있다.
  * 팀의 크기
  * 팀의 경험
  * 프로젝트의 크기
  * 특징 세트 (feature set)
  * 도메인 지식

* 균형을 잡는 것은 어려운 일이지만, 그래도 사용할 수 있는 몇 가지 규칙을 정리하면 다음과 같다.
  * 많은 개발자가 참여하는 프로젝트는 이후에 객체 생성과 사용 방법을 변경하기 어렵다. 따라서 추상화 방법을 사용하는 것이 좋다. 최대한 모듈과 부분을 분리하는 것이 좋다.
  * 의존성 주입 프레임워크를 사용하면, 생성이 얼마나 복잡한지는 신경 쓰지 않아도 된다.
  * 테스트를 하거나, 다른 애플리케이션을 기반으로 새로운 애플리케이션을 만든다면 추상화를 사용하는 것이 좋다.
  * 프로젝트가 작고 실험적이라면, 추상화를 하지 않고도 직접 변경해도 괜찮다.