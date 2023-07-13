💻 **좋은 코드 - 안정성**
=============



## 📖 가변성을 제한하라

코틀린을 구성하는 모듈 중 일부는 **상태(state)**를 가진다.

요소가 상태를 가지는 경우, 사용 방법 뿐 아니라 그 **이력(history)**에 대해서도 의존하게 되는데

이 때 상태를 적절이 관리하는 것은 꽤나 어려운 일이다.

> * 프로그램을 이해하고 디버그하기 힘들어진다. **(복잡성)**
>
> * 가변성이 있으면(mutability), 코드의 실행을 추론하기 어려워진다. **(일관성)**
>
> * **멀티스레드 프로그램**의 경우 적절한 동기화를 필요로 한다. **(변경이 일어나는 모든 부분에서 충돌이 발생할 수 있음)**
>
>   ###### 코틀린의 코루틴을 활용하면, 충돌 관련 문제가 줄어들긴 하지만 문제가 사라지는 건 아니다.
>
>   ###### 적절하게 동기화를 구현하는 것은 매우 어려운 일 -> **변할 수 있는 지점을 줄이는게 좋다.**
>
> * 테스트하기 어렵다.
>
> * 상태 변경시, 해당 변경을 다른 부분에 알려야 하는 경우가 있다.

이처럼 가변성이 주는 단점이 생각보다 많아서 이를 완전하게 제한하는 프로그래밍 언어인 **순수 함수형 언어**를 사용하기도 한다. (ex) Haskell)

하지만 가변성에 너무 많은 제한이 걸리기 때문에 개발 분야에서 주류로 사용되지는 않는다.

그럼 코틀린에서는 어떤식으로 가변성을 제한할 수 있을까?



### ✓ 코틀린에서 가변성 제한하기 

코틀린은 **가변성을 제한할 수 있게 설계**되어 있다. 따라서 불변 객체를 만들거나, 프로퍼티를 변경할 수 없게 막는 것이 굉장히 수월하다.

이를 위한 많은 방법 중 많이 사용되고 중요한 것들을 정리하면 다음과 같다.

> * ##### 읽기 전용 프로퍼티 (val)
>
> * ##### 가변 컬렉션과 읽기 전용 컬렉션 구분하기
>
> * ##### 데이터 클래스의 copy

각각의 내용을 살펴 보자.

> * #### **읽기 전용 프로퍼티 (val)**
>
>   코틀린은 value 값처럼 동작하는 읽기 전용 프로퍼티 val을 만들 수 있다. (읽고 쓸때는 var)
>
>   하지만 완전 변경이 불가능 한 것은 아닌데, 
>
>    - 읽기 전용 프로퍼티가 mutable 객체를 담고 있는 경우 - 내부적으로 변할 수 있다.
>
>      ```kotlin
>      val list = mutableListOf(1, 2, 3)
>      list.add(4)
>      
>      print(list) //[1, 2, 3, 4]
>      ```
>
>      
>
>    - 사용자 정의 게터로 var 프로퍼티를 사용하는 경우 - var 프로퍼티가 변할 때 변할 수 있다.
>
>      ```kotlin
>      var name: String = "Marcin"
>      var surname: String = "Moskala"
>      var fullname
>      	get() = "$name $surname"
>      
>      fun main(){
>        println(fullName)	//Marcin Moskala
>        name = "Maja"
>        println(fullName)	//Maja Moskala
>      }
>      ```
>
>      
>
>   이처럼 읽기 전용 프로퍼티 val의 값은 변경될 수 있긴 하지만, 프로퍼티 레퍼런스 자체를 변경할 수는 없으므로 **동기화 문제를 줄일 수 있다.**
>
>   이 때 중요한 것은 **val은 읽기 전용 프로퍼티지만, 불변을 의미하는 것은 아니다.**

> * #### **가변 컬렉션과 읽기 전용 컬렉션 구분하기**
>
>   코틑린의 컬렉션은 **읽고 쓸 수 있는 컬렉션**과 **읽기 전용 컬렉션**으로 구분된다.
>
>   - ###### 읽고 쓸 수 있는 컬렉션 - MutableIterable / MutableCollection / MutableSet / MutableList
>
>   - ###### 읽기 전용 컬렉션 - Iterable / Collection / Set / List
>
>   <img src="./image/코틑린 컬렉션 인터페이스 계층.png" width="800" height="300">
>
>   다음은 Iterable<T>.map의 구현을 단순하게 나타낸 것이다.
>
>   ```kotlin
>   inline fun <T, R> Iterable<T>.map(
>   	transformation: (T) -> R
>   ): List<R> {
>     val list = ArrayList<R>()
>     for(elem in this) {
>       list.add(transformation(elmem))
>     }
>     return list
>   }
>   ```
>
>   Iterable<T>.map은 읽기 전용 컬렉션의 일부지만, 
>
>   이러한 컬렉션을 진짜로 **불변(immutable)하게 만들지 않고, 읽기 전용으로 설계한 것**은 굉장히 중요한 일이다.
>
>   하지만 개발자가 컬렉션 **다운 캐스팅을 할 때 문제가 생기는데,** 
>
>   이것은 읽기 전용을 읽기 전용으로만 사용해야 하는 규약을 위반하고 추상화를 무시하는 행위가 된다. 
>
>   이런 코드는 안전하지 않고, 예측하지 못한 결과를 초래한다. 예를 들어,
>
>   ```kotlin
>   val list = listOf(1,2,3)
>   
>   //이렇게 하면 안됨 진짜
>   if (list is MutableList) {
>     list.add(4)
>   }
>   ```
>
>   이 코드는 java.lang.UnsupportegOperationException 예외를 뱉게 된다.
>
>   이와 같은 문제가 생기지 않게 하기 위해 다운캐스팅이 아니라 
>
>   **복제(copy)**를 통해서 새로운 mutable 컬렉션을 만드는 **list.toMutableList**를 활용해야 한다.
>
>   ```kotlin
>   val list = listOf(1,2,3)
>   
>   val mutableList = list.toMutableList()
>   mutableList.add(4)
>   ```
>
>   이렇게 작성할 경우 규약도 어기지 않고, 기존의 객체도 불변이므로 안전하다고 할 수 있다.

> * #### **데이터 클래스의 copy**
>
>   String이나 Int처럼 **불변 객체**를 많이 사용하는 데는 이유가 있다.
>
>   불변 객체를 사용할 경우, 다음과 같은 장점이 있다.
>
>   > * 한 번 정의된 상태가 유지되므로, 코드를 이해하기 쉽다.
>   > * 불변 객체는 공유했을 때도 충돌이 따로 이루어지지 않으므로, 병렬 처리를 안전하게 할 수 있다.
>   > * 불변 객체에 대한 참조는 변경되지 않으므로, 쉽게 캐시할 수 있다.
>   > * 불변 객체는 방어적 복사본(defensive copy)을 만들 필요가 없다.
>   > * 불변 객체는 다른 객체(가변, 불변)를 만들 때 활용하기 좋다. 또한 불변 객체는 실행을 더 쉽게 예측 가능하다.
>   > * 불변 객체는 Set / Map의 키로 사용할 수 있다. (가변 객체는 활용이 불가능 함)

가변 객체는 예측하기 어려우며 위험하다는 단점이 있지만, 불변 객체는 변경할 수 없다는 단점이 있다.

따라서 **불변 객체는 자신의 일부를 수정한 새로운 객체를 만들어내는 메서드를 가져야 한다**.

예를 들어 Int 자체는 불변이지만, 내부적으로 pluss / minus 메서드로 자신을 수정한 새로운 Int를 리턴할 수 있다.

Iterable의 경우도 읽기 전용이지만, map / filter 메서드로 자신을 수정한 새로운 Iterable 객체를 만들어서 리턴한다.

