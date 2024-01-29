### 요약

### 적절하게 null 을 처리하라
* ```String.toIntOrNull()``` 은 String 을 Int 로 바꿀 수 없을 경우 null 을 리턴한다.
* 이처럼 null 은 최대한 명확한 의미를 갖는 것이 좋다. -> nullable 값을 처리해야하기 때문인데, 이를 처리하는 사람은 API 사용자이다.

```kotlin
data class Webtoon(
    val subject: String
)

fun getWebtoon(): Webtoon? {
    return null
}

fun example1() {
    val webtoon: Webtoon? = getWebtoon()

    // 컴파일 오류
    webtoon.subject

    // 안전 호출
    webtoon?.subject

    // 스마트 캐스팅
    if (webtoon != null) webtoon.subject

    // not-null assertion
    webtoon!!.subject
}
```

* null 을 안전하게 처리하는 방법은 다음과 같다.
  * ```?.```, 스마트 캐스팅, Elvis 연산자 등을 활용해서 안전하게 처리한다.
  * 오류를 throw 한다.
  * 함수 또는 프로퍼티를 리팩터링해서 nullable 타입이 나오지 않게 바꾼다.

### null 을 안전하게 처리하기
* 안전 호출과 스마트 캐스팅이 있다.
* nullable 변수와 관련된 처리를 광범위하게 지원하는데, 대표적으로 인기있는 방법은 Elvis 연산자 사용이다. Elvis 연산자는 오른쪽에 return 또는 throw 를 포함한 모든 표현식이 허용된다.

```kotlin
data class Webtoon(
    val subject: String
)

fun getWebtoon(): Webtoon? {
    return null
}

fun main() {
    val subject1 = webtoon?.subject ?: "default"
    val subject2 = webtoon?.subject ?: return
    val subject3 = webtoon?.subject ?: throw Exception("Elvis Exception")
}
```

### 오류 throw 하기
* 만약 개발자가 null 이 되리라 예상하지 못했고, 실제로는 null 이 나와서 아무런 동작을 안했다면 이는 개발자가 오류를 찾기 어렵게 만든다.
* 이럴때는 강제로 오류를 발생시켜주는 것이 좋다.
* throw, requireNotNull, checkNotNull 등을 활용하면 좋다.

```kotlin
data class Webtoon(
  val id: Int? = null,
  val subject: String
)

fun example2(webtoon: Webtoon) {
  requireNotNull(webtoon.id)
  checkNotNull(webtoon.id)

  val id = webtoon.id ?: throw IllegalArgumentException("illegal argument exception")
}
```

### not-null assertion(!!) 과 관련된 문제
* nullable 를 처리하는 가장 간단한 방법은 not-null assertion(!!)을 사용하는 것 이다.
* 하지만, 어떤 대상이 null 이 아니라고 생각하고 다루면, NPE 가 발생할 확률이 있다.
* !! 는 사용하기는 쉽지만, 좋은 해결 방법은 아니다.
* !! 타입은 nullable 이지만, null 이 나오지 않는다는 것이 거의 확실한 상황에서 많이 사용된다. 하지만, 현재 확실하다고, 미래에 확실한 것은 아니다.

* 책에서는 컬렉션이 null 일 수 있다는고 말하는데, 잘못된 정보인 것 같다.
* 아래 코드를 보면 example3() 호출 시 NPE 가 발생한다고 나와있는데, 실제로는 NoSuchElementException 이 발생한다.

```kotlin
User
fun example3(vararg nums: Int): Int {
    return nums.max()!!
}

fun main() {
   example3()
}
```

* 일반적으로 !! 사용을 피해야 한다.
* !! 연산자를 보면 반드시 조심하고, 무언가 잘못되어 있을 가능성을 생각해야한다.

### 의미 있는 nullability 피하기
* nullability 는 어떻게든 적절하게 처리해야하므로, 추가 비용이 발생한다.
* 따라서, 필요한 경우가 아니라면, nullablility 자체를 피하는 것이 좋다.
* 중요한 메세지를 전달하는게 아니라면 null 을 사용하지 않는 것이 좋다. 만약 이유 없이 사용하면 !! 연산자를 사용하게 되고, 의미 없이 코드를 더럽히는 예외 처리를 해야할 것 이다.

#### nullablility 를 사용할 때 피하는 방법
* 클래스에서 nullablility 에 따라 여러 함수를 만들어서 제공 (ex. ```List<T>.get()```, ```List<T>.getOrNull()```)
* lateinit 프로퍼티와 notNull Delegate 사용 
* 빈 컬렉션 대신 null 을 리턴하지 않기
* nullable enum 과 None enum 은 다른 의미이다.

### lateinit 프로퍼티와 notNull Delegate
* 클래스가 클래스 생성 중에 초기화할 수 없는 프로퍼티를 가지는 것은 드문 일이 아니지만 분명 존재한다.
* 프로퍼티를 사용할 때마다 nullable 에서 null 이 아닌 것으로 타입 변환하는 것은 바람직하지 않다.
* 이러한 코드에 대한 바람직한 해결책은 나중에 속성을 초기화할 수 있는, lateinit 한정자를 사용하는 것이다.

* Entity 를 만들어서 DB 에 저장할 때 @CreatedDate 어노테이션을 사용하면 저장 시점에 해당 데이터가 생긴다.
* 따라서, 생성자를 만들 때 미리 초기화하지 않고, lateinit var 로 설정하면 조금 더 유연한 코드가 된다.

#### kotlin + Spring Data JPA 를 사용했을 때 createDate, updateDate 를 lateinit var 로 설정
```kotlin
@EntityListeners(AuditingEntityListener::class)
@MappedSuperclass
abstract class BaseEntity {
  @Id
  @GeneratedValue(strategy = GenerationType.IDENTITY)
  val id: Long = 0L

  @CreatedDate
  @Column(name = "created_at", nullable = false, updatable = false)
  lateinit var createdAt: LocalDateTime

  @LastModifiedDate
  @Column(name = "modified_at")
  lateinit var modifiedAt: LocalDateTime

  @CreatedBy
  @Column(name = "created_by", nullable = false, updatable = false)
  var createdBy: Long? = null

  @LastModifiedBy
  @Column(name = "modified_by")
  var modifiedBy: Long? = null
}
```

* lateinit var 를 만약 초기화 전에 값을 사용하려고 하면 예외가 발생한다. -> 즉, 처음 사용하기 전에 반드시 초기화가 되어 있을 경우에만 lateinit 을 붙여야한다.
* 만약 그런 값이 사용되어 예외가 발생하면, 그 사실을 알아야 하므로 예외가 발생하는 것은 오히려 좋은 일이다.
* lateinit var vs nullable
  * !! 연산자로 언팩하지 않아도 된다.
  * 이후에 어떤 의미를 나타내기 위해서 null 을 사용하고 싶을 때, nullable 로도 만들 수 있다.
  * 프로퍼티가 초기화된 이후에는 초기화되지 않은 상태로 돌아갈 수 없다.
* JVM 에서 Int, Long, Double, Boolean 과 같은 기본 타입과 연결된 타입으로 프로퍼티를 초기화해야 하는 경우에는 lateinit var 를 사용할 수 없다. -> lateinit 보단 느리지만, Delegates.notNull 을 사용하면 된다.
