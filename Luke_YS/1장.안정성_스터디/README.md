# 🔎 **좋은 코드 - 안정성** (스터디 내용 정리)



## 📖 변경 가능 지점

**가변성을 제한하라 - '다른 종류의 변경 가능 지점'** 파트에서는 변경 가능 지점을 설명하기 위해 다음과 같은 예제 코드가 등장했다.

```kotlin
val list1: MutableList<Int> = mutableListOf()
var list2: List<Int> = listOf()
```

두 코드는 다음과 같이 변경이 가능하다.

```kotlin
list1.add(1)
list2 = list2 + 1
```

본문에서는 두 가지 코드 모두 다음과 같이 += 연산자를 활용해 변경할 수 있다고 말한다.

```kotlin
list1 += 1	//list1.plusAssign(1)
list2 += 1	//list2 = list2.plus(1)
```

이 때 두 코드의 특징을 이야기하는데, 

첫 번째 코드는 **구체적인 리스트 구현 내부에 변경 가능 지점**이 있다는 것,

두 번째 코드는 **프로퍼티 자체가 변경 가능 지점**이라는 것이다.

이게 무슨 말일까? 코드를 분석해보자.

```kotlin
val list1: MutableList<Int> = mutableListOf() 	//프로퍼티는 불변, 컬렉션은 가변
var list2: List<Int> = listOf()									//프로퍼티는 가변, 컬렉션은 불변
```

만약 값을 추가하는 로직이 동작한다고 할 때 

첫 번째 코드의 경우엔 프로퍼티는 불변이기 때문에 프로퍼티의 수정은 불가하고, 프로퍼티 내부에 정의되어 있는 **가변 컬렉션에 값을 추가하는 방향으로 동작**한다.

두 번째 코드의 경우엔 프로퍼티는 가변, 내부 정의 컬렉션은 불변이기 때문에 내부에 있는 컬렉션을 수정할 수 없는 상황이므로, **가변 상태인 프로퍼티를 수정하는 방향으로 로직이 동작**한다.

이렇듯 이 두 코드는 **변경 가능 지점(mutating point)**에 차이를 가진다. 이게 왜 중요할까?

이는 두 번째 코드가 **멀티스레드 처리와 변경 추적에 유리**한 코드임을 뜻한다.



### ➤ 멀티스레드 처리에서의 유리함

멀티스레드 처리가 이루어질 경우 **각 스레드의 동기화**가 중요하게 생각된다.

이 때 내부 깊은 곳(여기서는 리스트 구현 내부)에 변경 가능 지점이 있는 코드라면 내부적인 스레드의 동기화가 잘 이루어지는지 불확실하다.

따라서 불확실한 구현 내부의 동기화보다는 두 번째 코드처럼 변경 가능 지점이 가까이 존재하는 첫 번째 코드가 더 안정성이 좋다라고 말할 수 있다.



### ➤ 변경 추적에서의 유리함

첫 번째 코드의 경우엔 **Delegates.observable**을 사용하면, 리스트에 변경사항이 생길 때 로그를 추적할 수 있다. (자세한 코드는 해당 주차 README 확인)

가변 컬렉션의 경우도 관찰할 수 있도록 하려면 추가적인 구현이 필요하다.

얼핏 보면 불변 프로퍼티에 가변 컬렉션을 사용하는 코드가 좀 더 변경 추적에 쉬운 과정을 가지고 있다고 보이지만,

실질적으로 **객체 변경 제어**를 생각했을 때 가변 프로퍼티를 사용하는 게 더 쉽다.

---



## 📖 방어적 복제(defensive copying)

방어적 복제란 무엇인가?

**생성자로부터 받은 가변 값들을 외부에서 변경하는 것을 막기 위해 복사본을 이용하는 것**을 뜻한다.

컬렉션에서 완벽한 방어적 복사를 위해 수행되어야 하는 3가지 요소가 있다.

- 일급 컬렉션 객체 **내부로 들어오는 컬렉션에 대한 방어적 복사** 수행하기
- 컬렉션의 내부 객체가 가변 객체라면 **컬렉션 내부 객체에 대한 방어적 복사** 수행하기
- 일급 컬렉션 **객체 외부로 나가는 컬렉션에 대한 방어적 복사** 수행하기

>###### ※ 일급 컬렉션(First Class Collection) - 컬 렉션을 래핑하면서, 그 외 다른 멤버 변수가 없는 상태를 의미함
>
>```java
>public class Distibutionkey {
>	private Map<Int, String> keys;
>	public Distributionkey(Map<Int, String> keys) {
>		this.keys = keys;
>	}
>}
>```



### ➤ 컬렉션에 대한 방어적 복사

예를 들어, 어떤 일급 컬렉션을 정의하여 값을 넣어준다고 했을 때 다음과 같은 일이 일어난다.

```kotlin
//Key객체와 해당 객체를 리스트로 가지는 일급 컬렉션 Keys 
val keyList = mutableListOf(Key("A"))
val keys = Keys(keyList)		//Keys에 keyList의 주소값이 들어가버림

keyList.add(Key("B"))				//KeyList에 B를 추가하고 싶었지만 Keys와 keyList 양쪽에 A, B가 들어감
```

이처럼 **얕은 복사**로 인해 같은 메모리 주소를 참조하는 일이 일어난다. (진짜 모든 언어 수업에서 적어도 한 번씩은 항상 얘기하는 문제)

```kotlin
class Keys(keys: List<Key>) {
	val keys: List<Key> = keys.toList()
}
```

이 때 일급 객체 컬렉션의 생성자를 파라미터의 역할만 하도록 바꿔주고, value 값은 클래스 내부에서 새로운 컬렉션을 반환하는 형태로 바꾸어 주면 컬렉션에 대한 깊은 복사가 수행된다.



### ➤ 컬렉션에 내부 객체에 대한 방어적 복사

```kotlin
data class Car(private val name: String = "", var position: Int = 0) {
	fun move() {
		position++
	}
}

class Cars(cars: List<Car>) {
	val cars: List<Car> = cars.toList()
}

class Test {
	fun listCopyTest() {
		val carList = mutableListOf(Car("carA"))
		val cars = Cars(carList)	//carA의 postion 값이 1
		carList[0].move()					//carA의 postion 값이 1
	}
}
```

첫 번째 carList의 요소를 바꿨음에도 cars 값이 바뀌었다.

이 문제도 마찬가지로 자바나 코틀린에서 주로 많이 사용하는 data class의 copy() 메소드로 싶은 복사를 수행한다.
