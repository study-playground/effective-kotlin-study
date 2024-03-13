### 데이터 집합 표현에 data 한정자를 사용하라

* 데이터를 한번에 전달해야 하는 상황에 일반적으로 data class 를 사용한다.
* data 한정자를 붙이면, 다음과 같은 함수가 자동으로 생성된다.
  * toString
  * equals / hashCode
  * copy
  * componentN

#### toString

* 클래스의 이름과 기본 생성자 형태로 모든 프로퍼티와 값을 출력해준다.
* ex. ```print(dataClass) DataClass(id=0, name=Hong)```

#### equals & hashCode
 
* equals 는 기본 생성자와 프로퍼티가 같은지 확인
* hashCode 는 서로 다른 객체의 value 가 같으면(equals 가 참이면), 같은 유니크값을 반환합니다.
* ex. ```dataClass == DataClass(0, "Hong")``` 

#### copy

* 기본 생성자 프로퍼티가 같은 새로운 객체를 복제한다.
* 새로 만들어진 객체의 값은 이름있는 아규먼트를 활용하여 변경할 수 있다.

```kotlin
val newObj = dataClass.copy(name = "YeongJun")
print(newObj) // DataClass(id=0, name=YeongJun)
```

#### componentN

* 위치를 기반으로 객체를 해제할 수 있게 해준다.

```kotlin
val (id, name) = dataClass
```

### 튜플 대신 데이터 클래스 사용하기

* 데이터 클래스는 튜플보다 많은 것을 제공한다.
* 코틀린의 튜플은 Serializable 을 기반으로 만들어지며, toString 을 사용할 수 있는 제네릭 데이터 클래스입니다.

```kotlin
public data class Pair<out A, out B> (
    public val first: A,
    public val second: B
) : Serializable {

    public override fun toString(): String =
        "($first, $second)"
}

public data class Triple<out A, out B, out C>(
    public val first: A,
    public val second: B,
    public val third: C
) : Serializable {

    public override fun toString(): String =
        "($first, $second, $third)"
}
```

* Pair 와 Triple 은 코틀린에 남아있는 마지막 튜플이다. -> 튜플은 데이터 클래스와 같은 역할을 하지만, 가독성이 나쁘고 데이터 클래스를 사용하는 것이 더 좋기 떄문에 점차 없어졌음

* 데이터 클래스를 활용하면 다음과 같은 점이 명확해진다.
  * 함수의 리턴 타입이 더 명확해지고, 리턴 타입이 더 짧아지며, 전달하기 쉬워진다. -> 데이터 클래스 자체를 리턴하기 때문 
  * 사용자가 데이터 클래스에 적혀 있는 것과 다른 이름을 활용해 변수를 해제하면 경고를 출력한다.
  * 좁은 스코프를 갖게 하고 싶으면, 가시성을 제한하면 된다. -> private