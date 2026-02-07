#### 1. create the database schema within PostgreSQL
```SQL
DROP TABLE IF EXISTS task;
CREATE TABLE task(id SERIAL PRIMARY KEY, name VARCHAR(50), description VARCHAR(50), priority VARCHAR(50));

INSERT INTO task (name, description, priority) VALUES ('cleaning', 'Clean the house', 'Low');
INSERT INTO task (name, description, priority) VALUES ('gardening', 'Mow the lawn', 'Medium');
INSERT INTO task (name, description, priority) VALUES ('shopping', 'Buy the groceries', 'High');
INSERT INTO task (name, description, priority) VALUES ('painting', 'Paint the fence', 'Medium');
INSERT INTO task (name, description, priority) VALUES ('exercising', 'Walk the dog', 'Medium');
INSERT INTO task (name, description, priority) VALUES ('meditating', 'Contemplate the infinite', 'High');
```

#### 2. TaskRepository 접근
- 중단(Suspend): 네트워크 요청이나 DB 쿼리처럼 오래 걸리는 작업이 시작되면, 현재 실행 중인 스레드를 놓아줍니다. (다른 작업이 이 스레드를 쓸 수 있게 됨)
- 재개(Resume): 작업이 완료되면, 중단되었던 지점부터 다시 실행을 이어갑니다.
```kotlin
interface TaskRepository {
    suspend fun allTasks(): List<Task>
    suspend fun tasksByPriority(priority: Priority): List<Task>
    suspend fun taskByName(name: String): Task?
    suspend fun addTask(task: Task)
    suspend fun removeTask(name: String): Boolean
}
```

```kotlin
class FakeTaskRepository : TaskRepository {
    //...

    override suspend fun allTasks(): List<Task> = tasks

    override suspend fun tasksByPriority(priority: Priority) = tasks.filter {
        //...
    }

    override suspend fun taskByName(name: String) = tasks.find {
        //...
    }

    override suspend fun addTask(task: Task) {
        //...
    }

    override suspend fun removeTask(name: String): Boolean {
        //...
    }
}
```

#### 3. Configure a database connection
- src/main/kotlin/com/example/Database.kt 에서 설정진행

```kotlin
fun Application.configureDatabases(){
  Database.connect(
    "jdbc:postgresql://localhost:5432/ktor_tutorical_db",
    user="postgres",
    password="password"
  )
}
```

#### 4. Table과 Task 를 맵핑
- src/main/kotlin/com/example/db/mapping.kt

```kotlin
package com.example.db

import kotlinx.coroutines.Dispatchers
import org.jetbrains.exposed.dao.IntEntity
import org.jetbrains.exposed.dao.IntEntityClass
import org.jetbrains.exposed.dao.id.EntityID
import org.jetbrains.exposed.dao.id.IntIdTable

object TaskTable : IntIdTable("task") {
    val name = varchar("name", 50)
    val description = varchar("description", 50)
    val priority = varchar("priority", 50)
}

class TaskDAO(id: EntityID<Int>) : IntEntity(id) {
    companion object : IntEntityClass<TaskDAO>(TaskTable)

    var name by TaskTable.name
    var description by TaskTable.description
    var priority by TaskTable.priority
}
```

```json
dependencies {
    //...
    implementation(libs.exposed.dao)
}
```

- suspendTransaction()은 DB Transaction을 실행 (데이터베이스 작업(추가, 수정, 삭제 등)을 할 때, 여러 단계를 하나의 작업 단위로 묶는 것을 말합니다.)
- IO Dispatcher를 사용한다는 것은 "이 작업은 DB를 기다려야 하는 작업이니, 그 전용 스레드 팀에게 맡기겠다"는 의미
- Blocking Job (차단 작업): DB 쿼리는 실행 후 결과를 받을 때까지 스레드를 '점유'하고 아무것도 못 하게 만듦
    - 이걸 "스레드가 차단(Block)되었다"고 한다.
- Offload (떠넘기기): Ktor 서버의 메인 엔진이 돌아가는 스레드가 이런 무거운 DB 작업 때문에 멈추면 안됨.
    - 그래서 이 무거운 짐을 **별도의 스레드 묶음(Thread Pool)**으로 전달

```kotlin
suspend fun <T> suspendTransaction(block: Transaction.() -> T): T =
    newSuspendedTransaction(Dispatchers.IO, statement = block)

fun daoToModel(dao: TaskDAO) = Task(
    dao.name,
    dao.description,
    Priority.valueOf(dao.priority)
)
```

#### 5. PostgreSQLRepository 만들기

```kotlin
package com.example.model

import com.example.db.TaskDAO
import com.example.db.TaskTable
import com.example.db.daoToModel
import com.example.db.suspendTransaction
import org.jetbrains.exposed.sql.SqlExpressionBuilder.eq
import org.jetbrains.exposed.sql.deleteWhere

class PostgresTaskRepository : TaskRepository {
    override suspend fun allTasks(): List<Task> = suspendTransaction {
        TaskDAO.all().map(::daoToModel)
    }

    override suspend fun tasksByPriority(priority: Priority): List<Task> = suspendTransaction {
        TaskDAO
            .find { (TaskTable.priority eq priority.toString()) }
            .map(::daoToModel)
    }

    override suspend fun taskByName(name: String): Task? = suspendTransaction {
        TaskDAO
            .find { (TaskTable.name eq name) }
            .limit(1)
            .map(::daoToModel)
            .firstOrNull()
    }

    override suspend fun addTask(task: Task): Unit = suspendTransaction {
        TaskDAO.new {
            name = task.name
            description = task.description
            priority = task.priority.toString()
        }
    }

    override suspend fun removeTask(name: String): Boolean = suspendTransaction {
        val rowsDeleted = TaskTable.deleteWhere {
            TaskTable.name eq name
        }
        rowsDeleted == 1
    }
}
```
#### 6. Switch

```kotlin
//...
import com.example.model.PostgresTaskRepository

//...

fun Application.module() {
    val repository = PostgresTaskRepository()

    configureSerialization(repository)
    configureDatabases()
    configureRouting()
}
```
