# RestTemplate vs Webclient

## RestTemplate

- RestTemplate 은 스프링에서 제공하는 블로킹 방식의 HttpClient이다
- 스프링 애플리케이션에서 다른 서버와 통신할 경우 유용하게 사용되고 있다
- 하지만 Spring 5 부터 Deprecated되어 스프링 MVC, 스프링 WebFlux 모두 이후 소개할 WebClient를 사용하길 권고한다

> [블로킹 방식의 RestTemplate 예시]
```kotlin
package com.fastcampus.springwebflux.webclient

import com.fastcampus.springwebflux.book.Book
import org.slf4j.LoggerFactory
import org.springframework.core.ParameterizedTypeReference
import org.springframework.http.HttpMethod
import org.springframework.web.bind.annotation.GetMapping
import org.springframework.web.bind.annotation.RestController
import org.springframework.web.client.RestTemplate

@RestController
class WebClientExample {
    val url = "http://localhost:8080/books"
    val log = LoggerFactory.getLogger(javaClass)

    @GetMapping("/books/block")
    fun getBooksBlockingWay(): List<Book> {
        log.info("Start RestTemplate")
        
        val restTemplate = RestTemplate()
        val response = restTemplate.exchange(url, HttpMethod.GET, null,
            object : ParameterizedTypeReference<List<Book>>() {})
        val result= response.body!!
        
        log.info("result : {}", result)
        log.info("Finish RestTemplate")
        
        return result
    }
}
```

> 결과
> Start RestTemplate
> result : [Book(id=1, name=코틀린 인 액션, price=30000), Book(id=2, name=HTTP 완벽 가이드 , price=40000)]
> Finish RestTemplate

- RestTemplate의 문제는 요청을 보낸 서버로 부터 응답을 받을 때까지 스레드가 블로킹되어 다른 일을 하지 못한다
- 만약 하나의 API에서 여러 서버의 응답을 받아 결합해서 처리하는 기능이 있는 경우라면 하나씩 처리하므로 응답이 느려지는 문제가 발생할 수 있다
- 이런 문제점 때문에 복수개의 응답을 처리하는 경우라면 CompletableFuture과 같은 방식을 사용해야한다

# 

## WebClient

- WebClient는 스프링에서 제공하는 리액티브 기반의 논블로킹 HttpClient이다
- 스프링 5 이후부터 RestTemplate을 대체하여 논블로킹 방식, 블로킹 방식 모두 사용이 가능하다
- WebClient를 사용하면 스레드가 응답을 기다릴 필요없이 처리할 수 있으므로 RestTemplate보다 부하를 줄일 수 있고 여러 서버의 응답을 받아서 처리하는 경우 동시에 여러 서버로 호출이 가능하 므로 빠르게 처리가 가능하다

```kotlin
@GetMapping("/books/nonblock")
fun getBooksNonBlockingWay(): Flux<Book> {
    log.info("Start WebClient")
    val flux = WebClient.create()
        .get()
        .uri(url)
        .retrieve()
        .bodyToFlux(Book::class.java)
        .map {
            log.info("result : {}", it)
            it
        }
    log.info("Finish WebClient")
    return flux
}
```