따라서, 위의 예시처럼 **우리가 직접 만드는 불변객체도 비슷한 형태로 작동해야 한다.**

```kotlin
class User(		//유저 클래스 - 성(suranme)을 변경해야 한다면?
  val name: String,
  val surname: String
) {
  fun withSurname(surname: String) = User(name, surname)	//자신의 일부를 수정한 새로운 객체를 만들어내는 메서드
}

var user = User("Maja", "Markiewicz")
user = user.withSurname("Moskala")
print(user)	//User(name=Maja, surname=Moskala)
```

하지만 모든 프로퍼티를 대상으로 이런 함수를 하나하나 만드는 건 너무나 귀찮은 일이다.

이럴 경우에는 data 한정자를 사용하면 된다.

```kotlin
data class User(
  val name: String,
  val surname: String
)

var user = User("Maja", "Markiewicz")
user = user.copy(surname = "Moskala")		//copy 메서드를 활용해 새로운 객체 생성
print(user)		//User(name=Maja, surname=Moskala)
```

코틀린에서는 이처럼 불변 특성을 가지는 **데이터 모델 클래**스를 만든다.

가변 객체가 변경 측면에서 유리할지는 몰라도 **데이터 모델 클래스를 만들어 불변 객체로 만드는 것이 더 많은 장점을 가지므로**,

기본적으로는 이렇게 만드는 게 좋다.



### ✓ 다른 종류의 변경 가능 지점

변경할 수 있는 리스트를 만들어야 한다고 해보자.

하나는 **mutable 컬렉션 (가변 컬렉션)**을 만드는 것이고, 다른 하나는 **var로 읽고 쓸 수 있는 프로퍼티 (가변 프로퍼티)**를 만드는 방법이 있을 것이다.

```kotlin
val list1: MutableList<Int> = mutableListOf()
var list2: List<Int> = listOf()
```

두 가지의 방법 모두 변경이 가능하지만 방법이 다르다.

```kotlin
list1.add(1)
list2 = list2 + 1
```

두 코드 모두 += 연산자를 가지고 변경할 수 있지만, 내부적인 처리는 다르다.

```kotlin
list1 += 1		//list1.plusAssign(1)로 변경됨
list2 += 1		//list2.plus(1)로 변경됨
```

두 가지 모두 정상 작동하지만, 장단점이 존재한다.

> 첫 번째 코드(가변 컬렉션)는 **구체적인 리스트 구현 내부에 변경 가능 지점**이 있다.
>
> 멀티 스레드 처리가 이루어질 경우, 내부적으로 적절한 동기화가 되어 있는지 확실하게 알 수 없어 위험하다.

> 두 번째 코드(가변 프로퍼티)는 **프로퍼티 자체가 변경 가능 지점**이다.
>
> 따라서 멀티스레드 처리의 안정성이 더 좋다고 할 수 있다.

이처럼 멀티스레드 안정성 측면에서 **가변 프로퍼티**를 쓰는게 좋지만,

이외에도 사용자 정의 세터를 활용해서 변경을 추적하는 것도 가능하다.

```kotlin
var names by Delgates.observable(listOf<String>()) { _, old, new ->
	println("Names가 $old에서 $new로 변합니다.")
}

names += "Fabio"
//Names가 []에서 [Fabio]로 변합니다.
names += "Bill"
//Names가 [Fabio]에서 [Fabio, Bill]로 변합니다.
```

가변 컬렉션도 이처럼 관찰할 수 있도록 만들려면 추가적인 구현이 필요하다. -> 귀찮고 어렵다.

따라서 **가변 프로퍼티에 읽기 전용 컬렉션을 넣어 사용**하는 것이 쉽다. (+private으로 만들 수도 있음)

추가적으로, 최악의 방식은 프로퍼티와 컬렉션을 모두 변경 가능한 지점으로 만드는 것이다. 

```kotlin
var list3 = mutableListOf<Int>()		//이렇겐 하지 말자...진짜 머리 아픈 코드가 될 수 있음
```

이럴 경우 변경될 수 있는 두 지점에 모두에 대한 동기화를 구현해야 함. (문제 + 1)

모호성이 발생해서 +=를 사용할 수 없게 됨. (문제 + 1)

따라서 상태를 변경할 수 있는 불필요한 방법은 최대한 배제하고 가변성을 제한하는 것이 필요하다.



### ✓ 변경 가능 지점 노출하지 말기

상태를 나타내는 가변 객체를 외부에 노출하는 것은 굉장히 위험한 일이다.

예를 들어 다음 코드를 보자.

```kotlin
data class User(val name: String)

class UserRepository {
	private val storedUsers: MutableMap<Int, String> =
		mutableMapOf()
		
		fun loadAll(): MutableMap<Int, String> {
			return storeUsrs
		}
		
		//...
}
```

loadAll 메서드를 사용해서 private 상태인 UserRepository를 수정할 수 있다.

```kotlin
val userRepository = UserRepository()

val storedUsers = userRepository.loadAll()
storedUsers[4] = "Kirill"
//...

print(userRepository.loadAll())		//{4=Kirill}
```

이러한 코드는 돌발적인 수정시에 위험할 수 있는데, 이를 처리하는 방법으로는 **방어적 복제(defensive copying)**가 있다.

###### **방어적 복제란**? 리턴되는 가변 객체를 복제하는 것

```kotlin
class UserHolder {
	private val user: MutableUser()
	
	fun get(): MutableUser {
		return user.copy()
	}
	
	//...
}
```

#### 계속 봤던 것처럼 **가능하다면 무조건 가변성을 제한하는 것이 좋다.** (결론 땅땅땅)



### ✓ 챕터 정리

* #### var보다는 val을 사용하는 것이 좋다.

* #### 가변 프로퍼티보다는 **불변 프로퍼티를 사용**하는 것이 좋다.

* #### 가변 객체와 클래스 보다는 **불변 객체와 클래스를 사용**하는 것이 좋다.

* #### 변경이 필요한 대상을 만들어야 한다면, **불변 데이터 클래스로 만들고 copy를 활용**하는 것이 좋다.

* #### 컬렉션에 상태를 저장해야 한다면, 가변 컬렉션보다는 **읽기 전용 컬렉션을 사용**하는 것이 좋다.

* #### 변이 지점을 적절하게 설계하고, **불필요한 변이 지점은 만들지 않는 것**이 좋다.

* #### 가**변 객체를 외부에 노출하지 않는 것**이 좋다.

끝.

------



## 📖 변수의 스코프를 최소화하라

상태를 정의할 때는 변수와 프로퍼티의 스코프를 최소화하는 것이 좋다.

* **프로퍼티보다는 지역 변수**를 사용하는 것이 좋다.
* **최대한 좁은 스코프**를 갖게 변수를 사용한다. (예를 들어, 반복문 내부에서만 변수가 사용된다면, 변수를 반복문 내부에 작성하는 것이 좋음)

코틀린의 **스코프는 기본적으로 중괄호**로 만들어지며, **내부 스코프에서 외부 스코프에 있는 요소에만 접근할 수 있다**. (밖에서 안으로는 접근을 못한단 소리)

```kotlin
val a = 1
fun fizz() {
	val b = 2
	print(a + b)
}
val buzz = {
	val c = 3
	print(a + c)
}
//이 위치에서는 a를 사용할 수 있지만, b와 c는 사용할 수 없음 (기본이지 기본)
```

다음은 변수 스코프를 제한하는 예시이다.

```kotlin
//나쁜 예
var user: User
for (i in users.indices) {
	user = users[i]
	print("User at $i is $user")
}

//좋은 예
for (i in users.indices) {
	val user = users[i]
	print("User at $i is $user")
}

//제일 좋은 예
for ((i, user) in users.withIndex()) {
	print("User at $i is $user")
}
```

