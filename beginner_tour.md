#### Variables
- Read only variables with `val`
- Mutable variables with `var`
```kotlin
val popcorn = 5
val hotdog = 7

var customers = 10
customers = 8
println(customers)
```
> We recommend declaring all variables as read-only (val) by default. Only use mutable variables (var) if you really need to. That way, you're less likely to accidentally change something that wasn't meant to change.

- `val` 을 사용하는걸 권장하는데... 초기값을 항상 필요로하면 DI 형태를 강조하는 언어인듯

#### String Templates
- Python 출력하는거랑 비슷함. (easy)

> A string value is a sequence of characters in double quotes ". Template expressions always start with a dollar sign $.

```kotlin
val customers = 10
println("There are $customers customers")
// There are 10 customers

println("There are ${customers + 1} customers")
// There are 11 customers
```
> To evaluate a piece of code in a template expression, place the code within curly braces {} after the dollar sign $.


#### Collection
| Collection type | Description |
|-----------------|-------------|
| Lists           | Ordered collections of items |
| Sets            | Unique unordered collections of items |
| Maps            | Sets of key-value pairs where keys are unique and map to only one value |


##### List
- read-only 라면 `listOf()`
- mutable 라면 `mutableListOf()`

```kotlin
val readOnlyShapes = listOf("triangle", "square", "circle")
println(readOnlyShapes)
// [triangle, square, circle]

val shapes : MutableList<String> = mutableListOf("triangle", "square", "circle")
println(shapes)
// [triangle, square, circle]
```

```kotlin
val readOnlyShapes = listOf("trianble", "square", "circle")
println("The first item in the list if : ${readOnlyShapes[0]}")
// The first item in the list is: triangle
```
인덱스 접근도 되고, first(), last() 접근도 된다. (java랑 비슷)

```kotlin
val readOnlyShapes = listOf("triangle", "square", "circle")
println("The first item in the list is: ${readOnlyShapes.first()}")
// The first item in the list is: triangle

println("This list has ${readOnlyShapes.count()} items")
// This list has 3 items

println("circle" in readOnlyShapes)
// true
```

- 리스트 원소를 바꾸려면 MutableList 사용!
```kotlin
val shapes: MutableList<String> = mutableListOf("triangle", "square", "circle")
// Add "pentagon" to the list
shapes.add("pentagon") 
println(shapes)  
// [triangle, square, circle, pentagon]

// Remove the first "pentagon" from the list
shapes.remove("pentagon") 
println(shapes)  
// [triangle, square, circle]
```

##### Set
- Unordered이고, Unique를 보장
- `setOf()` / `mutableSetOf()`로 사용하면 된다. 사용은 List랑 동일함

```kotlin
val readOnlyFruit = setOf("apple", "banana", "cherry", "cherry")
println(readOnlyFruit)
// [apple, banana, cherry]

println("This set has ${readOnlyFruit.count()} items")
// This set has 3 items

println("banana" in readOnlyFruit)
// true
```

- set에 추가하거나 삭제하고 싶으면 `mutableSetOf()` 사용
```kotlin
val fruit: MutableSet<String> = mutableSetOf("apple", "banana", "cherry", "cherry")
fruit.add("dragonfruit")    // Add "dragonfruit" to the set
println(fruit)              // [apple, banana, cherry, dragonfruit]

fruit.remove("dragonfruit") // Remove "dragonfruit" from the set
println(fruit)              // [apple, banana, cherry]
```

##### Map
- key-value를 저장함.
- mapOf() / mutableMapOf()

