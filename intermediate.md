#### Extension 함수

- 소프트웨어 개발에서는 원본 소스 코드를 변경하지 않고도 프로그램의 동작을 수정해야 하는 경우가 자주 있습니다.
  - 예를 들어, 서드파티 라이브러리의 클래스에 추가 기능을 넣고 싶을 때가 그렇습니다.
- 이런 경우 확장 함수(extension function) 를 사용해 클래스를 확장할 수 있습니다.
  - 확장 함수는 클래스의 멤버 함수와 동일하게 점(.) 연산자를 사용해 호출합니다.
- 확장 함수의 전체 문법을 소개하기 전에, 먼저 리시버(receiver) 개념을 이해해야 합니다.
  - 리시버란 함수가 호출되는 대상을 의미합니다. 다시 말해, 정보가 전달되거나 공유되는 대상이 리시버입니다.
 
sender와 receiver 예시
- `.first()` 함수는 readOnlyShapes 변수에 대해 호출되므로,
- readOnlyShapes 변수가 리시버(receiver) 입니다.

```kotlin
fun main(){
  val readOnlyShapes = listOf("triangle", "square", "circle")
  println("The first item in the list is : ${readOnlyShapes.first()}")
}
```

##### 확장 함수 정의 방법
- 확장 함수를 만들려면, 확장하려는 클래스 이름 뒤에 .을 붙이고 함수 이름을 작성합니다.
- 그 다음 일반 함수와 동일하게 매개변수와 반환 타입을 포함한 함수 선언을 이어서 작성하면 됩니다.

```kotlin
fun String.bold(): String = "<b>$this</b>"

fun main(){
  println("hello".bold())
}
```
1. String은 확장 대상 클래스입니다.
2. bold는 확장 함수의 이름입니다.
3. .bold() 확장 함수의 반환 타입은 String입니다.
4. "hello"는 String의 인스턴스이며, 리시버(receiver)입니다.
5. **리시버는 함수 본문 안에서 this 키워드를 통해 접근할 수 있습니다.**
    - this로 receiver에서 조작한다.!!!!
6. 문자열 템플릿($)을 사용해 this의 값을 참조합니다.
7. .bold() 확장 함수는 문자열을 받아서,
8. 굵은 텍스트를 나타내는 <b> HTML 요소로 감싼 문자열을 반환합니다.

---

- 확장 함수는 어디에서든 정의할 수 있기 때문에, 확장 중심 설계(extension-oriented design) 를 만들 수 있습니다.
- 이러한 설계 방식은 핵심 기능(core functionality)과 유용하지만 필수는 아닌 기능을 분리하여, 코드를 더 읽기 쉽고 유지보수하기 쉽게 만들어 줍니다.

Ktor 라이브러리의 HttpClient 클래스가 예시이다.
- 이 클래스의 핵심 기능은 request()라는 단 하나의 함수로 구성되어 있으며, 이 함수는 HTTP 요청에 필요한 모든 정보를 인자로 받아 처리합니다.
```kotlin
class HttpClient{
  fun request(method: String, url: String, headers: Map<String, String>): HttpResponse{
    // Network 관련 로직 작성되어있다고 가정
  }
}
```

- HTTP 요청이 GET 또는 POST 요청
- 이처럼 자주 쓰이는 경우를 위해 라이브러리에서 더 짧고 명확한 이름을 제공하는 것은 합리적입니다.
- 하지만 이러한 기능을 위해 새로운 네트워크 코드를 작성할 필요는 없습니다.
- 특정한 방식으로 request() 함수를 호출하기만 하면 되기 때문입니다.
- 즉, 이들은 별도의 .get() 및 .post() 확장 함수로 정의하기에 완벽한 대상입니다.

```kotlin
fun HttpClient.get(url: String): HttpResponse = request("GET", url, emptyMap())
fun HttpClient.post(url: String): HttpResponse = request("POST", url, emptyMap())
```
- .get()과 .post() 함수들은 HttpClient 클래스를 확장
- 이 함수들은 리시버(receiver)로 HttpClient 인스턴스에서 호출되기 때문에, HttpClient 클래스의 request() 함수를 직접 사용할 수 있습니다.

