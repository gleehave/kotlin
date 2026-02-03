#### 1. static HTML 뿌리기
- Routing.kt에 구현하면 즉각적으로 Render 가능
- 근데.. call.respondText는 뭐지?.. 뭔가 Django로 치면 request 객체같은건가..?
  - io.ktor.server.application.ApplicationCall
  - 즉, Request + Respond가 합쳐진 개념
  - suspend PipelineContext<Unit, ApplicationCall>.() -> Unit
  - 내부에서 리시버(this)가 자동으로 제공됨.
 
 
```kotlin
import io.ktor.http.ContentType  // 추가

fun Application.configureRouting() {
    routing {
        get("/tasks") {
            call.respondText(
                contentType = ContentType.parse("text/html"),
                    text = """
                <h3>TODO:</h3>
                <ol>
                    <li>A table of all the tasks</li>
                    <li>A form to submit new tasks</li>
                </ol>
                """.trimIndent()
            )
        }
    }
}
```

#### 2. Model Type 만들기
- Database 테이블과 매칭되는건 아니고.. DTO나 DAO 개념인듯.
- src/main/kotlin/com/example/model/Task.kt

```kotlin
enum class Priority{
  Low, Medium, High, Vital
}

data class Task(
  val name: String,
  val description: String,
  val priority: Priority  // enum 클래스를 나타내네. 여튼 클래스니까 가능
)
```

- 그런데.. Task라는 클래스에 확장함수를 구현하는건데.. `Task::taskAsRow` 는 뭐냐..?
    - Task 객체를 받아서 taskAsRow() 를 호출해주는 함수 자체를 넘긴다
    - 즉, 함수를 값 처럼 전달한다.
```
fun Task.taskAsRow() = """
    <tr>
        <td>$name</td><td>$description</td><td>$priority</td>
    </tr>
    """.trimIndent()

fun List<Task>.tasksAsTable() = this.joinToString(
    prefix = "<table rules=\"all\">",
    postfix = "</table>",
    separator = "\n",
    transform = Task::taskAsRow
)
```

- model/TaskRepository.kt 만들기
```kotlin
val tasks = mutableListOf(
    Task("cleaning", "Clean the house", Priority.Low),
    Task("gardening", "Mow the lawn", Priority.Medium),
    Task("shopping", "Buy the groceries", Priority.High),
    Task("painting", "Paint the fence", Priority.Medium)
)

fun Application.configureRouting(){
  routing{
    get("/tasks"){
      call.respondText(
        contentType = ContentType.parse("text/html")
        text = tasks.tasksAsTable()
      )
    }
  }
}
```

#### 3. model 부분을 Refactor

```kotlin
package model

object TaskRepository {
    private val tasks = mutableListOf(
        Task("cleaning", "Clean the house", Priority.Low),
        Task("gardening", "Mow the lawn", Priority.Medium),
        Task("shopping", "Buy the groceries", Priority.High),
        Task("painting", "Paint the fence", Priority.Medium)
    )

    fun allTasks(): List<Task> = tasks

    fun tasksByPriority(priority: Priority) = tasks.filter {
        it.priority == priority
    }

    fun taskByName(name: String) = tasks.find {
        it.name.equals(name, ignoreCase = true)
    }

    fun addTask(task: Task) {
        if(taskByName(task.name) != null) {
            throw IllegalStateException("Cannot duplicate task names!")
        }
        tasks.add(task)
    }
}
```

object? 는 뭔데?
- kotlin의 싱글턴(Singleton) 선언 키워드
- 클래스를 정의하면서 동시에 인스턴스를 딱 하나만 만든다
    - 오.. 이걸 그냥 제공한다고?
 
| 구분 | 의미 |
|------|------|
| `object` | 전역 싱글턴 |
| `class` | 여러 인스턴스 생성 가능 |
| `companion object` | 클래스에 종속된 싱글턴 |


