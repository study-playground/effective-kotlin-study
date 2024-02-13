### 코딩 컨벤션을 지켜라
> https://kotlinlang.org/docs/coding-conventions.html

* 코틀린은 굉장히 잘 정리된 코딩 컨벤션을 가지고 있다.

### 대표 예시
* 많은 파라미터를 갖고 있는 클래스는 각각의 파라미터를 한 줄씩 작성하는 방법을 사용한다.

```kotlin
class Person(
    val id: Int = 0,
    val name: String = "",
    val surname: String = ""
) : Human(id, name)
```

### 장점
* 어떤 프로젝트를 접해도 쉽게 이해할 수 있다.
* 다른 외부 개발자도 프로젝트의 코드를 쉽게 이해할 수 있다.
* 다른 개발자도 코드의 작동 방식을 쉽게 추측할 수 있다.
* 코드를 병합하고, 한 프로젝트의 코드 일부를 다른 코드로 이동하는 것이 쉽다.