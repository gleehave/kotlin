#### Open Classes
- 인터페이스나 추상 클래스를 사용할 수 없으면, Open 을 사용해서 클래스를 상속하도록 명시적 선언가능
```kotlin 
open class Vehicle(val make: String, val model: String)

class Car(make: String, model: String, val numberOfDoors: Int) : Vehicle(make, model)

fun main() {
    // Creates an instance of the Car class
    val car = Car("Toyota", "Corolla", 4)

    // Prints the details of the car
    println("Car Info: Make - ${car.make}, Model - ${car.model}, Number of doors - ${car.numberOfDoors}")
    // Car Info: Make - Toyota, Model - Corolla, Number of doors - 4
}
```

##### 상속된 fun 재정의 하기
- 어떤 클래스를 상속받되 일부 동작을 변경하고 싶다면, 상속된 동작을 재정의(override) 할 수 있습니다.
- 기본적으로 부모 클래스의 멤버 함수나 프로퍼티는 재정의할 수 없습니다.
- 추상 클래스와 마찬가지로, 이를 위해서는 특정 키워드를 추가해야 합니다.

```kotlin
open fun displayInfo(){}
```

```kotlin
override fun displayInfo(){}
```

```kotlin
open class Vehicle(val make: String, val model: String) {
    open fun displayInfo() {
        println("Vehicle Info: Make - $make, Model - $model")
    }
}

class Car(make: String, model: String, val numberOfDoors: Int) : Vehicle(make, model) {
    override fun displayInfo() {
        println("Car Info: Make - $make, Model - $model, Number of Doors - $numberOfDoors")
    }
}

fun main() {
    val car1 = Car("Toyota", "Corolla", 4)
    val car2 = Car("Honda", "Civic", 2)

    // Uses the overridden displayInfo() function
    car1.displayInfo()
    // Car Info: Make - Toyota, Model - Corolla, Number of Doors - 4
    car2.displayInfo()
    // Car Info: Make - Honda, Model - Civic, Number of Doors - 2
}
```
1. Vehicle 클래스를 상속하는 Car 클래스의 두 인스턴스 car1과 car2를 생성합니다.
2. Car 클래스에서 displayInfo() 함수를 재정의하여, 기존 정보에 더해 **문의 개수(number of doors)**도 함께 출력하도록 합니다.
3. car1과 car2 인스턴스에서 재정의된 displayInfo() 함수를 호출합니다.

##### Open과 Interface
- 클래스를 하나 상속하면서 동시에 여러 인터페이스를 구현할 수 있습니다.
- 이 경우, 콜론(:) 뒤에 먼저 부모 클래스를 선언한 다음, 그 뒤에 구현할 인터페이스들을 나열해야 합니다.

```kotlin
// 인터페이스 정의
interface EcoFriendly {
    val emissionLevel: String
}

interface ElectricVehicle {
    val batteryCapacity: Double
}

// 부모 클래스
open class Vehicle(val make: String, val model: String)

// 자식 클래스
open class Car(
    make: String,
    model: String,
    val numberOfDoors: Int
) : Vehicle(make, model)

// Car를 상속하고 두 개의 인터페이스를 구현하는 새로운 클래스
class ElectricCar(
    make: String,
    model: String,
    numberOfDoors: Int,
    val capacity: Double,
    val emission: String
) : Car(make, model, numberOfDoors), EcoFriendly, ElectricVehicle {

    override val batteryCapacity: Double = capacity
    override val emissionLevel: String = emission
}
```

##### Enum 클래스
- Enum 클래스는 유한하고 서로 구별되는 값들의 집합을 표현할 때 유용합니다.
- Enum 클래스는 enum 상수(enum constants)를 포함하며, 이 상수들은 해당 enum 클래스의 인스턴스입니다.

```kotlin
enum class State

enum class State{
  IDLE, RUNNING, FINISHED
}

val state = State.RUNNING
```

```kotlin
enum class State {
    IDLE, RUNNING, FINISHED
}

fun main() {
    val state = State.RUNNING

    val message = when (state) {
        State.IDLE -> "It's idle"
        State.RUNNING -> "It's running"
        State.FINISHED -> "It's finished"
    }

    println(message)
    // It's running
}
```

```kotlin
enum class Color(val rgb: Int) {
    RED(0xFF0000),
    GREEN(0x00FF00),
    BLUE(0x0000FF),
    YELLOW(0xFFFF00);

    fun containsRed() = (this.rgb and 0xFF0000 != 0)
}

fun main() {
    val red = Color.RED
    
    // Calls containsRed() function on enum constant
    println(red.containsRed())
    // true

    // Calls containsRed() function on enum constants via class names
    println(Color.BLUE.containsRed())
    // false
  
    println(Color.YELLOW.containsRed())
    // true
}
```




