#### Official Docs
https://ktor.io/docs/server-create-a-new-project.html

#### Official Docs부터 읽는 이유?
- Ktor이라는 Framework를 하나도 모르겠다.
- 뭔가 기본 틀은 나와있으니 이걸 보면 눈에 들어올것 같다.
    - 폴더구조, 환경설정관리, 제안하는 Code 구조

## 읽으면서 익숙하지 않은 것들 정리

#### 1. Configuration files, and other kinds of content, live within the src/main/resources folder.
<img width="294" height="187" alt="image" src="https://github.com/user-attachments/assets/f34522f3-ac4d-4d3f-a55d-0663d6478c9d" />

 - 설정 파일이나 static 파일들은 resources에 위치시킴.
 - 설정 파일을 resources에 두는군...?

#### 2. Ktor 실행 시, Port 바꾸기는 환경설정 파일로 관리도 가능하네.
- 완전 비슷하진 않겠지만 env 파일이랑 비슷할 것 같다.
- 암호화 걸고, Docker Image / 공유 금지

##### yaml 버전
```yaml
ktor:
    application:
        modules:
            - com.example.ApplicationKt.module
    deployment:
        port: 8080
```
##### hocon 버전
```hocon
ktor {
    deployment {
        port = 8080
        port = ${?PORT}
    }
    application {
        modules = [ com.example.ApplicationKt.module ]
    }
}
```

#### 3. Code Level에서 Port도 바꿈 (src/main/kotlin/Application.kt 위치에 있음)
- embeddedServer() 함수는 port를 바꿀 수 있다고 나와있음.

```kotlin
fun main() {
    embeddedServer(
        Netty,
        port = 8080, // This is the port to which Ktor is listening
        host = "0.0.0.0",
        module = Application::module
    ).start(wait = true)
}

fun Application.module() {
    configureRouting()
}
```

#### 4. Http Endpoint 추가 (src/main/kotlin/Application.kt 위치)
- Routing.kt 로 대체하는 방법도 있다고 함.
- 아마. Rounting.kt로 분리하는게 구분도 쉽고 해서, Routing.kt로 많이 사용할 것 같음.

```kotlin
fun Application.configureRouting() {
    routing {
        get("/") {
            call.respondText("Hello World!")
        }
        get("/test1") {
            val text = "<h1>Hello From Ktor</h1>"
            val type = ContentType.parse("text/html")
            call.respondText(text, type)
        }
    }
}
```

#### 5. static content 설정하기?
```kotlin
fun Application.configureRouting() {
    routing {
        staticResources("/content", "mycontent")
        // ...
    }
}
```
1. staticResources()를 호출하면 애플리케이션에서 HTML, JavaScript 파일과 같은 표준 웹사이트 콘텐츠를 제공할 수 있음
- 서버 관점에서는 정적(static) 콘텐츠
2. /content URL은 해당 콘텐츠를 가져오기 위해 사용되는 경로를 지정합니다.
3. mycontent 경로는 정적 콘텐츠가 위치할 폴더의 이름입니다.
- Ktor는 이 폴더를 resources 디렉터리 안에서 찾습니다.

-> Django로 치면 Template이나 그런거네.

```
<html lang="en">
    <head>
        <meta charset="UTF-8" />
        <title>My sample</title>
    </head>
    <body>
        <h1>This page is built with:</h1>
        <ol>
            <li>Ktor</li>
            <li>Kotlin</li>
            <li>HTML</li>
        </ol>
    </body>
</html>
```
src/main/resources/mycontent/sample.html 하면 http://0.0.0.0:9292/content/sample.html 에서 정적페이지 던지네

#### 6. Error Handler 등록!!!
> You can handle errors in your Ktor application using the StatusPages plugin.

Routing.kt에서 추가함
```kotlin
fun Application.configureRouting() {
    install(StatusPages) {
        exception<IllegalStateException> { call, cause ->
            call.respondText("App in illegal state as ${cause.message}")
        }
    }
    routing {
        // ...
    }
}
```
These lines install the StatusPages plugin and specify what actions to take when an exception of type IllegalStateException is thrown.

```kotlin
fun Application.configureRouting() {
    install(StatusPages) {
        exception<IllegalStateException> { call, cause ->
            call.respondText("App in illegal state as ${cause.message}")
        }
    }
    routing {
        // ...

        get("/error-test") {
            throw IllegalStateException("Too Busy")
        }
    }
}
```
✅ exceptions
예외(Exception) 타입 기준으로 응답을 설정, 서버 코드에서 Throwable 이 발생했을 때 처리
보통 500, 403 같은 오류 응답을 여기서 정의
```kotlin
install(StatusPages) {
    exception<Throwable> { call, cause ->
        call.respondText(
            text = "500: $cause",
            status = HttpStatusCode.InternalServerError
        )
    }
}
// 서버 처리 중 어떤 예외든 발생하면 "500: 예외내용" 을 응답으로 반환
```
```kotlin
install(StatusPages) {
    exception<Throwable> { call, cause ->
        if (cause is AuthorizationException) {
            call.respondText(
                text = "403: $cause",
                status = HttpStatusCode.Forbidden
            )
        } else {
            call.respondText(
                text = "500: $cause",
                status = HttpStatusCode.InternalServerError
            )
        }
    }
}
// 특정 예외만 따로 처리
```
✅ status

HTTP 상태 코드 기준으로 응답을 설정, 예외가 없어도, 특정 상태 코드가 반환될 때 동작
```kotlin
install(StatusPages) {
    status(HttpStatusCode.NotFound) { call, status ->
        call.respondText(
            text = "404: Page Not Found",
            status = status
        )
    }
}
```

✅ statusFile

HTTP 상태 코드에 따라 정적 파일(HTML 등)을 응답 파일은 classpath (src/main/resources) 에 있어야 함
에러 페이지를 HTML 파일로 보여주고 싶을 때 사용

StatusPages는 서버 오류를 처리하게 해줌.
1. 예외가 던져졌을 때 -> 커스텀 메시지 or 페이지
2. 특정 HTTP 상태(404, 500) -> 별도의 처리

StatusFile이란?
- statusFile은 특정 HTTP 상태 코드에 대해 미리 만들어둔 HTML 파일을 응답으로 보내는 기능
- src/main/resources/error401.html 에 위치하면 보여줌
```kotlin
install(StatusPages) {
    // 401 상태 → resources/error401.html 반환
    // 402 상태 → resources/error402.html 반환
    statusFile(
        HttpStatusCode.Unauthorized,
        HttpStatusCode.PaymentRequired,
        filePattern = "error#.html"
    )
}
```