스코프를 좁게 만드는 것이 좋은 이유는 굉장히 많지만, 제일 중요한 이유는 **프로그램을 추적하고 관리하기 쉽기 때문**이다.

요소가 많아져서 변경될 수 있는 부분이 많아지면, 프로그램을 이해하기 어려워진다. 

애플리케이션이 간단할수록 읽기도 쉽고 안전하다. (가변 프로퍼티보다 불변 프로퍼티를 선호하는 이유와 일맥상통함)

가변 프로퍼티는 좁은 스코프에 걸쳐 있을수록, 변경을 추적하는 것이 쉽고, 이렇게 추적이 되어야 코드를 이해하고 변경하는 것도 쉬워진다.



변수는 읽기 전용 또는 읽고 쓰기 전용 여부와 상관 없이, **변수를 정의할 때 초기화 되는 것이 좋다.**

If, when, try-catch, Elvis 표현식 등을 활용하면, 최대한 변수를 정의할 때 초기화할 수 있다.

```kotlin
//나쁜 예
val user: User
if (hasValue) {
	user = getValue()
} else {
	user = User()
}

//좋은 예
val user: User = if(hasValue) {
	getValue()
} else {
	User()
}
```

여러 프로퍼티를 한꺼번에 설정해야 하는 경우에는 **구조분해 선언(destructuring declaration)**을 활용하는 것이 좋다.

```kotlin
//나쁜 예
fun updateWeather(degrees: Int) {
	val description: String
	val color: Int
	//if문 안에서 초기화되는 중
	if(...){
		//어쩌구
	}
}

//좋은 예
fun updateWeather(degrees: Int) {
	//선언과 동시에 초기화 하는 중 (when을 활용)
	val (description, color) = when {
		//저쩌구
	}
}
```

여하튼 변수의 스코프가 넓으면 굉장히 위험하다. 좀 더 자세히 살펴보자.



### ✓ 캡처링

예를 들어 에라토스테네스의 체를 구현하는 문제를 보자.

```kotlin
val primes: Sequence<Int> = sequence {
	var numbers = generateSequence(2) { it + 1 }
	
	var prime: Int
	while (true) {
		prime = numbers.first()
		yield(prime)
		numbers = numbers.drop(1)
			.filter { it % prime != 0 }
	}
}
```

코드 상으로는 문제가 없어보이지만 실행 결과가 이상하게 나온다.

이유는 prime이라는 변수를 캡처했기 때문이다.

반복문 내부에서 filter를 활용하여 prime으로 나눌 수 있는 숫자를 필터링한다.

하지만 시퀀스를 활용하고 있기 때문에 필터링이 지연되고, 최종적인 prime 값으로만 필터링이 된다. 

prime이 2로 설정되어 있을 때 필터링된 4를 제외하면, drop만 동작하므로 그냥 연속된 숫자가 나와버리게 된 것이다.

따라서 항상 잠재적인 **캡쳐 문제를 주의**해야 한다. 가변성을 피하고 스코프 범위를 좁게 만들면, 이런 문제를 간단히 피할 수 있다.



### ✓ 챕터 정리

또 똑같은 얘기 또 하지만, 여러 가지 이유로 변**수의 스코프는 좁게 만들어서 활용하는 것**이 좋다.

또한 **var 보다는 val을 사용하는 것**이 좋다. **람다에서 변수를 캡쳐한다는 것**을 꼭 기억하고 쓰자.

간단한 규칙만 지켜 주면, 발생할 수 있는 여러 문제를 차단할 수 있을 것이다.

끝.

------



## 📖 최대한 플랫폼 타입을 사용하지 말라

코틀린의 등장과 함께 소개된 널 안정성(null-safety)은 코틀린의 주요 사기 기능 중 하나이다.

하지만 널 안정성 메커니즘이 없는 다른 언어들을 코틀린과 연결해서 사용할 때는 예외가 발생할 수 있다.

만약 자바에서 String 타입을 리턴하는 메서드가 있다고 생각할 때 코틀린에서는 어떻게 사용할까?

**@Nullable** 어노테이션이 붙어 있다면, 이것을 nullable로 추정하고, **String?**으로 변경하면 될 일이다.

**@NotNull** 어노테이션이 붙어 있다면, **String**으로 변경하면 그만이다.

근데 만약 아무것도 안붙어 있으면 어떻게 해야할까?

최대한 안전하게 접근한다면 nullable로 가정하고 다루어야 한다.

하지만 어떤 메서드는 null을 리턴하지 않을 것이 확실할 수도 있다. 이런 경우 not-null 단정(!!)을 사용한다.

이러한 nullable 문제와 관련하여 자주 문제되는 부분이 자바의 제네릭 타입이다.

```java
//자바 코드
public class UserRepo {
	public List<User> getUsers() {
		//...
	}
}
```

```kotlin
val users: List<User> = UserRepo().users!!.filterNotNull()
```

다음과 같이 자바 API에서 List<User>을 리턴하고 아무 어노테이션이 붙지 않는다고 할 경우를 생각해보자.

우리는 이것을 사용할 때 리스트와 리스트 내부의 User 객체들이 널이 아니라는 사실을 알아야 한다.

더 나아가 List<List<User>>를 리턴하다고 생각하면 벌써 머리가 아프다.

```kotlin
val users: List<List<User>> = UserRepo().groupedUsers!!.map { it!!.filterNotNull() }
```

리스트는 적어도 map과 filterNotNull 등의 메소드를 제공하지만, 다른 제네릭 타입이라고 생각하면 좀 많이 끔찍해진다.

그래서 코틀린은 다른 언어에서 넘어온 타입들을 특수하게 **플랫폼 타입(platform type)**으로서 다룬다.

플랫폼 타입은 String! 처럼 타입 이름 뒤에 ! 기호를 붙여서 표기한다. (물론 노테이션이 직접적으로 코드에 나타나진 않음)

따라서 다음 코드와 같은 형태로 선택적으로 사용한다.

```java
//자바 코드
public class UserRepo {
	public User getUser() {
		//...
	}
}
```

```kotlin
val repo = UserRepo()
val user1 = repo.user					//user1의 타입은 User!
val user2: User = repo.user		//user2의 타입은 User
val user3: User? = repo.user	//user3의 타입은 User?
```

이렇게 사용하게 되면 위의 문제가 사라지게 된다.

```kotlin
val users: List<User> = UserRepo().users
val users: List<List<User>> = UserRepo().groupedUsers
```

하지만 여전히 null이 아니라고 생각되는 것이 null일 가능성이 있기해 항상 조심하고 주의를 기울여야 한다.

설계자가 명시적으로 표시해 두지 않는 이상 언제든지 동작이 변경될 가능성이 있다.

따라서 왠만하면 **자바와 코틀린을 함께 사용할 때는 @Nullable 또는 @NotNull 어노테이션을 항상 붙여서 사용하는 것이 권장 된다.**

안전하지 않은 플랫폼 타입을 왜 최대한 빠르게 제거해야 하는지 다음 코드에서 살펴보자.

```java
//자바 코드
public class JavaClass {
	public String getValue() {
		return null;
	}
}
```

```kotlin
fun statedType() {
	val value: String = JavaClass().value
	//...
	println(value.length)
}

fun platformType() {
	val value = JavaClass().value
	//...
	println(value.length)
}
```

두 가지 타입 모두 NPE가 발생한다. 일반적으로 개발자는 getValue가 null을 리턴할 것이라고 가정하지 않으므로, 실수했다고 생각할 것이다.

하지만 이 두 코드는 오류의 발생 위치에서 많은 차이가 있다.

> **statedType**의 경우 자바에서 값을 가져오는 위치에서 NPE가 발생한다.
>
> 이 위치의 오류의 경우 null이 아니라고 예상을 했지만 null이 나온다는 것을 굉장히 쉽게 알 수 있다. 따라서 코드를 쉽게 수정할 수 있다.