```kotlin
class HttpClient{
  fun request(method: String, url: String, headers: Map<String, String>): HttpResponse{
    println("Requesting $method to $url with headers: $headers")
    return HttpResponse("Response from $url")
  }
}

fun HttpClient.get(url: String): HttpResponse = request("GET", url, emptyMap())

fun main() {
    val client = HttpClient()

    // request() 멤버 함수를 직접 사용해 GET 요청
    val getResponseWithMember = client.request("GET", "https://example.com", emptyMap())

    // get() 확장 함수를 사용해 GET 요청
    // 이때 client 인스턴스가 리시버
    val getResponseWithExtension = client.get("https://example.com")
}
```

#### Scope functions
- 전역 스코프(Global scope) 프로그램 어디에서든 접근할 수 있는 변수 또는 객체
- 지역 스코프(Local scope) 정의된 블록이나 함수 내부에서만 접근할 수 있는 변수 또는 객체

Kotlin에는 객체를 중심으로 임시 스코프를 생성하고 특정 코드를 실행할 수 있게 해주는 스코프 함수(scope function) 가 존재함.
- 스코프 함수를 사용하면, 임시 스코프 안에서 객체 이름을 반복해서 작성할 필요가 없어 코드를 더 간결하게 만들 수 있습니다.
- 사용하는 스코프 함수에 따라, 객체는 `this` 키워드로 접근하거나 `it` 키워드를 통해 람다의 인자로 접근할 수 있습니다.

it로 접근하는건 신기하네

Kotlin에는 총 다섯 가지 스코프 함수가 있습니다.
- let
- apply
- run
- also
- with
- 각 스코프 함수는 람다 표현식(lambda expression) 을 인자로 받아, 객체 자체 또는 람다 실행 결과를 반환합니다.

##### Let
- 코드에서 null 체크를 수행하고, 그 이후에 반환된 객체를 계속 사용하고 싶을 때 let 스코프 함수를 사용
```kotlin
fun sendNotification(recipientAddress: String): String{
  println("Yl $recipientAddress")
  return "Notification Sent!"
}

fun getNextAddress(): String{
  return "sebastian@jetbrains.com"
}

fun main(){
  val address: String? = getNextAddress()
  sendNotification(address)
}
```
- nullable 타입(String?) 인 address 변수를 생성한다. 하지만 sendNotification() 함수를 호출할 때 문제가 발생.
- 이 함수는 address가 null일 수 있다는 것을 허용하지 않기 때문에 발생함
```
Argument type mismatch: actual type is 'String?', but 'String' was expected.
```

##### null check 해서 처리
```kotlin
val address: String? = getNextAddress()
val confirm = if (address != null) {
    sendNotification(address)
} else {
    null
}
```

##### let 으로 처리
```kotlin
val address: String? = getNextAddress()
val confirm = address?.let {
    sendNotification(it)
}
```
1. address와 confirm 변수를 생성합니다.
2. address 변수에 대해 safe call(?.) 을 사용해 let 스코프 함수를 호출
3. let 스코프 함수 내부에 임시 스코프를 생성
4. **sendNotification() 함수를 람다 표현식으로 let에 전달**
5. 임시 스코프 안에서 it 키워드를 사용해 address 변수에 접근
6. 실행 결과를 confirm 변수에 할당

##### Apply
- apply 스코프 함수는 객체(예: 클래스 인스턴스)를 생성한 직후에 초기화할 때 사용
- 이 방식을 사용하면, 객체를 생성한 뒤에 별도로 설정하는 대신 생성과 초기화를 한 번에 처리할 수 있음

```kotlin
class Client(){
  var token: String? = null

  fun connect() = println("Connected!")
  fun authenticate() = println("Authenticated!")
  fun getData() : String{
    println("getting data!")
    return "Mock Data"
  }
}

val client = Client()

fun main(){
  client.token = "aaaa"
  client.connect()
  client.authenticate()
  client.getData()
}
```
- token이라는 하나의 프로퍼티와 connect(), authenticate(), getData()라는 세 개의 멤버 함수를 가진 Client 클래스
- Client 클래스의 인스턴스인 client를 먼저 생성한 뒤, token 프로퍼티를 초기화하고 멤버 함수들을 호출
- 실제 환경에서는 클래스 인스턴스를 생성한 후 설정(configure)하고 사용하기까지 시간이 걸리는 경우가 많습니다.

