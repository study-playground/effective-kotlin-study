### 일반적인 프로퍼티 패턴은 프로퍼티 위임을 만들어라

* 코틀린은 코드 재사용과 관련해서 프로퍼티 위임이라는 새로운 기능을 제공한다.
* 프로퍼티 위임을 사용하면 일반적인 프로퍼티의 행위를 추출해서 사용할 수 있다.
* 코틀린의 대표적인 stdlib 의 라이브러리 중 하나인 lazy 프로퍼티 -> ```val value by lazy { createValue }``` -> 처음 사용하는 요청이 들어올 때 초기화

* 프로퍼티 위임은 프로퍼티의 접근자(getter, setter) 구현을 다른 객체에게 맡길 수 있는 문법이다. 
* 즉, 다른 객체의 메서드를 활용해서 프로퍼티의 접근자(getter, setter)를 만들 수 있다. 
* 반복적으로 사용되는 프로퍼티 행위를 추출해서 재사용함으로써 코드를 간결하게 만들 수 있다. 
* 예를 들어서, Int 타입의 프로퍼티에 음수가 저장되는 것을 방지하는 Setter 를 프로젝트 내에서 반복적으로 사용하고 싶을 수 있다. 
* 이러한 요구사항을 가진 프로퍼티마다 Setter 를 일일이 정의하는 것은 너무 번거로울 수 있다. 
* 따라서 이러한 프로퍼티 행위를 추출하여 OnlyPositive 라는 객체를 만들자. 이때 메서드 이름이 중요하다. 
* Getter는 getValue, Setter는 setValue 함수를 사용해서 만들어야한다. 
* 이후 프로퍼티 선언문 뒤에 "by 객체"를 적으면 해당 객체가 프로퍼티의 Getter, Setter를 대리 구현하게 된다.
* 아래는 프로퍼티 위임의 예시이다.

```kotlin
class DelegateProperty<T>(var value: T) {
    operator fun getValue(thisRef: Any?, property: KProperty<*>): T {
        print("getValue")
        return value
    }

    operator fun setValue(thisRef: Any?, property: KProperty<*>, newValue: T) {
        print("setValue")
        value = newValue
    }
}
```

* by 디컴파일

```kotlin
var token: String? by DelegateProperty(null)

@JvmField
private val 'token$delegate' = DelegateProperty<String>(null)
var token: String?
	get() = 'token$delegate'.getValue(this, ::token)
	set(value) {
		'token$delegate'.setValue(this, ::token, value)
	}
```

* 코드에서 알 수 있는 것처럼 getValue 와 setValue 는 단순하게 값만 처리하게 바뀌는 것이 아니라, 컨텍스트(this)와 프로퍼티 레퍼런스의 경계도 같이 사용하는 형태로 바뀌게 된다.
* 프로퍼티에 대한 레퍼런스는 이름, 어노테이션과 관련된 정보등을 얻을 때 사용된다.
* 그리고 컨텍스트는 함수가 어떤 위치에서 사용하되는지와 관련된 정보를 제공해준다. 
* 이러한 정보로 인해서 getValue()와 setValue() 메서드가 여러개 있어도 컨텍스트를 활용함으로, 상황에 따라서 적절한 메서드가 선택된다.
* 객체를 프로퍼티에 위임하려면 val 는 getValue 연산, var 의 경우 getValue 와 setValue 연산이 필요하다.
* 이러한 연산은 지금까지 살펴본 것처럼 멤버 함수로도 만들 수 있지만, 확장함수로도 만들 수 있다.