- 특이한건 `to`라고 표현한다.
```kotlin
// Read-only map
val readOnlyJuiceMenu = mapOf(
    "apple" to 100,
    "kiwi" to 190,
    "orange" to 100
)
println(readOnlyJuiceMenu)
// {apple=100, kiwi=190, orange=100}

// Mutable map with explicit type declaration
val juiceMenu: MutableMap<String, Int> = mutableMapOf(
    "apple" to 100,
    "kiwi" to 190,
    "orange" to 100
)
println(juiceMenu)
// {apple=100, kiwi=190, orange=100}

println("The value of apple juice is: ${readOnlyJuiceMenu["apple"]}")
// The value of apple juice is: 100
```
- 추가하거나 삭제할 때, mutableMapOf() 사용
```kotlin
val juiceMenu: MutableMap<String, Int> = mutableMapOf(
    "apple" to 100,
    "kiwi" to 190,
    "orange" to 100
)
juiceMenu["coconut"] = 150 // Add key "coconut" with value 150 to the map
println(juiceMenu)
// {apple=100, kiwi=190, orange=100, coconut=150}

val juiceMenu: MutableMap<String, Int> = mutableMapOf(
    "apple" to 100,
    "kiwi" to 190,
    "orange" to 100
)
juiceMenu.remove("orange")    // Remove key "orange" from the map
println(juiceMenu)
// {apple=100, kiwi=190}

println(readOnlyJuiceMenu.containsKey("kiwi"))
// true

println(readOnlyJuiceMenu.keys)
// [apple, kiwi, orange]

println(readOnlyJuiceMenu.values)
// [100, 190, 100]

println("orange" in readOnlyJuiceMenu.keys)
// true

// Alternatively, you don't need to use the keys property
println("orange" in readOnlyJuiceMenu)
// true

println(200 in readOnlyJuiceMenu.values)
// false
```


> If you try to access a key-value pair with a key that doesn't exist in a map, you see a null value:

```kotlin
// Read-only map
val readOnlyJuiceMenu = mapOf("apple" to 100, "kiwi" to 190, "orange" to 100)
println("The value of pineapple juice is: ${readOnlyJuiceMenu["pineapple"]}")
// The value of pineapple juice is: null
```

#### if 와 when
- 조건문 제어를 if와 when으로 제공함
```kotlin
val d : Int
val chck = true

if (check){
  d = 1
} else {
  d = 2
}

println(d) // 1
```
> Kotlin에는 condition ? then : else 형태의 삼항 연산자가 없다. 대신 if를 표현식(expression) 으로 사용할 수 있다. 각 분기에서 실행할 코드가 한 줄뿐이라면 중괄호 {}는 생략할 수 있다.
- 이건 좀 아쉽군
```kotlin
val a = 1
val b = 1
println(if (a > b) a else b)
```

when 사용 방법:
- 평가하려는 값을 소괄호 () 안에 넣는다.
- 각 분기(branch)는 중괄호 {} 안에 작성한다.
- 각 분기에서 조건과, 조건이 만족될 때 수행할 동작은 ->로 구분한다.
- 값을 반환하지 않고 동작만 수행한다.

값을 반환하지 않는게 특이함
```kotlin
val obj = "Hello"

when (obj){
  "1" -> println("one")
  "Hello" -> println("Greeting")
  else -> println("Unknown")
}

// Greeting

val trafficLightState = "Red" // This can be "Green", "Yellow", or "Red"

val trafficAction = when (trafficLightState) {
    "Green" -> "Go"
    "Yellow" -> "Slow down"
    "Red" -> "Stop"
    else -> "Malfunction"
}

println(trafficAction)  
// Stop
```

#### Range
- Kotlin에서 범위를 생성하는 가장 일반적인 방법은 .. 연산자를 사용하는 것이다.
  - 예를 들어 1..4는 1, 2, 3, 4와 같다.

- 끝 값을 포함하지 않는 범위를 선언하려면 ..< 연산자를 사용한다.
  - 예를 들어 1..<4는 1, 2, 3과 같다.

- 역순 범위를 선언하려면 downTo를 사용한다.
  - 예를 들어 4 downTo 1은 4, 3, 2, 1과 같다.

- 증가 값이 1이 아닌 범위를 선언하려면 step과 원하는 증가 값을 사용한다.
  - 예를 들어 1..5 step 2는 1, 3, 5와 같다.

Char 범위에서도 동일하게 사용할 수 있다:
- 'a'..'d'는 'a', 'b', 'c', 'd'와 같다.
- 'z' downTo 's' step 2는 'z', 'x', 'v', 't'와 같다.

```
for(number in 1..5){
  print(number)
}

val cakes = listOf("carrot", "cheese", "chocolate")
for(cake in cakes){
  println("Yummy, it's a $cake cake!")
}
// Yummy, it's a carrot cake!
// Yummy, it's a cheese cake!
// Yummy, it's a chocolate cake!
```