> **platformType**은 값을 활용할 때 NPE가 발생한다.
>
> 실제로는 저런 간단한 표현식보다 더 복잡한 표현식을 사용할 때 이러한 오류가 발생할 것이다.
>
> 플랫폼 타입으로 지정된 변수는 nullable일 수도 있고, 아닐 수도 있다. (+ 여러 가지 가능성이 존재함)
>
> 따라서 **오류를 찾는 데 굉장히 오랜 시간이 걸리게 될 것**이다.

또한 **플랫폼 타입이 전파(다른 곳에서 사용)되는 일은 굉장히 위험**하다.

항상 위험을 내포하고 있으므로, (또 얘기하지만) **안전한 코드를 원한다면 이런 부분은 제거하는 것이 좋다.**

인텔리제이 IDEA에서는 이러한 플랫폼 타입에 사용에 대해 경고를 출력해주기도 한다.



### ✓ 챕터 정리

다른 프로그래밍 언어로부터 와서 nullable 여부를 알 수 없는 타입을 **플랫폼 타입(platform type)**이라고 한다.

이러한 플랫폼 타입은 해당 부분만이 아니라 이를 **활용하는 곳까지 영향을 줄 수 있는 위험한 코드**이다.

따라서 이러한 코드를 사용하고 있다면 **해당 코드에서 제거하는 것이 권장**되며, 

연결되어 있는 필드에 **nullable 여부를 지정하는 어노테이션을 활용**하는 것도 좋다.

끝.

------



## 📖 inferred 타입으로 리턴하지 말라

코틀린의 **타입 추론(type inference)**은 JVM 세계에서 가장 널리 알려진 코틀린의 특징이다.

다만 타입 추론을 사용할 때는 몇 가지 위험한 부분들이 있는데, 이러한 위험을 피하려면

할당 때 **inferred 타입은 오른쪽에 있는 피연산자에 맞게 설정**된다는 사실을 기억해야 한다. (절대로 슈퍼클래스 또는 인터페이스로는 설정되지 않음)

```kotlin
open class Animal
class Zebra: Animal()

fun main() {
	var animal = Zebra()
	animal = Animal()		//오류: Type mismatch
}
```

일반적인 경우엔 이러한 것이 문제가 되지 않는다. 타입을 그냥 명시적으로 지정하면 될 일이다.

```kotlin
open class Animal
class Zebra: Animal()

fun main() {
	var animal: Animal = Zebra()
	animal = Animal()		//오류: Type mismatch
}
```

하지만 직접적으로 라이브러리(또는 모듈)를 조작할 수 없는 경우에는 이러한 문제를 간단하게 해결하긴 힘들다.

그리고 이러한 경우에 inferred 타입을 노출한다면, 위험한 일이 발생할 수 있다.

```kotlin
interface CarFactory {
	fun produce(): Car
}
```

다른 것을 지정하지 않을 경우 디폴트로 생성되는 자동차가 있다고 할 때,

```kotlin
val DEFAULT_CAR: Car = Fiat126P()
```

대부분의 공장에서 Fiat126P 라는 자동차를 생산하므로, 이를 디폴트로 두고 DEFAULT_CAR는 Car로 명시적으로 지정되어 있으므로

따로 필요없다고 판단하여, 함수의 리턴 타입을 제거했다고 가정 하자.

```kotlin
interface CarFactory {
	fun produce() = DEFAULT_CAR
}
```

그런데 이후에 다른 사람이 코드를 보다가, DEFAULT_CAR는 타입 추론에 의해 자동으로 타입이 지정될 것이므로, 

Car를 명시적으로 지정하지 않아도 된다고 생각해서, 다음과 같이 코드를 변경했다고 해 보자.

```kotlin
val DEFAULT_CAR = Fiat126P()
```

이제 이렇게 되면 CarFactory는 Fiat126P 이외의 자동차를 생산하지 못하는 문제가 발생한다.

이처럼 인터페이스를 사용했다면 수월하게 오류를 찾을 수 있을 것이지만, 외부 API를 사용했다면 꽤나 머리가 아픈 오류가 될 것이다.

리턴 타입은 API를 잘 모르는 사람에게 전달해 줄 수 있는 중요한 정보이다.

따라서 **리턴 타입은 외부에서 확인할 수 있도록 명시적으로 지정**해 주는 것이 좋다.



### ✓ 챕터 정리

**타입을 확실하게 지정해야 하는 경우에는 명시적으로 타입을 지정해야 한다는 원칙**만 가지면 된다. 

이는 굉장히 중요한 정보이므로 숨기지 않는 것이 좋다.

또한 안전을 위해서 **외부 API를 만들 때는 반드시 타입을 지정**하고, 이렇게 지정한 타입을 특별한 이유와 확실한 확인없이 제거해서는 안된다.

inferred 타입은 프로젝트가 진전될 때, 제한이 너무 많아지거나 예측 못한 결과를 낼 수 있다는 사실을 명심해야 한다.

끝.

------



## 📖 예외를 활용해 코드에 제한을 걸어라

확실하게 어떤 형태로 동작해야 하는 코드가 있다면, **예외를 활용해 제한을 걸어주는 게 좋다.**

코틀린에서는 코드의 동작에 제한을 걸 때 다음과 같은 방법을 사용할 수 있다.

> * **require 블록** : 아규먼트를 제한할 수 있다.
> * **check 블록** : 상태와 관련된 동작을 제한할 수 있다.
> * **assert 블록** : 어떤 것이 true인지 확인할 수 있다. (Assert 블록은 테스트 모드에서만 작동함)
> * **return 또는 throw와 함께 활용하는 Elvis 연산자**

다음은 이러한 메커니즘을 사용하는 간단한 예이다.

```kotlin
//Stack<T>의 일부
fun pop(num: Int = 1): List<T> {
	require(num <= size) {
		"Cannot remove more elements than current size"
	}
	check(isOpen) { "Cannot pop from closed stack" }
	val ret = collection.take(num)
	collection = collection.drop(num)
	assert(ret.size == num)
	return ret
}
```

이런 식으로 제한을 걸어 주면 다양한 장점이 발생한다.

* ###### 제한을 걸면 문서를 읽지 않은 개발자도 문제를 확인할 수 있다.

* ###### 문제가 있을 경우에 함수가 예상하지 못한 동작을 하지 않고 예외를 throw 한다. (문제를 놓치지 않고 코드가 더 안정적으로 작동)

* ###### 코드가 어느정도 자체적으로 검사된다.

* ###### 스마트 캐스트 기능을 활용할 수 있게 되므로, 캐스트(타입 변환)를 적게 할 수 있다.

이러한 제한들과 관련된 내용을 좀 더 자세히 살펴보자. 먼저 가장 많이 사용하는 아규먼트와 관련된 내용부터 보자.



### ✓ 아규먼트

함수를 정의할 때 타입 시스템을 활용해서 **아규먼트(Argument)에 제한을 거는 코드**를 많이 사용하고는 한다.

* 숫자를 아규먼트로 받아서 팩토리얼을 계산한다면 숫자는 양의 정수여야 한다.
* 좌표들을 아규먼트로 받아서 클러스터를 찾을 때는 비어 있지 않은 좌표 목록이 필요하다.
* 사용자로부터 메일 주소를 입력받을 때는 값이 입력되어 있는지, 그리고 메일 형식이 올바른지 확인해야 한다.

```kotlin
fun factorial(n: Int): Long {
	require(n >= 0)
	return if (n <= 1) else factorial(n - 1) * n
}

fun findClusters(points: List<Point>): List<Cluster> {
	require(points.isNotEmpty())
	//...
}

fun sendEmail(user: User, message: String) {
	requireNotNull(user.email)
	require(isValidEmail(user.email))
	//...
}
```

