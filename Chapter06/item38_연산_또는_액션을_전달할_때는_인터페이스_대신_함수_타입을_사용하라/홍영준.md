### 연산 또는 액션을 전달할 때는 인터페이스 대신 함수 타입을 사용하라

* 자바에서 SAM(Single-Abstract Method) 은 추상 매소드가 단 하나인 인터페이스의 특성을 나타내는 것으로 SAM 을 만족하는 인터페이스만 익명클래스, 람다표현식으로 구현될 수 있다.
* 대부분 프로그래밍에서는 함수타입이 없는데, 그래서 역산 또는 액션을 전달할 때 SAM 을 활용하여 전달한다.

```kotlin
fun setOnClickListener(listener: OnClick) {
    // ...
}

setOnClickListener(object : OnClick {
    override fun clicked(view: View) {
        // ...
    }
})
```

* 위 코드를 함수 타입을 사용하는 코드로 변경하면, 더 많은 자유를 얻을 수 있다.

```kotlin
// 함수 타입을 사용하도록 변경
fun setOnClickListener(listener: (String) -> Unit) {
    listener("This is listener")
}

// 선언된 함수 타입을 구현한 객체로 전달
class PrintListener: (String) -> Unit {
    override fun invoke(p1: String) {
        println(p1)
    }
}

fun main() {
    // 람다 표현식으로 전달
    setOnClickListener {
        println(it)
    }

    // 익명 함수로 전달
    setPrintListener(
        fun(str) {
            println(str)
        }
    )

    // 함수 레퍼런스로 전달
    setPrintListener(::println)
 
    // 선언된 함수 타입을 구현한 객체로 전달
    setPrintListener(PrintListener())
}
```

* SAM 의 장점 중 하나인 아규먼트에 이름을 붙이는 것 또한, 타입 별칭을 사용하면 똑같이 구현할 수 있다.

```kotlin
typealias OnPrintListener = (String) -> Unit

fun setTypeAliasPrintListener(listener: OnPrintListener) {
    listener("This is new listener")
}
```

### 언제 SAM 을 사용해야 할까 ?

* 코틀린이 아닌 다른 언어에서 사용할 클래스를 설계할 때는 SAM 을 사용하는 것이 좋다.