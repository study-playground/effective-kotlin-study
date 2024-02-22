### 타입 파라미터의 섀도잉을 피하라

* 프로퍼티와 파라미터가 같은 이름을 가질 수 있다. -> 지역 파라미터가 외부 스코프에 있는 프로퍼티를 가리게된다. -> 이를 섀도잉이라 한다.

```kotlin
class Webtoon(val name: String) {
    fun addComic(name: String) {
        println(name)
    }
}

fun main() {
    val webtoon = Webtoon("여신강림")
    webtoon.addComic("신의탑")
    // 신의탑 출력
}
```

* 섀도잉 현상은 클래스 타입 파라미터와 함수 타입 파라미터 사이에서도 발생한다.
* 개발자가 제네릭을 제대로 이해하지 못할 때, 다양한 문제가 발생할 수 있다.

```kotlin
interface Novel
class ReturnOfTheMadDemon: Novel
class ADanceOfSwordsInTheNight: Novel

class Series<T: Novel> {
    // Series 와 addNovel 의 파라미터가 독립적으로 동작
    fun <T: Novel> addNovel(novel: T) {
        println(novel)
    }
}

fun main() {
    // ReturnOfTheMadDemon 타입으로 생성
    val series = Series<ReturnOfTheMadDemon>()
    series.addNovel(ReturnOfTheMadDemon())
    // ADanceOfSwordsInTheNight 타입도 넣을 수 있게 된다.
    series.addNovel(ADanceOfSwordsInTheNight())
}
```

```kotlin
interface Novel
class ReturnOfTheMadDemon: Novel
class ADanceOfSwordsInTheNight: Novel

class Series<T: Novel> {
    // class 타입 파라미터를 재사용
    fun addNovelReuseType(novel: T) {
        println(novel)
    }
}

fun main() {
    val series = Series<ReturnOfTheMadDemon>()
    series.addNovelReuseType(ReturnOfTheMadDemon())
    series.addNovelReuseType(ADanceOfSwordsInTheNight()) // Type ERROR
}
```