일반적으로 이러한 제한들을 걸 때는 **require 함수를 사용**한다. require 함수는 제한을 확인하고, 제한을 만족하지 못하는 경우 예외를 throw 한다.

이와 같은 형태의 입력 유효성 검사 코드는 **함수의 가장 앞부분에 배치**되므로, 읽는 사람도 쉽게 확인할 수 있다.

또한 람다를 활용해서 지연 메시지를 정의할 수도 있다.

```kotlin
fun factorial(n: Int): Long {
	require(n >= 0) { "Cannot calculate factorial of $n " + "because it is smaller than 0" }
	return if(n <= 1) 1 else factorial(n - 1) * n
}
```

이처럼 **require 함수는 아규먼트와 관련된 제한을 걸 때 사용**할 수 있다.

이외에도 예외를 활용해 제한을 거는 대표적인 대상으로 상태가 있다.



### ✓ 상태

어떤 구체적인 조건을 만족할 때만 함수를 사용할 수 있게 해야 할 때가 있다.

* 어떤 객체가 미리 초기화되어 있어야만 처리를 하게 하고 싶은 함수
* 사용자가 로그인했을 때만 처리를 하게 하고 싶은 함수
* 객체를 사용할 수 있는 시점에 사용하고 싶은 함수

```kotlin
fun speak(text: String) {
	check(isInitialized)
	//...
}

fun getUserInfo(): UserInfo {
	checkNotNull(token)
	//...
}

fun next(): T {
	check(isOpen)
	//...
}
```

**상태와 관련된 제한을 걸 때는 일반적으로 check 함수**를 사용한다.

얼핏 require 함수와 비슷하고, 상태가 올바른지 확인할 때 사용한다.

예외 메시지는 require과 마찬가지로 지연 메시지를 사용해서 변경할 수 있다.

**함수 전체에 대한 어떤 예측이 있을 때는 일반적으로 require 블록 뒤에 배치**한다.



### ✓ Assert 계열 함수 사용

함수가 올바르게 구현되었다면, 확실하게 참을 낼 수 있는 코드들이 있다.

여러 구현 문제로 발생할 수 있는 추가적인 문제를 예방하려면, 단위 테스트를 사용하는 것이 좋다.

```kotlin
class StackTest {
	@Test
	fun 'Stack pops correct number of elements'() {
		val stack = Stack(20) { it }
		val ret = stack.pop(10)
		assertEquals(10, ret.size)
	}
	
	//...
}
```

단위 테스트는 구현의 정확성을 확인하는 가장 기본적인 방법이다. 

위의 테스트는 스택이 10개의 요소를 팝하면, 10개의 요소가 나온다는 보편적인 사실을 테스트하고 있다.

하지만 위의 한 가지 경우만 테스트해서 모든 상황에서 괜찮을지는 알 수 없다.

따라서 모든 pop 호출 위치에서 제대로 동작하는지 확인해도 좋다.

```kotlin
fun pop(num: Int = 1): List<T> {
	//...
	assert(ret.size == num)
	return ret
}
```

이러한 조건은 현재 코틀리니/JVM에서만 활성화되며, -ea JVM 옵션을 활성화해야 확인할 수 있다.

하지만 이러한 코드는 테스트시에만 활성화되므로, 오류가 발생해도 사용자가 알아차릴 수는 없다.

만약 정말 이 코드가 심각한 오류고, 심각한 결과를 초래할 수 있는 경우에는 check를 사용하는 것이 좋다.

단위 테스트에서 assert를 사용하면 다음과 같은 장점이 있다.

* Assert 계열의 함수는 코드를 자체 점검하며, 더 효율적으로 테스트할 수 있게 해 준다.
* 특정 상황이 아닌 모든 상황에 대한 테스트를 할 수 있다.
* 실행 시점에 정확하게 어떻게 되는지 확인할 수 있다.
* 실제 코드가 더 빠른 시점에 실패하게 만듭니다. (예상하지 못한 동작이 언제 어디서 실행되었는지 빠르게 찾기 가능)



### ✓ nullability와 스마트 캐스팅

코틀린에서 require과 check 블록으로 어떤 조건을 확인해서 true가 나왔다면, 해당 조건은 이후로도 true일거라 가정한다. 

따라서 이를 활용해서 타입 비교를 했다면, **스마트 캐스트가 작동**한다.

```kotlin
fun changeDress(person: Person) {
	require(person.outfit is Dress)
	val dress: Dress = person.outfit		//require에서 확인했으므로 자동 캐스트
}
```

이러한 특징은 어떤 대상이 null인지 확인할 때 굉장히 유용하다.

```kotlin
class Person(val email: String?) 

fun sendEmail(person: Person, message: String) {
	require(person.email != null)
	val email: String = person.email		//null이 아니므로 String이 됨 (?가 안붙음)
  //...
}
```

추가로 requireNotNull / checkNotNull 이라는 특수한 함수도 스마트 캐스트를 지원한다.

nullability를 목적으로, **오른쪽에 throw나 return을 두고 Elvis 연산자를 활용**하는 경우가 많다.

```kotlin
fun sendEmail(person: Person, text: String) {
	val email: String = person.email ?: return
	//...
}
```

이처럼 오른쪽에 return을 넣으면 오류를 발생시키지 않고 단순히 함수를 중지시킬 수도 있다.

프로퍼티에 문제가 있어서 null일 때 여러 처리를 해야 할 때도, return/throw와 run 함수를 조합해서 활용하면 된다.

```kotlin
fun sendEmail(person: Person, text: String) {
	val email: String = person.email ?: run {
		log("Email not sent, no email address")
		return
	}
  //...
}
```

이처럼 return과 throw를 활용한 Elvis 연산자는 **nullable을 확인**할 때 굉장히 많이 사용되는 관용적인 방법이다.

또한 이러한 함수는 **함수의 앞부분에 넣어서 잘 보이게 만드는 것**이 좋다.



### ✓ 챕터 정리

이번 챕터에서 활용한 내용을 기반으로, 다음과 같은 이득을 얻었다.

* #### 제한을 훨씬 더 쉽게 확인할 수 있다.

* #### 애플리케이션을 더 안정적으로 지킬 수 있다.

* #### 코드를 잘못 쓰는 상황을 막을 수 있다. 

* #### 스마트 캐스팅을 활용할 수 있다.

이를 활용했던 메커니즘을 정리하면 다음과 같다.

* #### **require 블록** : 아규먼트와 관련된 예측을 정의할 때 사용하는 범용적인 방법

* #### check 블록 : 상태와 관련된 예측을 정의할 때 사용하는 범용적인 방법

* #### **assert 블록** : 테스트 모드에서 테스트를 할 때 사용하는 범용적인 방버

* #### **return과 throw와 함께 Elvis 연산자 사용하기**

끝.

------



## 📖 사용자 정의 오류보다는 표준 오류를 사용하라

require / check / assert 함수를 사용하면, 대부분의 코틀린 오류를 처리할 수 있다.

하지만 이외에도 예측하지 못한 상황을 나타내야 하는 경우가 있다.

예를 들어 JSON 형식을 파싱하는 라이브러리를 구현한다고 해볼 때 기본적으로 입력된 JSON 파일의 형식에 문제가 있다면,

JSONParsingException 등을 발생시키는 것이 좋을 것이다.

하지만 표준 라이브러리에는 이를 나타내는 적절한 오류가 없으므로, 사용자 정의 오류를 사용한다.

하지만 **가능하다면 최대한 표준 라이브러리의 오류를 사용하는 것**이 좋다.

**표준 라이브러리의 오류는 많은 개발자가 알고 있으므로**, 이를 재사용하는 것이 좋다.

