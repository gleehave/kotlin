#### Object 선언
- Kotlin 에서는 단 하나의 인스턴스만 가지는 클래스를 선언할 수 있음.(e.g., Singleton)
- 프로그램에서 단일 참조 지점으로 유도할 수 있음.
- 기본적으로 Lazy Initialization 형태이기에 실제로 접근될 때에만 생성됨.
    - Thread Safe 형태를 Kotlin 언어에서 보장함.
```kotlin
object DoAuth {}
```

```
object DoAuth{
  fun takeParams(username: String, password: String){
    println("input Auth Paramters = $username:$password")
  }
}

fun main(){
  DoAuth.takeParams("coding_ninja", "NlnjaC0ding!")
}
```
- DoAuth는 takeParams가 처음 호출 될 때, 생성됨.

##### Data objects
- 객체 선언의 내용을 쉽게 출력할 수 있도록 Kotlin에서 제공함
- toString()
- equals()
- 단, copy()는 제공하지 않는데, 1개의 인스턴스만 가지기 때문에 복사할 수 없음.

```kotlin
data object AppConfig{}
```

```kotlin
data object AppConfig {
    var appName: String = "My Application"
    var version: String = "1.0.0"
}

fun main() {
    println(AppConfig)
    // AppConfig
    
    println(AppConfig.appName)
    // My Application
}
```

##### Companion objects
- Kotlin에서 클래스(Class)는 객체를 1개 가질 수 있다. 이 때, 이를 companion object 라고 함.
- 클래스(Class) 당 1개의 companion object를 선언함.
- companion object는 해당 클래스가 처음 참조될 때 생성됨.
- companion object는 내부에 선언된 모든 속성(property)와 함수(function)는 해당 클래스의 모든 인스턴스에서 공유됨.

```kotlin
class BigBen{
  companion object Bonger{
    fun getBongs(nTimes: Int){
      repeat(nTimes) {print("BONG ")}
    }
  }
}

fun main(){
  BigBen.getBongs(12)
}
```

- companion objects를 사용하면서 얻는 장점이 뭐지?

- 생성자는 내부에 두고, companion object의 create()와 같은 함수를 선언해서 Factory Method Pattern 형태로 사용 가능
```kotlin
class User private constructor(
    val name: String
) {
    companion object {
        fun create(name: String): User {
            require(name.isNotBlank())
            return User(name)
        }
    }
}
```

- 상수관리가 편함
```kotlin
class ApiClient {
    companion object {
        const val BASE_URL = "https://api.example.com"
    }
}
```

