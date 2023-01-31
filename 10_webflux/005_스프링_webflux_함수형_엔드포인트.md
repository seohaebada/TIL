# 스프링 webflux - 함수형 엔드포인트

## 함수형 엔드포인트

- 스프링 WebFlux는 클라이언트의 요청을 라우팅하고 처리할 수 있는 람다 기반 프로그래밍 모델인 함수형 엔드포인트(Functional Endpoints) 를 제공한다
- 함수형 엔드포인트는 요청을 분석해 핸들러로 라우팅하는 라우터 함수(RouterFunction) 과 요청 객체를 전달 받아 응답을 제공하는 핸들러 함수(HandlerFunction) 으로 이뤄져 있다

<br/>

## 라우터 함수

- 라우터 함수는 클라이언트로부터 전달받은 요청을 해석하고 그에 맞는 핸들러로 전달하는 역할
- 라우터 함수는 `@Configuration`에 `RouterFunction`을 반환하는 빈(Bean)으로 등록할 수 있으며 빈의 이름을 다르게하여 여러개의 라우터 함수를 등록할 수 있다

- 여러개의 라우터 등록 
```kotlin
package com.fastcampus.springwebflux
import org.springframework.context.annotation.Bean
import org.springframework.context.annotation.Configuration
import org.springframework.web.reactive.function.server.RouterFunction
import org.springframework.web.reactive.function.server.RouterFunctions.route
import org.springframework.web.reactive.function.server.ServerResponse

@Configuration
class Router {
    @Bean
    fun helloRouter(handler: HelloHandler): RouterFunction<ServerResponse> =
        route()
            .GET("/", handler::sayHello)
            .build()
    @Bean
    fun userRouter(handler: UserHandler): RouterFunction<ServerResponse> =
        route()
            .GET("/users/{id}", handler::getUser)
            .GET("/users", handler::getAll)
            .build()
}
```

- 라우터 함수는 이러한 URI 패턴 매칭에 따른 분배 역할을 위해 RouterFunctions.route() 라는 유용한 빌더를 제공하고 있음
- RouterFunctions.route()를 사용하면 HTTP 요청 메서드 GET, POST, PUT, DELETE 등에 대한 매핑을 위한 편리한 함수를 제공하므로 목적에 맞게 URI 패턴을 등록하여 라우팅 룰을 생성할 수 있다

<br/>

## 중첩 라우터

- 중복된 경로를 그룹화하고 싶은 경우 중첩 라우터(Nested Router) 를 사용할 수 있다
- 중첩 라우터를 사용해 그룹화하면 코드의 중복을 제거하고 가독성 좋은 라우터 함수를 작성할 수 있다

```kotlin
@Bean
fun userRouter(handler: UserHandler): RouterFunction<ServerResponse> =
    router {
        "/users".nest {
            GET("/users/{id}", handler::getUser)
            GET("/users", handler::getAll)
        }
    }
```

<br/>

## 핸들러 함수

- 핸들러 함수는 라우터 함수로부터 전달받은 요청 객체인 ServerRequest 를 이용해 로직을 처리한 후 응답 객체인 ServerResponse 를 생성한 뒤 반환한다
- ServerRequest, ServerResponse는 불변 객체로 설계되었다
- 기본적으로 리액터의 Publisher인 Mono 또는 Flux로 응답 본문을 작성한다

```kotlin
package com.fastcampus.springwebflux

import org.springframework.stereotype.Component
import org.springframework.web.reactive.function.server.ServerRequest
import org.springframework.web.reactive.function.server.ServerResponse
import reactor.core.publisher.Mono

class User(val id: Long, val email: String)

@Component
class UserHandler {
    val users = listOf(
        User(id = 1, email = "user1@gmail.com"),
        User(id = 2, email = "user2@gmail.com")
    )
    
    fun getUser(request: ServerRequest): Mono<ServerResponse> =
        users.find { request.pathVariable("id").toLong() == it.id }
            ?.let {
                ServerResponse.ok().bodyValue(it)
            } ?: ServerResponse.notFound().build()
    
    fun getAll(request: ServerRequest): Mono<ServerResponse> =
        ServerResponse.ok().bodyValue(users)
}
```