잘 만들어진 규약을 가진 널리 알려진 요소를 재사용한다면, **다른 사람들이  API를 더 쉽게 배우고 이해할 수 있다.**

일반적으로 사용되는 예외를 몇 가지 정리해 보면 다음과 같다.

* IllegalArgumentException과 IllegalStateException
* IndexOutOfBoundsException
* ConcurrentModificationException
* UnsupportedOperationException
* NoSuchElementException

해당 예외들의 자세한 정보는 구글링 해보자.

끝.

------



## 📖 결과 부족이 발생할 경우  null과 Failure를 사용하라

함수가 원하는 결과를 만들어 낼 수 없을 때가 있다.

* 서버로부터 데이터를 읽어 들이려고 했는데, 인터넷 연결 문제로 읽어 들이지 못한 경우
* 조건에 맞는 첫 번째 요소를 찾으려 했는데, 조건에 맞는 요소가 없는 경우
* 텍스트를 파싱해서 객체를 만들려고 했는데, 텍스트의 형식이 맞지 않는 경우

이러한 상황을 처리하는 메커니즘은 크게 다음과 같이 두 가지가 있다.

1. null 또는 '실패를 나타내는  sealed 클래스(일반적으로 Failure라는 이름을 붙인다)'를 리턴한다.
2. 예외를 throw 한다.

이 두 가지는 중요한 차이점이 있는데, 일단 예외는 정보를 전달하는 방법으로 사용해서는 안된다.

##### 예외는 잘못된 특별한 상황을 나타내야 하며, 처리되어야 한다. 예외는 예외적인 상황이 발생했을 때 사용하는 것이 좋다.

이유는 다음과 같다.

* 많은 개발자가 예외가 전파되는 과정을 제대로 추적하지 못한다.
* 코틀린의 모든 예외는 unchecked 예외이다. 따라서 사용자가 예외처리를 하지 않을 수도 있고, 이와 관련된 내용은 문서에도 제대로 드러나지 않는다.
* 예외는 예외적인 상황을 처리하기 위해서 만들어졌으므로 명시적인 테스트(explicit test)만큼 빠르게 동작하지 않는다.
* try-catch 블록 내부에 코드를 배치하면, 컴파일러가 할 수 있는 최적화가 제한된다.

반면 첫 번째로 얘기했던  **null과 Failure는 예상되는 오류를 표현할 때 굉장히 좋다.**

이는 명시적이고 효율적이며, 간단한 방법으로 처리할 수 있다.

따라서 충분히 예측할 수 있는 범위의 오류는 null과 Failure를 사용하고, 예측하기 어려운 예외적인 범위의 오류는 예외를  throw해서 처리하는 것이 좋다.

```kotlin
inline fun <reified T> String.readObjectOrNull(): T? {
	//...
	if(incorrectSign) {
		return null
	}
	//...
	return result
}

inline fun <reified T> String.readObject(): Result<T> {
	//...
	if(incorrectSign) {
		return Failure(JsonParsingException())
	}
	//...
	return Success(result)
}

sealed class Result<out T>
class Success<out T>(val result: T): Result<T>()
class Failure(val throwable: Throwable): Result<Nothing>()

class JsonParsingException: Exception()
```

이렇게 표시되는 오류는 다루기 쉬우며 놓치기 어렵다.

null을 처리해야 한다면, 사용자는 안전 호출(safe call) 또는  Elvis 연산자 같은 다양한 널 안정성(null-safety) 기능을 활용한다.

```kotlin
val age = userText.readObjectOrNull<Person>()?.age ?: -1
```

Result와 같은 공용체(union type)를 리턴하기로 했다면, when 표현식을 사용해서 이를 처리할 수 있다.

```kotlin
val person = userText.readObjectOrNull<Person>()
val age = when(person) {
	is Success -> person.age
	is Failure -> -1
}
```

이러한 오류 처리 방식은 try-catch 블록보다 효율적이며, 사용하기 쉽고 더 명확하다.

**null 값과 sealed result 클래스는 명시적으로 처리할 수 있으며, 애플리케이션의 흐름을 중지하지도 않는다.**

null과 sealed result 클래스의 차이점은,

​	추가적인 정보 전달의 경우 -> sealed result

​	그게 아니면 -> null

이런 방식으로 사용하는 것이 일반적이다.

일반적으로 두 가지 형태의 함수를 사용한다. List를 기반으로 살펴보자.

* **ge**t: 특정 위치에 있는 요소를 추출할 때 사용한다. 만약 요소가 해당 위치에 없다면  IndexOutOfBoundsException을 발생시킨다.
* **getOrNull**: out of range 오류가 발생할 수 있는 경우에 사용하며, 발생한 경우에는  null을 리턴한다.

다른 선택지로 getOrDefault라는 선택지도 있지만 **일반적으로 getOrNull 또는  Elvis 연산자(?:)를 사용**하는 것이 쉽다.

개발자는 항상 자신이 요소를 안전하게 추출할 거라 생각한다. 따라서 nullable을 리턴하면 안된다. 

개발자에게 null이 발생할 수 있다는 경고를 주려면 , getOrNull 등을 사용해서 **무엇이 리턴되는지 예측할 수 있게 해야한다.**

끝.

------



## 📖 적절하게 null을 처리하라

**null은 '값이 부족하다(lack of value)'는 것을 나타낸다.** 프로퍼티가 null이라는 것은 값이 설정되지 않았거나, 제거되었다는 것을 의미한다.

함수가 null을 리턴한다는것은 함수에 따라서 여러 의미를 가질 수 있다.

* String.toIntOrNull()은 String을 Int로 적절하게 변환할 수 없을 경우 null을 리턴한다.
* Iterable<T>.firstOfNull(() -> Boolean)은 주어진 조건에 맞는 요소가 없을 경우 null을 리턴한다.

이처럼 **null은 최대한 명확한 의미를 갖는 것이 좋다**. 이는 nullable 값을 처리해야 하기 때문인데,

이를 처리하는 사람은 API 사용자(API 요소를 사용하는개발자)이다.

```kotlin
val printer: Printer? = getPrinter()
printer.print()	//컴파일 오류

printer?.print()	//안전 호출
if (printer != null) printer.print()	//스마트 캐스팅
printer!!.print()	//not-null assertion
```

기본적으로 nullable 타입은 세 가지 방법으로 처리한다.

* ?. / 스마트 캐스팅 / Elvis 연산자 등을 활용해서 **안전하게 처리**한다. 
* **오류를 throw** 한다.
* 함수 또는 프로퍼티를 **리팩터링해서 nullable 타입이 나오지 않게** 바꾼다.



### ✓ null을 안전하게 처리하기

null을 안전하게 처리하는 방법 중 널리 사용되는 방법으로는 **안전 호출(safety call)**과 **스마트 캐스팅(smart casting)**이 있다.

```kotlin
print?.print()		//안전 호출
if (printer != null)	printer.print() 		//스마트 캐스팅
```

두 가지 모두 printer가 null이 아닐때 print 함수를 호출한다.

사용자 관점에서도 안전하며, 개발자에게도 정말 편리하여, nullable 값을 처리할 때는 이 방법을 가장 많이 활용한다.

이외에도 코틀린은 nullable 변수와 관련된 처리를 굉장히 광범위하게 지원한다. 대표적으로는 **Elvis 연산자**를 사용하는 것이다.

```kotlin
val printerName1 = printer?.name ?: "Unnamed"
val printerName2 = printer?.name ?: return
val printerName3 = printer?.name ?: throw Error("Printer must be named")
```

이처럼 다양한 객체가 nullable과 관련된 처리를 지원한다.

스마트 캐스팅은 **코틀린의 규약 기능(contracts feature)**을 지원한다. 이 기능을 사용하면 다음과 같이 스마트 캐스팅할 수 있다.

