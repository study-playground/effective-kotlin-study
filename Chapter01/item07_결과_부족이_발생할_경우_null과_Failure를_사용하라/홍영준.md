### 요약

### 결과 부족이 발생할 경우 null 과 Failure 를 사용하라
* 함수가 원하는 결과를 만들어 낼 수 없을 때 처리하는 메커니즘은 크게 두 가지가 있다.
  * null 또는 '실패를 나타내는 sealed class (일반적으로 Failure 라는 이름을 붙입니다.)를 리턴'
  * 예외를 throw 한다.

### 에외를 throw 한다.
* 예외는 정보를 전달하는 방법으로 사용해서는 안된다.
* 즉, 예외는 특별한 상황을 나타내야 하며, 처리되어야 한다. -> 예외는 예외적인 상황이 발생했을 때 사용하는 것이 좋다.

#### 이유
* 많은 개발자가 예외가 전파되는 과정을 제대로 추적하지 못한다.
* 코틀린의 모든 예외는 unchecked 이기 때문에, 사용자가 예외를 처리하지 않을 수도 있으며 문서에 재대로 드러나지 않을 수 있다.
* 예외는 예외적인 상황을 처리하기 위해서 만들어졌으므로 명시적인 테스트만큼 빠르게 동작하지 않는다.
* try-catch 블록 내부에 코드를 배치하면, 컴파일러가 할 수 있는 최적화가 제한된다.

### null 또는 '실패를 나타내는 sealed class (일반적으로 Failure 라는 이름을 붙입니다.)를 리턴'
* null 과 Failure 는 예상되는 오류를 표현할 때 굉장히 좋다.
* 이는 명시적이고, 효율적이며, 간단한 방법을 처리할 수 있다.
* 따라서 충분히 예측할 수 있는 범위의 오류는 null 과 Failure 를 사용하고, 예측하기 어려운 예외적인 범위의 오류는 예외를 throw 해서 처리하는 것이 좋다.
  * **질문**
    * 충분히 예측할 수 있는 범위의 오류는 어떤게 있을까 ?
    * 예측하기 어려운 예외적인 범위의 오류는 무엇일까 ?
    * 예외를 던지는게 더 좋지 않을까 ? (ex. 내 정보 조회 -> 없으면 null 리턴 or Exception 발생 -> 어떤게 더 좋은 방법일까 ?)
    * readObjectOrNull 은 네이밍 상 Object 혹은 Null 리턴이니까 그렇다쳐도 Sealed class (Failure) 를 이용하는게 예외를 던지는 것 보다 정말 좋은 방법일까 ?
  * **답변1 : 현 카카오페이 백엔드 개발자**
    * 팀 내에서 컨벤션으로 예외를 던지지 않고, 200 status 에 Sealed class 를 사용하고있다고함 (code 커스텀) -> FE 와 BE 가 서로 협의하는 부분이라고 생각한다고함
  * **답변2 : 현 쏘카 안드로이드 개발자**
    * Failure 를 사용하는 것이 더 좋다고 생각한다. (실제로 팀에서 그렇게 쓰고있음)
    * 예외의 갯수나 타입이 명확히 지정되어있지 않다면 그 외의 모든 예외를 when을 사용하더라도 나머지는 else 로 처리해야한다.
    * 그럼 이게 저기로 빠져버리면 무슨 에러인지 핸들링하기 아주 난해해져서 BE 와 협의해서 Sealed Class 로 Union Type 을 만들어서 사용하고 있다고함
    * 언제 어떤 예외가 보내질지 예측이 불가능한 경우가 생긴다면 unknown 타입이라는걸 넣어놓고 그걸 응답해달라고 BE 한테 말해서 그렇게 받고 있음
    * 정리하면, 타협하기 나름이지만 앱 개발자 입장에서 둘 다 써보니까 Failure 같은 유니온 타입을 사용하는게 더 편하더라

```kotlin
// null return
// inline - reified 제네릭 타입 사용
inline fun <reified T> String.readObjectOrNull(): T?{
    // ...
    if (incorrectSign) {
        return null
    }
    // ...
    return result
}

// Sealed Class 이용
fun <reified T> String.readObject(): Result<T>{
    //...
    if(incorrectSign) {
        return Failure(JsonParsingException())
    }
    // ...
    retrun Success(result)
}

sealed class Result<out T>

class Success<out T>(val result: T): Result<T>()
class Failure(val throwable: Throwable): Result<Nothing>()

class JsonParsingException: Exception()
```

* null 을 처리해야한다면, Elvis 연산자 같은 다양한 null-safety 기능을 활용하면 좋다.

```kotlin
fun main() {
    val age = userText.readObjectOrNull<Person>()?.age ?: -1
}
```

* Result 와 같은 공용체를 리턴하기로 했다면, when 표현식으로 처리할 수 있다.
```kotlin
fun main() {
    val person = userText.readObject<Person>()
    val age = when (person) {
        is Success -> person.age
        is Failure -> -1
    }
}
```

* 이러한 오류 처리 방식은 try-catch 블록보다 효율적이며, 사용하기 쉽고 더 명확하다.
* 예외는 놓칠 수 있으며, 전체 애플리케이션을 중지시킬 수도 있다.
* null 값과 sealed result 클래스는 명시적으로 처리해야 하며, 애플리케이션의 흐름을 중지하지도 않는다.