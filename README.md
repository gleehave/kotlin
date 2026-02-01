# Python / Kotlin

#### 1. 변수
```python
name = "Anne"  # 가변
age = 32
```
```kotlin
var name = "Anne"   // 가변 (mutable)
val age = 32        // 불변 (read-only)
```

Kotlin은 정적 타입 언어로 var(가변)와 val(불변)을 명시적으로 구분합니다. 
Python과 달리 타입 추론이 가능하지만, 내부적으로 타입이 고정됩니다.

#### 2. 함수
```python
def greet(name="Guest"):
    return f"Hello, {name}!"
```
```kotlin
fun greet(name: String = "Guest"): String {
    return "Hello, $name!"
}
// 또는 한 줄로
fun greet(name: String = "Guest") = "Hello, $name!"
```
두 언어 모두 기본 매개변수를 지원하지만, Kotlin은 타입을 명시해야 하며 함수 선언 시 반환 타입도 지정해야 합니다. 
하지만 Python의 기본 매개변수 함정(mutable default arguments)을 Kotlin은 매 호출마다 평가하여 피할 수 있습니다.
​

#### 3. 컬렉션 처리
**List**
```python
nums = [1, 2, 3]
nums.append(4)
```
```kotlin
val nums = listOf(1, 2, 3)          // 불변
val mutableNums = mutableListOf(1, 2, 3)
mutableNums.add(4)
```
**Set**
```python
tags = {"kotlin", "python"}
tags.add("ktor")
```
```kotlin
val tags = setOf("kotlin", "python")          // 불변
val mutableTags = mutableSetOf("kotlin", "python")
mutableTags.add("ktor")
```
**Dict**
```python
user = {
    "id": 1,
    "name": "Anne"
}
user["age"] = 32
```
```kotlin
val user = mapOf(
    "id" to 1,
    "name" to "Anne"
)

val mutableUser = mutableMapOf(
    "id" to 1,
    "name" to "Anne"
)
mutableUser["age"] = 32
```
**List Comprehension**
```python
squares = [x * x for x in nums if x % 2 == 0]
```
```kotlin
val squares = nums
    .filter { it % 2 == 0 }
    .map { it * it }
```

#### 4. 클래스와 데이터 클래스
```python
@dataclass
class Post:
    id: int
    content: str
    author: Optional[str] = None
```
```kotlin
data class Post(
    val id: Int,
    val content: String,
    val author: String? = null
)
```

Kotlin의 data class는 언어 내장 기능으로 자동으로 equals(), hashCode(), toString(), copy() 등을 생성합니다. Python은 decorator가 필요하며, Kotlin의 ? 문법은 null safety를 언어 레벨에서 강제합니다.

- dataclass에서 equals()는 값을 비교.
- dataclass에서 hashCode()는 Set에 데이터가 add될 때, 같은 값인지 판단(값이 같으면 같은 hash)
- dataclass에서 copy()는 불변 객체를 안정하게 수정. 
```kotlin
val user = User(1, "LEE")
val person = user.copy(name="KIM")
```
​
#### 5. Coroutines (코루틴)

```kotlin
suspend fun fetchUser(): UserData = coroutineScope {
    val userDetails = async { api.fetchUserDetails() }
    val posts = async { api.fetchPosts() }
    UserData(userDetails.await(), posts.await())
}
```

Python의 async/await는 단일 스레드에서 실행되지만, 
Kotlin coroutine은 멀티스레드 풀에서 실행되어 CPU 코어를 더 효율적으로 활용합니다

#### 6. Extension Functions (확장 함수)
```kotlin
fun String.removeWhitespace(): String {
    return this.replace(" ", "")
}

// 사용
val result = "Hello World".removeWhitespace()  // "HelloWorld"
```​