```kotlin
println("What is your name?")
val name = readLine()
if (!name.isNullOrBlank()) {
	println("Hello ${name.toUpperCase()}")
}

val news: List<News>? = getNews()
if(!news.isNullOrEmpty()) {
	news.forEach { notifyUser(it) }
}
```

###  

### ⌲ 추가 개념

> * #### 방어적 프로그래밍이란?
>
>   방어적 프로그래밍은 코드가 프로덕션 환경으로 들어갔을 때 발생할 수 있는 수많은 것들로부터 프로그램을 방어해서 안정성을 높이는 방법을 나타내는 굉장히 포괄적인 용어이다. 상황을 처리할 수 있는 올바른 방법이 있을 때는 굉장히 좋다.

> * #### 공격적 프로그래밍이란?
>
>   언제나 모든 상황을 안전하게 처리하는 것은 불가능하다. 예상치 못한 상황이 발생했을 때, 이러한 문제를 개발자에게 알려서 수정하게 만드는 것을 공격적 프로그래밍이라고 한다. 이전에 보았던 **require / check / assert** 가 바로 이러한 공격적 프로그래밍을 위한 도구이다.

공격적 / 방어적이라는 이름 때문에 두 개념이 충돌되는 것처럼 보이지만, 사실은 **코드의 안전을 위해 모두 필요한 프로그래밍 기법**이다.



### ✓ 오류 throw 하기

이전 코드는 printer가 null일 때, 개발자에게 이를 알리지 않고 코드가 그대로 진행된다. 

하지만 printer가 null이 되리라 예상하지 못했다면, print 메서드가 호출되지 않아서 이상할 것이다. 

이는 개발자가 오류를 찾기 어렵게 만든다.

따라서 다른 개발자 입장에서 당연하게 생각하게 되는 부분에서 문제가 발생할 경우에는 **개발자에게 오류를 강제로 발생시켜 주는게 좋다**.

이 때 throw / !! / requiredNotNull / checkNotNull 등을 활용한다.

```kotlin
fun process(user: User) {
	requiredNotNull(user.name)
	val context = checkNotNull(context)
	val networdService = 
		getNetworkService(context) ?:
		throw NoInternetConnection()
	networkService.getData { data, userData ->
		show(data!!, userData!!)
	}
}
```



### ✓ not-null assertion(!!)과 관련된 문제

nullable을 처리하는 가장 간단한 방법은 not-null assertion(!!)을 사용하는 것이다.

그런데 !!를 사용하면 자바에서 nullable을 처리할 때 발생할 수 있는 문제가 똑같이 발생한다.

어떤 대상이 null이 아니라고 생각하고 다루면, NPE 예외가 발생한다. 

!!은 사용하기 쉽지만, 좋은 해결 방법은 아니다.

예외가 발생할 때, 어떤 설명도 없는 **제네릭 예외(generic exception)**이 발생하게 된다.

또한 코드가 짧고 너무 사용하기 쉽다 보니 남용하게 되는 문제도 생긴다.

!!는 nullable이지만, null이 나오지 않는다는 것이 거의 확실한 상황에서 많이 사용된다.

하지만 현재가 확실하다고 미래도 확실한 것은 아니다.

**Nullability(널일 수 있는지)와 관련된 정보는 숨겨져 있으므로, 굉장히 쉽게 놓칠 수 있다.** 

따라서 변수를 null로 설정하고, 이후에 !! 연산자를 사용하는 방법은 좋은 방법이 아니다.

이런 상황에 코드를 작성하는 올바른 방법은 **lateinit 또는 Delegates.notNull을 사용**하는 것이다.

!! 연산자가 의미있는 경우는 매우 드물기 때문에 일반적으로 n**ullability 가 제대로 표현되지 않는 라이브러리를 사용할 때 정도에만 사용**해야 한다.

**일반적으로 !! 연산자 사용을 무조건 피해야 한다.**



### ✓ 의미 없는 nullability 피하기

nullability는 어떻게든 적절히 처리해야 하므로, 추가 비용이 발생한다.

따라서 굳이 필요한 경우가 아니라면, **nullability 자체를 피하는 것**이 좋다.

따라서 다른 개발자가 보기에 의미가 없을 때는 **null을 사용하지 않는 것**이 좋다.

만약 무지성 null을 사용했다면, 다른 개발자들이 코드를 작성할 때, 위험한 !! 연산자 사용을 하게 되고, 의미 없이 코드를 더럽히는 예외처리가 필요하게 될 수 있다.

* 클래스에서 nullability에 따라 여러 함수를 만들어서 제공할 수도 있다.
* 어떤 값이 클래스 생성 이후에 확실하게 설정된다는 보장이 있다면, **lateinit 프로퍼티와 notNull 델리게이트**를 사용해라.
* 빈 컬렉션 대신  null을 리턴하지 마라. (요소가 부족하다는 것을 나타내려면, 빈 컬렉션을 사용)
* nullable enum과 None enum 값은 완전히 다른 의미이다. (nullable enum -> 별도 처리 필요 / None enum -> 정의에 없음, 필요할 경우에 추가)



### ✓ lateinit 프로퍼티와 notNull 델리게이트

클래스가 클래스 생성 중에 초기화할 수 없는 프로퍼티를 가지는 것은 충분히 있을 수 있는 일이다.

이러한 프로퍼티는 사용전에 반드시 초기화해서 사용해야 한다.

예를 들어 JUnit의 @BeforeEach처럼 **다른 함수들보다도 먼저 호출되는 함수에서 프로퍼티가 설정되는 경우**가 있다.

```kotlin
class UserControllerTest {
	private var dao: UserDto? = null
	private var controller: UserController? = null
	
	@BeforeEach
	fun init() {
		dao = mockk()
		controller = UserController(dao!!)
	}
	
	@Test
	fun Test() {
		controller!!.doSomething()
	}
}
```

프로퍼티를 사용할 때마다 nullable에서 null이 아닌 것으로 타입 변환하는 것은 바람직하지 않다.

이런 값은 테스트 전에 설정될 것이라는 것이 명확하므로, 의미 없는 코드가 사용된다고 할 수 있다.

따라서 이러한 코드에 대한 해결책은 나중에 속성을 초기화하는 **lateinit 한정자**를 사용하는 것이다.

```kotlin
class UserControllerTest {
	private lateinit var dao: UserDto
	private lateinit var controller: UserController
	
	@BeforeEach
	fun init() {
		dao = mockk()
		controller = UserController(dao!!)
	}
	
	@Test
	fun Test() {
		controller!!.doSomething()
	}
}
```

물론 이러한 경우에도 초기화 전에 값을 사용하려고 하면 예외가 발생한다.

이렇게 얘기하면 사용하는데 쫄아서 사용을 두려워할 수 있지만, 걱정할 필요 없다.

무조건 사**용하기 전에 반드시 초기화가 되어 있을 경우에만 lateinit을 붙이는 것**이다.

lateinit은 nullable과 비교해서 다음과 같은 차이가 있다.

* !! 연산자로 언팩(unpack)하지 않아도 된다.
* 이후에 어떤 의미를 나타내기 위해서 null을 사용하고 싶을 때, nullable로 만들 수도 있다.
* 프로퍼티가 초기화된 이후에는 초기화되지 않은 상태로 돌아갈 수 없다.

**lateinit은 프로퍼티를 처음 사용하기 전에 반드시 초기화될 거라고 예상되는 상황에 활용한다.**

이러한 상황으로는 **명확한 순서가 있는 라이프 사이클(lifecycle)을 갖는 경우**에 해당한다.

예를 들어 안드로이드 Activity의 onCreate / IOS UIVewController의 viewDidAppear / 리액트 React.Component의 componentDidMount 등이 대표적이다.

반대로 lateinit을 사용할 수 없는 경우도 있다.