#### Functions
```kotlin
fun sum(x: Int, y: Int): Int{
  return x + y
}

fun main(){
  println(sum(1, 2)) // 3
}
```

- java의 void 반환형 함수는 마지막 `: Int`를 안하면 된다.
```kotlin
fun printMessage(message: String){
  println(message)
  // `return Unit` or `return` is optional
}

fun main(){
  printMessage("Hello")
  // Hello
}
```

```kotlin
val registeredUsernames = mutableListOf("john_doe", "jane_smith")
val registeredEmails = mutableListOf("john@example.com", "jane@example.com")

fun registerUser(username: String, email: String): String{
  if (username in registeredUsernames){
    return "Username already taken. Please choose a different username."
  }
  if (email in registeredEmails) {
      return "Email already registered. Please use a different email."
  }

  registeredUsernames.add(username)
  registeredEmails.add(email)

  return "User registered successfully: $username"
}
fun main() {
    println(registerUser("john_doe", "newjohn@example.com"))
    // Username already taken. Please choose a different username.
    println(registerUser("new_user", "newuser@example.com"))
    // User registered successfully: new_user
}
```

#### class
##### properties
인스턴스가 생성된 이후에 값이 변경될 필요가 없다면, 프로퍼티는 읽기 전용(val) 으로 선언하는 것을 권장한다.
- 소괄호 () 안에서 val이나 var 없이 프로퍼티를 선언할 수도 있지만, 이 경우 해당 값들은 인스턴스가 생성된 이후에는 외부에서 접근할 수 없다.
- 소괄호 () 안에 포함된 내용은 클래스 헤더(class header) 라고 한다.
```kotlin
class Contact(val id: Int, var email: String){
  val category: String = ""
}
```

##### instance 만들기
- 클래스에서 객체를 생성하려면 `생성자(constructor)` 를 사용해 클래스의 인스턴스를 선언한다.
- Kotlin에서는 기본적으로 클래스 헤더에 선언된 매개변수 를 사용해 자동으로 생성자(primary constructor)가 생성된다.
- Contact is a class.
- contact is an instance of the Contact class.
- id and email are properties.
- id and email are used with the default constructor to create contact.

```kotlin
class Contact(val id: Int, var email: String)

fun main() {
    val contact = Contact(1, "mary@gmail.com")
}
```

```kotlin
class Contact(val id: Int, var email: String)

fun main() {
    val contact = Contact(1, "mary@gmail.com")
    
    // Prints the value of the property: email
    println(contact.email)           
    // mary@gmail.com

    // Updates the value of the property: email
    contact.email = "jane@gmail.com"
    
    // Prints the new value of the property: email
    println(contact.email)           
    // jane@gmail.com
}
```

##### Member Function
```
class Contact(val id: Int, var email: String){
  fun printId(){
    println(id)
  }
}

fun main(){
  val contact = Contact(1, "mary@gmail.com")
  contact.printId()
}
```

##### Data classes
- Kotlin에는 데이터를 저장하는 데 특히 유용한 데이터 클래스(data class) 가 있다.
    - 데이터 클래스는 일반 클래스와 동일한 기능을 가지지만, 추가적인 멤버 함수들이 자동으로 제공 된다는 점이 특징이다.
- 이 멤버 함수들을 사용하면 인스턴스를 읽기 쉬운 형태로 출력 하거나, 클래스 인스턴스 간 비교, 인스턴스 복사 등의 작업을 쉽게 수행할 수 있다.
- 이러한 함수들이 자동으로 제공되기 때문에, 각 클래스마다 동일한 보일러플레이트 코드 를 직접 작성할 필요가 없다.
```kotlin
data class User(val name: String, val id: Int)
```
| Function            | Description                                                                 |
|---------------------|------------------------------------------------------------------------------|
| `toString()`        | 클래스 인스턴스와 그 프로퍼티들을 읽기 쉬운 문자열 형태로 출력한다.          |
| `equals()` or `==`  | 클래스 인스턴스들을 비교한다.                                                |
| `copy()`            | 다른 인스턴스를 복사하여 새 인스턴스를 생성하며, 일부 프로퍼티는 변경 가능하다. |

