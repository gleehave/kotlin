#### 1. Plugin 추가
- Kotlinx.serialization을 추가

#### 2. model 만들기
- src/main/kotlin/com/example/model/Task.kt
```kotlin
package com.example.model

import kotlinx.serialization.Serializable

enum class Priority{
  Low, Medium, High, Vital
}

@Serializable
data class Task(
  val name: String,
  val description: String,
  val priority: Priority
)
```

#### 3. Routing에서 사용해보기

```kotlin
package com.example

import com.example.model.*
import io.ktor.server.application.*
import io.ktor.server.http.content.*
import io.ktor.server.response.*
import io.ktor.server.routing.*

fun Application.configureRouting() {
    routing {
        staticResources("static", "static")

        get("/tasks") {
            call.respond(
                listOf(
                    Task("cleaning", "Clean the house", Priority.Low),
                    Task("gardening", "Mow the lawn", Priority.Medium),
                    Task("shopping", "Buy the groceries", Priority.High),
                    Task("painting", "Paint the fence", Priority.Medium)
                )
            )
        }
    }
}
```

#### 4. Content Negotiation 
> This plugin looks at the types of content that the client can render and matches these against the content types that the current service can provide. Hence, the term Content Negotiation.
> In HTTP the client signals which content types it can render through the Accept header. The value of this header is one or more content types. In the case above you can examine the value of this header by using the development tools built into your browser.

Browser에서 수신(Accept)할 수 있는 content 타입을 매칭
- src/main/kotlin/com/example/Serialization.kt
```kotlin
install(ContentNegotiation){
  json()
}
```
- 이 코드는 ContentNegotiation 플러그인을 설치하고, 동시에 kotlinx.serialization 플러그인을 설정합니다. 
- 이를 통해 클라이언트가 요청을 보낼 때, 서버는 객체를 JSON 형식으로 직렬화하여 응답할 수 있습니다.
- 브라우저에서 보낸 요청의 경우, Content Negotiation 플러그인은 JSON만 반환할 수 있다는 것을 알고 있으며, 브라우저는 전달받은 어떤 콘텐츠든 표시하려고 시도합니다. 따라서 이 요청은 문제없이 성공하게 됩니다.

#### 5. Repository
```kotlin
package com.example.model

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
        if (taskByName(task.name) != null) {
            throw IllegalStateException("Cannot duplicate task names!")
        }
        tasks.add(task)
    }
}
```

- GET 요청
```kotlin
package com.example

import com.example.model.Priority
import com.example.model.TaskRepository
import io.ktor.http.*
import io.ktor.server.application.*
import io.ktor.server.http.content.*
import io.ktor.server.response.*
import io.ktor.server.routing.*

fun Application.configureRouting() {
    routing {
        staticResources("static", "static")

        //updated implementation
        route("/tasks") {
            get {
                val tasks = TaskRepository.allTasks()
                call.respond(tasks)
            }

            get("/byName/{taskName}") {
                val name = call.parameters["taskName"]
                if (name == null) {
                    call.respond(HttpStatusCode.BadRequest)
                    return@get
                }

                val task = TaskRepository.taskByName(name)
                if (task == null) {
                    call.respond(HttpStatusCode.NotFound)
                    return@get
                }
                call.respond(task)
            }
            get("/byPriority/{priority}") {
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
                    call.respond(tasks)
                } catch (ex: IllegalArgumentException) {
                    call.respond(HttpStatusCode.BadRequest)
                }
            }
        }
    }
}
```

- POST 요청
```kotlin
//...

fun Application.configureRouting() {
    routing {
        //...

        route("/tasks") {
            //...

            //add the following new route
            post {
                try {
                    val task = call.receive<Task>()
                    TaskRepository.addTask(task)
                    call.respond(HttpStatusCode.Created)
                } catch (ex: IllegalStateException) {
                    call.respond(HttpStatusCode.BadRequest)
                } catch (ex: SerializationException) {
                    call.respond(HttpStatusCode.BadRequest)
                }
            }
        }
    }
}
```

- Request Body 구성해서 전송하면 POST 요청 성공
```
###

POST http://0.0.0.0:8080/tasks
Content-Type: application/json

{
    "name": "cooking",
    "description": "Cook the dinner",
    "priority": "High"
}
```














