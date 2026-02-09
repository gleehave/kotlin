#### Lambda
> 람다 표현식은 리시버(receiver) 를 가질 수도 있습니다.
> 이 경우, 람다 표현식 안에서 매번 리시버를 명시적으로 지정하지 않아도 리시버의 모든 멤버 함수나 프로퍼티에 접근할 수 있습니다.

> 리시버를 가지는 람다 표현식은 리시버를 가진 함수 리터럴(function literals with receiver) 이라고도 부릅니다.
> 리시버를 가진 람다 표현식의 문법은 함수 타입을 정의할 때 다릅니다. 먼저 확장하고자 하는 리시버를 작성한 뒤, . 을 붙이고 나머지 함수 타입 정의를 완성합니다.

```kotlin
MutableList<Int>.() -> Unit
```
- 리시버: MutableList<Int>
- 괄호 () 안에 함수 파라미터 없음
- 반환값 없음: Unit

```kotlin
class Canvas {
    fun drawCircle() = println("🟠 Drawing a circle")
    fun drawSquare() = println("🟥 Drawing a square")
}

// Lambda expression with receiver definition
fun render(block: Canvas.() -> Unit): Canvas {
    val canvas = Canvas()
    // Use the lambda expression with receiver
    canvas.block()
    return canvas
}

fun main() {
    render {
        drawCircle()
        // 🟠 Drawing a circle
        drawSquare()
        // 🟥 Drawing a square
    }
}
```
- Canvas 클래스에는 원과 사각형을 그리는 동작을 시뮬레이션하는 두 개의 함수가 있습니다.
- render() 함수는 block 파라미터를 받아 Canvas 클래스의 인스턴스를 반환합니다.
- block 파라미터는 리시버를 가진 람다 표현식이며, 이때 Canvas 클래스가 리시버입니다.
- render() 함수는 Canvas 클래스의 인스턴스를 생성한 다음, 해당 인스턴스를 리시버로 사용하여 block() 람다 표현식을 호출합니다.
- main() 함수는 람다 표현식을 인자로 전달하여 render() 함수를 호출하며, 이 람다 표현식은 block 파라미터로 전달됩니다.
- render() 함수에 전달된 람다 내부에서, 프로그램은 Canvas 클래스의 인스턴스에 대해 drawCircle() 과 drawSquare() 함수를 호출합니다.
- drawCircle() 과 drawSquare() 함수는 리시버를 가진 람다 표현식 안에서 호출되므로, 마치 Canvas 클래스 내부에 있는 것처럼 직접 호출할 수 있습니다.

```kotlin
class MenuItem(val name: String)

class Menu(val name: String){
  val items = mutableListOf<MenuItem>()

  fun item(name: String){
    items.add(MenuItem(name))
  }
}

fun menu(name: String, init: Menu.() -> Unit): Menu {
    val menu = Menu(name)
    menu.init()  // python에서는 init(menu)
    return menu
}

fun printMenu(menu: Menu){
  println("Menu: ${menu.name}")
  menu.items.forEach { println(" Item: ${it.name}")}
}

fun main(){
  val mainMenu = menu("Main Menu"){
    item("Home")
    item("Settings")
    item("Exit")
  }
  // Print the menu
  printMenu(mainMenu)
  // Menu: Main Menu
  //   Item: Home
  //   Item: Settings
  //   Item: Exit
}
```

##### Exercise 1
```kotlin
fun fetchData(callback: StringBuilder.() -> Unit) {
    println("3")
    val builder = StringBuilder("Data received")
    println("4")
    builder.callback()
    println("5")
}

fun main() {
    fetchData {
        // Write your code here
        // Data received - Prddocessed
        println("1")
        append("- Processed")
        println("2")
        println(this.toString())
        println("6")
    }
}

3
4
1
2
Data received- Processed
6
5

main()
 └─ fetchData() 시작
     ├─ print 3
     ├─ print 4
     ├─ callback 실행
     │   ├─ print 1
     │   ├─ append
     │   ├─ print 2
     │   ├─ print builder
     │   └─ print 6
     └─ print 5
```

##### Exercise 2
```kotlin
class Button{
    fun onEvent(action: ButtonEvent.() -> Unit){
        val event = ButtonEvent(
            isRightClick=false,
            amount=2,
            potion=Position(100, 200)
        )

        event.action()
    }
}

data class ButtonEvent(
    val isRightClick: Boolean,
    val amount: Int,
    val position: Position
)

data class Position(
    val x : Int,
    val y : Int
)

fun main(){
    val button = Button()

    button.onEvent{
        if (!isRightClick && amount == 2){
            println("Double Click")
        }
    }
}
```

##### Exercise 3

```kotlin
fun List<Int>.incremented(): List<Int> {
    val originalList = this
    return buildList {
        // Write your code here
        for (n in originalList) add(n + 1)
    }
}

fun main() {
    val originalList = listOf(1, 2, 3)
    val newList = originalList.incremented()
    println(newList)
    // [2, 3, 4]
}
```

