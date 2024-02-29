### API 안정성을 확인하라

* 프로그래밍에서는 안정적이고 최대한 표준적인 API(Application Programming Interface)를 선호한다.
* 이유
  * API 가 변경되고 개발자가 이를 업데이트 했다면 다른 코드들에 영향을 줄 수 있다.
  * 사용자가 새로운 API 를 배워야한다.

* 좋은 API를 한번에 설계할수 없다. -> API 제작자는 계속해서 개선하기 위해 변경을 원한다.

* 일반적으로 버전을 활용해서 라이브러리와 모듈의 안정성을 나타내는데, 일반적으로는 시멘틱 버저닝 방법(Major, Minor, Patch)을 사용한다. 
  * Major : 호환되지 않는 수준의 API 변경 
  * Minor : 이전 변경과 호환되는 기능을 추가 
  * Patch : 간단한 버그 수정


> https://github.com/JetBrains/kotlin/releases/tag/v1.9.22
> 코틀린의 현 시점 최근 버전은 1.9.22 이다.
> major : 1
> minor : 9
> patch : 22

* 안정적인 API 일부를 변경해야 한다면, 전환하는 데 시간을 두고 Deprecated 어노테이션을 활용해서 사용자에게 미리 알려줘야 한다.
* 또한 직접적인 대안이 있다면, ReplaceWith 를 붙여주면 더욱 좋다.
```kotlin
@Deprecated("Use suspending getUsers instaed")
ReplaceWith("getUsers()")
fun getUsers(callback: (List<User>) -> Unit) {
    // ...
}
```
