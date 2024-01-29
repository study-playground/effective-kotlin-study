### 요약
* 타입을 확실하게 지정해야 하는 경우에는 명시적으로 타입을 지정해야 한다는 원칙만 갖고 있으면 된다.
* 안전을 위해서 외부 API 를 만들 때는 반드시 타입을 지정하고, 이렇게 지정한 타입을 특별한 이유와 확실한 확인 없이 제거하지 않는 것이 좋다.

### inferred 타입으로 리턴하지 말라
* 할당 떄 inferred 타입은 정확하게 오르쪽에 있는 피연산자에 맞게 설정된다. 
* 절대로 슈퍼클래스 또는 인터페이스로는 설정되지 않는다.

```kotlin
fun main() {
    // type mismatch
    var webtoon1 = TrueBeauty()
    webtoon1 = Webtoon()

    // TrueBeauty 는 Webtoon 을 상속받고 있기 때문에 해당 코드는 가능 (타입추론)
    var trueBeauty = Webtoon()
    trueBeauty = TrueBeauty()

    // 타입 명시적으로 지정해주면 type mismatch 해결
    var webtoon2: Webtoon = Kubera()
    webtoon2 = Webtoon()

    // type mismatch
    var kubera: Kubera = Webtoon()
}

open class Webtoon
class TrueBeauty: Webtoon()
class Kubera: Webtoon()
```

* 직접 라이브러리(또는 모듈)를 조작할 수 없는 경우에는 이러한 문제를 명시적으로 지정하는거처럼 간단하게 해결할 수 없다.
* 그리고, 이러한 경우에 inferred 타입을 노출하면, 위험한 일이 발생할 수 있다.

* WebtoonFactory 인터페이스에 DEFAULT_WEBTOON 을 리턴하는 메서드가 있다.
* DEFAULT_WEBTOON 에 명시적으로 지정되어있던 리턴타입(Webtoon)을 제거
* DEFAULT_WEBTOON 의 타입 추론에 의해 자동으로 타입이 지정되고 TrueBeauty 로만 지정이된다.
* 따라서, override 시 리턴 타입이 TrueBeauty 외에는 아무것도 만들 수 없게된다.
* 우리가 인터페이스를 만들었다면 쉽게 찾을 수 있지만, 외부 API 라면 찾을 수 없다.

```kotlin
open class Webtoon
class TrueBeauty: Webtoon()

// val DEFAULT_WEBTOON: Webtoon = TrueBeauty()
val DEFAULT_WEBTOON = TrueBeauty()

interface WebtoonFactory {
    fun produce() = DEFAULT_WEBTOON
}

class WebtoonFactoryImpl : WebtoonFactory {
    // 에러 발생 !!
    override fun produce(): Webtoon {
        return super.produce()
    }
}
```
