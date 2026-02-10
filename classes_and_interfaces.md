#### 클래스 상속(Class Inheritance)
- 기본적으로 Kotlin의 클래스는 상속되지 않는다.
  - 단일 상속만 지원한다.
  - 어떤 클래스의 부모 클래스는 다시 다른 클래스(즉, 조부모 클래스(grandparent))를 상속받을 수 있으며, 이렇게 계층 구조(hierarchy) 가 형성됩니다.
  - Kotlin의 클래스 계층 구조 최상단에는 공통 부모 클래스인 Any 가 있습니다.
  - 모든 클래스는 결국 Any 클래스를 상속받습니다
  -  Any 클래스는 `toString()` 함수를 자동으로 제공
  -  클래스들 간에 일부 코드를 공유하기 위해 상속을 사용하고 싶다면, 먼저 **추상 클래스(abstract class) 를 사용하는 것을 고려**해본다

```kotlin
val car1 = Car("Toyota", "Corolla", 4)

// Uses the .toString() function via string templates to print class properties
println("Car1: make=${car1.make}, model=${car1.model}, numberOfDoors=${car1.numberOfDoors}")
// Car1: make=Toyota, model=Corolla, numberOfDoors=4
```

#### Abstract classes
- 추상 클래스(abstract class) 는 기본적으로 상속이 가능
- 추상 클래스의 목적은 다른 클래스가 상속하거나 구현(implement) 할 멤버를 제공 
- 그 결과 추상 클래스는 생성자(constructor) 를 가질 수 있지만, 인스턴스를 직접 생성할 수는 없습니다.
```kotlin
abstract class Product(val name: String, var price: Double){
  abstract val category: String

  fun productInfo(): String{
    return "Product: $name, Category: $category, Price: $price"
  }
}
```

- 생성자는 상품의 이름(name) 과 가격(price) 을 위한 두 개의 매개변수
- 상품의 카테고리(category) 를 문자열로 담는 추상 프로퍼티
- 상품에 대한 정보를 출력하는 함수

```kotlin
class Electronic(name: String, price: Double, val warranty: Int): Product(name, price){
  override val category = "Electronic"
}
```
Electronic 클래스는:
- Product 추상 클래스를 상속
- 생성자에 전자제품에 특화된 추가 매개변수인 warranty(보증기간)를 가짐
- category 프로퍼티를 오버라이드하여 문자열 "Electronic" 을 가짐

```kotlin
fun main() {
    // Electronic 클래스의 인스턴스를 생성
    val laptop = Electronic(name = "Laptop", price = 1000.0, warranty = 2)

    println(laptop.productInfo())
    // Product: Laptop, Category: Electronic, Price: 1000.0
}
```
- 추상 클래스는 이런 방식으로 코드를 공유하기에 매우 유용하지만, Kotlin의 클래스는 단일 상속만 지원하기 때문에 제약 존재
- 여러 출처로부터 상속(또는 구현)이 필요하다면 인터페이스(interfaces) 사용을 고려

#### Interfaces

- 인터페이스는 인스턴스를 생성할 수 없습니다. 생성자나 클래스 헤더(기본 생성자 선언부)가 없음
- 인터페이스의 함수와 프로퍼티는 기본적으로 상속(구현) 가능
- 구현을 제공하지 않는 함수는 abstract로 표시할 필요가 없습니다. (자동으로 추상 멤버가 됩니다)
- 추상 클래스와 마찬가지로, 인터페이스는 클래스들이 나중에 상속(구현)하고 구현할 수 있는 함수와 프로퍼티의 집합을 정의하는 데 사용
- 인터페이스를 선언하려면 interface 키워드를 사용

```kotlin
interface PaymentMethod
```

인터페이스는 다중 상속을 지원하므로, 하나의 클래스가 여러 인터페이스를 동시에 구현할 수 있습니다. 
인터페이스를 구현하는 클래스를 만들려면, 클래스 헤더 뒤에 콜론(:)을 붙이고 구현할 인터페이스 이름을 작성
인터페이스에는 생성자가 없기 때문에 인터페이스 이름 뒤에 괄호 ()를 붙이지 않습니다:

```kotlin
class CreditCardPayment : PaymentMethod
```