하지만 apply 스코프 함수를 사용하면, 클래스 인스턴스를 생성, 설정, 멤버 함수 호출까지 모두 코드의 한 위치에서 한 번에 처리 가능
```kotlin
val client = Client().apply{
  token = "asdf"
  connect()
  authenticate()
}

fun main(){
  client.getData()
}
```
1. Client 클래스의 인스턴스로 client를 생성
2. client 인스턴스에 대해 apply 스코프 함수를 사용
3. apply 스코프 함수 내부에 임시 스코프를 생성하여, 프로퍼티나 함수를 사용할 때 client 인스턴스를 명시적으로 참조할 필요가 없음
4. apply 스코프 함수에 전달된 람다 표현식에서 token 프로퍼티를 업데이트하고 connect()와 authenticate() 함수를 호출
5. main() 함수에서 client 인스턴스의 getData() 멤버 함수를 호출

##### Run
- run 스코프 함수는 apply와 유사하게 객체를 초기화하는 데 사용할 수 있지만, 특정 시점에 객체를 초기화하고 그 즉시 결과를 계산해야 할 때 더 적합
```kotlin
val client: Client = Client().apply {
    token = "asdf"
}

fun main() {
    val result: String = client.run {
        connect()
        // connected!
        authenticate()
        // authenticated!
        getData()
        // getting data!
    }
}
```
1. Client 클래스의 인스턴스로 client를 생성합니다.
2. client 인스턴스에 대해 apply 스코프 함수를 사용합니다.
3. apply 스코프 함수 내부에 임시 스코프를 생성하여, 프로퍼티나 함수에 접근할 때 client 인스턴스를 명시적으로 참조하지 않아도 되도록 합니다.
4. apply 스코프 함수에 전달된 람다 표현식에서 token 프로퍼티를 업데이트합니다.
5. String 타입의 result 변수를 생성합니다.
6. client 인스턴스에 대해 run 스코프 함수를 사용합니다.
7. run 스코프 함수 내부에 임시 스코프를 생성하여, 프로퍼티나 함수에 접근할 때 client 인스턴스를 명시적으로 참조하지 않아도 되도록 합니다.
8. run 스코프 함수에 전달된 람다 표현식에서 connect(), authenticate(), getData() 함수를 호출합니다.
9. 실행 결과를 result 변수에 할당합니다.

##### Also
- also 스코프 함수는 객체에 대해 부가적인 작업(예: 로그 출력) 을 수행한 뒤, 객체 자체를 그대로 반환하여 이후 코드에서 계속 사용하고 싶을 때 사용

```kotlin
fun main(){
  val medals: List<String> = listOf("Gold", "Silver", "Bronze")
  val reversedLongUppercaseMedals: List<String> = medals.map{it.uppercase()}.filter{it.length > 4}.reversed()
}
``` 
1. 문자열 리스트를 담고 있는 medals 변수를 생성합니다.
2. List<String> 타입의 reversedLongUppercaseMedals 변수를 생성합니다.
3. medals 변수에 대해 .map() 확장 함수를 사용합니다.
4. .map()에 람다 표현식을 전달하고, it 키워드를 통해 각 요소에 접근하여 .uppercase() 확장 함수를 호출합니다.
6. medals 변수에 대해 .filter() 확장 함수를 사용합니다.
7. .filter()에 조건(predicate) 람다를 전달하고, 각 요소의 길이가 4보다 큰지 검사합니다.
8. medals 변수에 대해 .reversed() 확장 함수를 사용합니다.
9. 그 결과를 reversedLongUppercaseMedals 변수에 할당합니다.

> medals 변수가 어떻게 변하는지 확인하기 위해 로그를 추가하고 싶을 수 있습니다.

```kotlin
fun main(){
  val medals: List<String> = listOf("GOLD", "SILVER", "BRONZE")
  val reversedLongUppdercaseMedals: List<String> = medals
    .map{ it.ppercase() }
    .also {println(it)}
    .filter{it.length > 4}
    .also{println(it)}
    .reversed()
}
```