##### Copy instance
- 데이터 클래스 인스턴스의 완전히 동일한 복사본 을 생성하려면, 해당 인스턴스에서 copy() 함수를 호출하면 된다.
- 일부 프로퍼티 값을 변경한 복사본을 생성하려면, copy() 함수 호출 시 변경할 프로퍼티를 함수 매개변수로 전달 하면 된다.
```kotlin
// Creates an exact copy of user
println(user.copy())       
// User(name=Alex, id=1)

// Creates a copy of user with name: "Max"
println(user.copy("Max"))  
// User(name=Max, id=1)

// Creates a copy of user with id: 3
println(user.copy(id = 3)) 
// User(name=Alex, id=3)
```

##### instance 비교
```
val user = User("Alex", 1)
val secondUser = User("Alex", 1)
val thirdUser = User("Max", 2)

// Compares user to second user
println("user == secondUser: ${user == secondUser}") 
// user == secondUser: true

// Compares user to third user
println("user == thirdUser: ${user == thirdUser}")   
// user == thirdUser: false
```

#### Null Safe
- Kotlin은 값이 없거나 아직 설정되지 않은 경우에 null 값을 사용한다.
- 이미 컬렉션(Collections) 장에서, 맵(map)에 존재하지 않는 키로 키-값 쌍에 접근했을 때 Kotlin이 null 을 반환하는 예제를 본 적이 있을 것이다.
- 이처럼 null 값을 사용하는 것은 유용하지만, 코드가 null 을 처리하도록 준비되어 있지 않다면 문제가 발생할 수 있다.

이러한 문제를 방지하기 위해 Kotlin에는 널 안정성(null safety) 이 존재한다.
- 널 안정성은 실행 시(run time) 가 아니라 컴파일 시점(compile time) 에 null 로 인한 잠재적인 문제를 감지한다.

널 안정성은 다음과 같은 기능들의 조합이다:
- 프로그램에서 null 값 허용 여부를 명시적으로 선언 할 수 있다.
- null 값 여부를 검사 할 수 있다.
- null 값을 가질 수 있는 프로퍼티나 함수에 대해 안전 호출(safe call) 을 사용할 수 있다.
- null 값이 감지되었을 때 수행할 동작을 선언 할 수 있다.

Nullable 타입
- Kotlin은 nullable 타입을 지원하며, 이를 통해 선언된 타입이 null 값을 가질 수 있도록 할 수 있다.
    - 기본적으로 Kotlin의 타입은 null 값을 허용하지 않는다.
    - nullable 타입은 타입 선언 뒤에 ?를 명시적으로 추가 하여 선언한다.
```kotlin
fun main(){
  val neverNull : String = "This can't be null"
  neverNull = null
  // Throws a compiler error

  var nullable: String? = "You can keep a null here"
  nullable = null

  // By default, null values aren't accepted
  var inferredNonNull = "The compiler assumes non-nullable"

  // Throws a compiler error
  inferredNonNull = null

  fun strLength(notnull: String): Int{
    return notnull.length
  }

  println(strLength(neverNull)) // 18
  println(strLength(nullable))  // Throws a compiler error
}
```

##### Check for null values
```kotlin
fun describeString(): String{
  if (maybeString != null && maybeString.length > 0){
    return "String of length ${maybeString.length}"
  } else {
    return "Empty or null string"
  }
}
```

##### Use safe calls
- null 값을 가질 수 있는 객체의 프로퍼티에 안전하게 접근 하려면 안전 호출 연산자 `?.` 를 사용한다.
- 안전 호출 연산자는 객체 자체가 null 이거나, 접근하려는 프로퍼티 중 하나가 null 인 경우 null 을 반환 한다.
- 이 방식은 null 값으로 인해 코드에서 오류가 발생하는 것을 방지할 때 유용하다.

```kotlin
fun lengthString(maybeString: String?): Int? = maybeString?.length

fun main(){
  val nullString: String? = null
  println(lengthString(nullString)) // null
}

```
- 여기서 특징은 Compiler Error가 나오는게 아니라 null을 반환하는거 같다.

```kotlin
person.company?.address?.country
```
- 위의 형태처럼 체이닝 형태로 접근할 수 있는데 1개라도 null이면 null을 반환한다.




