tasks.filter / find/ add 가 가능한건 확장함수 덕인가?
- Kotlin 표준 라이브러리의 확장 함수
```
fun <T> Iterable<T>.filter(...)
fun <T> Iterable<T>.find(...)

filter(tasks) { ... }
find(tasks) { ... }

tasks.filter { ... }
tasks.find { ... }
```

- Routing.kt로 돌아와서 TaskRepository로 구현
```kotlin
fun Application.configureRouting() {
    routing {
        get("/tasks") {
            val tasks = TaskRepository.allTasks()
            call.respondText(
                contentType = ContentType.parse("text/html"),
                text = tasks.tasksAsTable()
            )
        }
    }
}
```

#### 4. URL 파라미터 
##### 4-1. {priority?} 
/tasks/byPriority/High
/tasks/byPriority/
- 2가지 형태가 가능함. 다만 아래 코드에서 null이면 400 Bad Request로 반환

##### 4-2. return@get은 뭐야?
| 상황             | 의미                |
| -------------- | ----------------- |
| `return`       | **가장 바깥 함수에서 반환** |
| `return@label` | **특정 람다에서만 빠져나감** |

- return@get 는 "여기 get{} 블록만 종료한다." 의 의미

##### 4-3. Priority.valueOf()
- Priority.valueOf("High")   // Priority.High
- Priority.valueOf("LOW")    // ❌ IllegalArgumentException

```kotlin
routing {
    get("/tasks") { ... }

    //add the following route
    get("/tasks/byPriority/{priority?}") {
        val priorityAsText = call.parameters["priority"]
        if (priorityAsText == null) {
            call.respond(HttpStatusCode.BadRequest)
            return@get
        }

        try {
            val priority = Priority.valueOf(priorityAsText)
            val tasks = TaskRepository.tasksByPriority(priority)

            if (tasks.isEmpty()) {
                call.respond(HttpStatusCode.NotFound)
                return@get
            }

            call.respondText(
                contentType = ContentType.parse("text/html"),
                text = tasks.tasksAsTable()
            )
        } catch(ex: IllegalArgumentException) {
            call.respond(HttpStatusCode.BadRequest)
        }
    }
}
```

#### 5. POST 요청 (with HTML Form)
- src/main/resources/task-ui/task-form.kt

##### 5-1. form
```kotlin
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Adding a new task</title>
</head>
<body>
<h1>Adding a new task</h1>
<form method="post" action="/tasks">
    <div>
        <label for="name">Name: </label>
        <input type="text" id="name" name="name" size="10">
    </div>
    <div>
        <label for="description">Description: </label>
        <input type="text" id="description" name="description" size="20">
    </div>
    <div>
        <label for="priority">Priority: </label>
        <select id="priority" name="priority">
            <option name="Low">Low</option>
            <option name="Medium">Medium</option>
            <option name="High">High</option>
            <option name="Vital">Vital</option>
        </select>
    </div>
    <input type="submit">
</form>
</body>
</html>
```

##### 5-2. routing에 추가
```kotlin
import io.ktor.server.http.content.staticResources

fun Application.configureRouting() {
    routing {
        //add the following line
        staticResources("/task-ui", "task-ui")

        get("/tasks") { ... }

        get("/tasks/byPriority/{priority?}") { … }

        //add the following route
        post("/tasks") {
            val formContent = call.receiveParameters()

            val params = Triple(
                formContent["name"] ?: "",
                formContent["description"] ?: "",
                formContent["priority"] ?: ""
            )

            if (params.toList().any { it.isEmpty() }) {
                call.respond(HttpStatusCode.BadRequest)
                return@post
            }

            try {
                val priority = Priority.valueOf(params.third)
                TaskRepository.addTask(
                    Task(
                        params.first,
                        params.second,
                        priority
                    )
                )

                call.respond(HttpStatusCode.NoContent)
            } catch (ex: IllegalArgumentException) {
                call.respond(HttpStatusCode.BadRequest)
            } catch (ex: IllegalStateException) {
                call.respond(HttpStatusCode.BadRequest)
            }
        }
    }
}
```
