1. medals 변수에 대해 also 스코프 함수를 사용합니다.
2. also 스코프 함수 내부에 임시 스코프를 생성하여, medals 변수를 명시적으로 참조하지 않고도 사용할 수 있습니다.
3. also에 전달된 람다 표현식에서 it 키워드를 사용해 medals를 함수 인자로 전달하여 println()을 호출합니다.
4. also 함수는 객체 자체를 그대로 반환합니다.
5. 이 때문에 also는 로그 출력뿐만 아니라, 디버깅, 여러 연산의 체이닝, 그리고 주요 로직에 영향을 주지 않는 부수 효과(side effect) 를 수행할 때 매우 유용

##### with
- 다른 스코프 함수들과 달리, with는 확장 함수가 아님
- 따라서 문법이 다르며, 리시버 객체를 인자로 전달해야 합니다.

with 스코프 함수는 하나의 객체에 대해 여러 함수를 연속으로 호출하고 싶을 때 사용합니다.
```kotlin
class Canvas {
    fun rect(x: Int, y: Int, w: Int, h: Int): Unit = println("$x, $y, $w, $h")
    fun circ(x: Int, y: Int, rad: Int): Unit = println("$x, $y, $rad")
    fun text(x: Int, y: Int, str: String): Unit = println("$x, $y, $str")
}

fun main() {
    val mainMonitorPrimaryBufferBackedCanvas = Canvas()

    mainMonitorPrimaryBufferBackedCanvas.text(10, 10, "Foo")
    mainMonitorPrimaryBufferBackedCanvas.rect(20, 30, 100, 50)
    mainMonitorPrimaryBufferBackedCanvas.circ(40, 60, 25)
    mainMonitorPrimaryBufferBackedCanvas.text(15, 45, "Hello")
    mainMonitorPrimaryBufferBackedCanvas.rect(70, 80, 150, 100)
    mainMonitorPrimaryBufferBackedCanvas.circ(90, 110, 40)
    mainMonitorPrimaryBufferBackedCanvas.text(35, 55, "World")
    mainMonitorPrimaryBufferBackedCanvas.rect(120, 140, 200, 75)
    mainMonitorPrimaryBufferBackedCanvas.circ(160, 180, 55)
    mainMonitorPrimaryBufferBackedCanvas.text(50, 70, "Kotlin")
}
```
1. rect(), circ(), text()라는 세 개의 멤버 함수를 가진 Canvas 클래스를 정의합니다.
2. 각 멤버 함수는 전달된 파라미터를 이용해 문자열을 출력합니다.
3. Canvas 클래스의 인스턴스인 mainMonitorPrimaryBufferBackedCanvas를 생성합니다.

```kotlin
val mainMonitorSecondaryBufferBackedCanvas = Canvas()

with(mainMonitorSecondaryBufferBackedCanvas) {
    text(10, 10, "Foo")
    rect(20, 30, 100, 50)
    circ(40, 60, 25)
    text(15, 45, "Hello")
    rect(70, 80, 150, 100)
    circ(90, 110, 40)
    text(35, 55, "World")
    rect(120, 140, 200, 75)
    circ(160, 180, 55)
    text(50, 70, "Kotlin")
}
```
1. mainMonitorSecondaryBufferBackedCanvas 인스턴스를 리시버로 하여 with 스코프 함수를 사용합니다.
2. with 내부에 임시 스코프가 생성되어, 멤버 함수를 호출할 때 인스턴스 이름을 반복해서 작성할 필요가 없습니다.
3. with에 전달된 람다 표현식 안에서 서로 다른 인자를 사용해 여러 멤버 함수를 연속으로 호출합니다.

##### 요약

| Function | 객체 접근 방식 | 반환값   | 주요 사용 사례                                 |
| -------- | -------- | ----- | ---------------------------------------- |
| `let`    | `it`     | 람다 결과 | null 체크를 수행하고, 이후 반환된 객체로 추가 작업을 수행할 때   |
| `apply`  | `this`   | 객체 자체 | 객체를 생성하는 시점에 초기화할 때                      |
| `run`    | `this`   | 람다 결과 | 객체를 초기화하면서 **동시에 결과를 계산**할 때             |
| `also`   | `it`     | 객체 자체 | 객체를 반환하기 전에 **추가 작업(로그, 디버깅 등)** 을 수행할 때 |
| `with`   | `this`   | 람다 결과 | 하나의 객체에 대해 여러 함수를 호출할 때                  |