```kotlin
interface PaymentMethod{
  fun initiatePayment(amount: Double): String
}

class CreditCardPayment(
  val cardNumber: String,
  val cardHolderName: String,
  val expiryDate: String
): PaymentMethod{
  override fun initiatePayment(amount: Double): String{
    return "Payment of $$amount initiated using Credit Card ending in ${cardNumber.takeLast(4)}"
  }
}

fun main() {
    val paymentMethod = CreditCardPayment("1234 5678 9012 3456", "John Doe", "12/25")
    println(paymentMethod.initiatePayment(100.0))
    // Payment of $100.0 initiated using Credit Card ending in 3456.
}
```

- 클래스가 여러 인터페이스를 구현하도록 만들려면, 
- 클래스 헤더 뒤에 콜론(:)을 붙이고 구현할 인터페이스 이름들을 쉼표(,)로 구분해 나열

```kotlin
class CreditCardPayment: PaymentMethod, PaymentType
```

```kotlin
interface PaymentMethod {
    fun initiatePayment(amount: Double): String
}

interface PaymentType {
    val paymentType: String
}

class CreditCardPayment(val cardNumber: String, val cardHolderName: String, val expiryDate: String) : PaymentMethod,
    PaymentType {
    override fun initiatePayment(amount: Double): String {
        // Simulate processing payment with credit card
        return "Payment of $$amount initiated using Credit Card ending in ${cardNumber.takeLast(4)}."
    }

    override val paymentType: String = "Credit Card"
}

fun main() {
    val paymentMethod = CreditCardPayment("1234 5678 9012 3456", "John Doe", "12/25")
    println(paymentMethod.initiatePayment(100.0))
    // Payment of $100.0 initiated using Credit Card ending in 3456.

    println("Payment is by ${paymentMethod.paymentType}")
    // Payment is by Credit Card
}
```

#### 위임(Delegation)
- 인터페이스는 유용하지만, 인터페이스에 함수가 많다면 이를 구현하는 자식 클래스들에 보일러플레이트(boilerplate) 코드가 많이 생길 수 있습니다.
- 클래스의 동작 중 일부분만 오버라이드하고 싶은 경우에도, 같은 코드를 여러 번 반복해야 하는 일이 발생합니다.

보일러플레이트 코드란, 소프트웨어 프로젝트의 여러 부분에서 거의 또는 전혀 변경 없이 반복해서 재사용되는 코드 덩어리를 말합니다.
예를 들어, 여러 함수와 color라는 프로퍼티 하나를 가진 DrawingTool이라는 인터페이스가 있다고 해봅시다:

```kotlin
interface DrawingTool {
    val color: String
    fun draw(shape: String)
    fun erase(area: String)
    fun getToolInfo(): String
}
```

- DrawingTool 인터페이스를 구현하고, 모든 멤버에 대한 구현을 제공하는 PenTool 클래스

```kotlin
class PenTool : DrawingTool {
    override val color: String = "black"

    override fun draw(shape: String) {
        println("Drawing $shape using a pen in $color")
    }

    override fun erase(area: String) {
        println("Erasing $area with pen tool")
    }

    override fun getToolInfo(): String {
        return "PenTool(color=$color)"
    }
}

```

- PenTool과 동작은 동일하지만 color 값만 다른 클래스를 만들고 싶다면?
- 한 가지 접근 방식은, DrawingTool 인터페이스를 구현한 객체(예: PenTool 인스턴스)를 파라미터로 받는 새 클래스를 만드는 것입니다. 
- 그 클래스 내부에서 color 프로퍼티만 오버라이드할 수 있습니다.

```kotlin
class CanvasSession(val tool: DrawingTool) : DrawingTool {
    override val color: String = "blue"

    override fun draw(shape: String) {
        tool.draw(shape)
    }

    override fun erase(area: String) {
        tool.erase(area)
    }

    override fun getToolInfo(): String {
        return tool.getToolInfo()
    }
}
```
- DrawingTool 인터페이스에 멤버 함수가 많이 있다면, CanvasSession 클래스에 들어가는 보일러플레이트 코드의 양이 매우 커질 수 있음을 알 수 있습니다.
- by 키워드를 사용하여 인터페이스 구현을 클래스 인스턴스에 위임(delegate) 할 수 있습니다.

```kotlin
class CanvasSession(val tool: DrawingTool) : DrawingTool by tool {
    // 보일러플레이트 코드 없음!
    override val color: String = "blue"
}
```