JVM에서 Int / Long / Double / Boolean 과 같은 **기본 타입과 연결된 타입으로 프로퍼티를 초기화하는 경우**이다.

이때는 lateinit보다는 약간 느리지만, **Delegates.notNull을 사용**한다.

```kotlin
class DoctorActivity: Activity() {
	private var doctorId: Int by Delegates.notNull()
	private var fromNotification: Boolean by Delegate.notNull()
	
	override fun onCreate(savedInstanceState: Bundle?) {
		super.onCreate(savedInstanceState)
		doctorId = intent.extras.getInt(DOCTOR_ID_ARG)
		fromNotification = intent.extras.getBoolean(FROM_NOTIFICATION_ARG)
	}
}
```

위의 코드처럼 onCreate 때 초기화하는 프로퍼티는 지연 초기화하는 형태로 다음과 같이 **프로퍼티 위임(property delegation)**을 사용할 수도 있다.

```kotlin
class DoctorActivity: Activity() {
	private var doctorId: Int by arg(DOCTOR_ID_ARG)
	private var fromNotification: Boolean by arg(FROM_NOTIFICATION_ARG)
}
```

**프로퍼티 위임을 사용하면, nullability로 발생하는 여러 문제들을 안전하게 처리할 수 있다.**

끝.

------



## 📖 use를 사용하여 리소스를 닫아라

더 이상 필요하지 않을 때, close 메서드를 사용해서 명시적으로 닫아야 하는 리소스가 있다.

코틀린/JVM에서 사용하는 자바 표준 라이브러리에는 이런 리소스들이 굉장히 많다. 예시로

* #### InputStream과 OutputStream

* #### java.sql.connection

* #### java.io.Reader(FileReader / BufferedReader / CSSParser)

* #### java.new.Socket과 java.util.Scanner

등이 있다.

이러한 리소스들은 **AutoCloseable을 상속받는 Closeable 인터페이스를 구현하고 있다.**

이러한 모든 리소스들은 최종적으로 리소스에 대한 레퍼런스가 없어질 때, 가비지 컬렉터가 처리한다.

하지만 굉장히 느리며 유지 비용이 많이 들어간다.

따라서 필요 없다면 **명시적으로 close 메서드를 호출**해주는 것이 좋다.

전통적으로는 try-finally 블록을 사용해서 처리했지만, 굉장히 복잡하고 좋지않다.

또한 이러한 예외를 따로 처리하지 않기 때문에 리소스를 닫을 때 예외가 발생할 수도 있다.

**Use 함수를 사용해서 모든 Closeable 객체에 사용할 수 있다.**

```kotlin
fun countCharactersInFile(path: String): Int {
	val reader = BufferedReader(FileReader(path))
	reader.use {
		return reader.lineSequence().sumBy { it.length }
	}
}
```

**람다 매개변수로 리시버가 전달되는 형태**도 있으므로, 다음과 같이 줄일 수도 있다.

```kotlin
fun countCharactersInFile(path: String): Int {
	BufferedReader(FileReader(path)).use { reader -> return reader.lineSequence().sumBy { it.length }}
}
```

파일을 리소스로 사용하는 경우가 많고, 한 줄씩 읽어 들이는 경우도 많으므로, 코틀린 표준 라이브러리는 **useLines 함수도 제공**한다.

```kotlin
fun countCharactersInFile(path: String): Int {
	File(path).useLines { lines -> return lines.sumBy { it.length }}
}
```

이렇게 처리하면 메모리에 파일의 내용을 한 줄씩만 유지하므로, 대용량 파일도 적절하게 처리할 수 있다.

다만 파일의 줄을 한 번만 사용할 수 있다는 단점이 있다.



### ✓ 챕터 정리

use를 사용하면 Closeable/AutoCloseable를 구현한 객체를 쉽고 안전하게 처리할 수 있다.

또한 파일을 처리할 때는 파일을 한 줄씩 읽어 들이는 useLines를 사용하는 것이 좋습니다.

끝.

------



## 📖 단위 테스트를 만들어라

사실 코드를 안전하게 만드는 가장 궁극적인 방법은 다양한 종류의 테스트를 하는 것이다.

사용자 관점에서 애플리케이션 외부적으로 제대로 작동하는지 확인하는 것이 목표인 테스트는 개발자에게 유용하지만 충분하지는 않을 수 있다.

이러한 문제를 해결하기 위해 **단위 테스트(unit test)**가 필요하다.

단위 테스트는 개발자가 작성하며, 개발자에게 유용하다.

단위 테스트는 일반적으로 다음과 같은 내용을 확인한다.

* **일반적인 유스케이스**(이를 happy path라고 표현한다): 요소가 사용될  것이라고 생각하는 일반적인 방법을 테스트한다.
* 일**반적인 오류케이스와 잠재적인 문제**: 제대로 동작하지 않을 거라고 예상되는 일반적인 부분, 과거에 문제가 발생했던 부분 등을 테스트 한다.
* **에지 케이스와 잘못된 아규먼트**: Int의 경우 Int.MAX_VALUE를 사용하는 경우, nullable의 경우 'null' 또는 'null 값으로 채워진 객체'를 사용하는 경우를 의미한다.

단위 테스트는 개발자가 만들고 있는 요소가 제대로 동작하는지를 빠르게 피드백 해주므로 개발하는 동안에 큰 도움이 된다.

테스트는 **계속해서 축적**되므로, **회귀 테스트도 쉽다**. 또한 **수동으로 테스트하기 어려운 것들도 확인**할 수 있다.

TDD라는 접근 방식도 있다. 개발전에 테스트를 작성하고 테스트를 통과시키는 것을 목적으로 하나하나 구현해 나가는 방식이다.

> * #### 단위 테스트의 장단점
>
>   - ##### 장점
>
>     + **테스트가 잘 된 요소는 신뢰할 수 있다.** (요소를 활용한 작업에 자신감이 생김)
>     + **테스트가 잘 만들어져 있다면, 리팩토링 하는 것이 두렵지 않다.** (리팩토링 했을 때 버그가 생기는지 쉽게 확인할 수 있음)
>     + **수동으로 테스트하는 것보다 훨씬 빠르다.** (개발의 전체적인 속도가 빨라지고 재밌어짐)
>
>   - ##### 단점
>
>     + 단위 테스트를 만드는 데 시간이 걸린다. (**장기적으로는 효율적**이나 단기적으로는 시간이 걸리는 편임)
>     + 테스트를 활용할 수 있도록 코드를 조정해야 한다. (**오히려 훌륭하고 잘 정립된 아키텍쳐를 사용하는 것이 강제되어 이득**임)
>     + **좋은 단위 테스트를 만드는 작업이 꽤나 어렵다**. (잘못 만들어진 단위 테스트는 득보다 실이 더 클 수 있음)

효과적인 단위 테스트를 하는 방법을 습득하고, 단위 테스트를 위한 코드를 작성하는 것이 생각보다 쉽지 않다.

숙련된 코틀린 개발자가 되려면, 단위 테스트와 관련된 기술을 습득하고, 중요한 코드라고 할 수 있는 다음과 같은 부분에 대해 단위 테스트하는 방법을 알고 있어야 한다.

* 복잡한 부분
* 계속해서 수정이 일어나고 리팩토링이 일어날 수 있는 부분
* 비즈니스 로직 부분
* 공용 API 부분
* 문제가 자주 발생하는 부분
* 수정해야 하는 프로덕션 버그



### ✓ 챕터 정리

#### 테스트는 애플리케이션이 진짜로 올바르게 동작하는지 확인하는 아주 중요한 작업이다.

#### 그 중에서도 개발 과정에서 가장 효율적으로 활용할 수 있는 **단위 테스트**를 꼭 기억하자.

끝.

------

* Effective Kotlin - 마르친 모스칼라 지음
