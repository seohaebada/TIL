# 스프링 webflux - 어노테이션 컨트롤러

## 어노테이션 컨트롤러 

- 웹플럭스에서 제공하는 애노테이션 컨트롤러(Annotation Controller) 모델은 전통적인 스프링 MVC에서 사용하던 애노테이션 기반의 요청, 응답을 그대로 사용할 수 있다

```kotlin
@RestController
class HelloController {
    @GetMapping("/hello")
    fun hello() = "Hello WebFlux"
}
```

> 실행 결과
> $ curl localhost:8080/hello
> Hello WebFlux

# 

## Book 서비스 개발

### GET /books

- 저장된 전체 서적 리스트를 가져오는 API
- 컨트롤러 함수의 반환타입이 Mono, Flux인 경우 WebFlux에서 자동으로 subscribe를 호출해준다

> [Controller]
```kotlin
package com.fastcampus.springwebflux.book

import org.springframework.web.bind.annotation.GetMapping
import org.springframework.web.bind.annotation.RestController
import reactor.core.publisher.Flux

@RestController
class BookController(
    private val bookService: BookService,
) {
    @GetMapping("/books")
    fun getAll(): Flux<Book> {
        return bookService.getAll()
    }
}
```

> [Service]
```kotlin
package com.fastcampus.springwebflux.book

import org.springframework.stereotype.Service
import reactor.core.publisher.Flux
import java.util.concurrent.atomic.AtomicInteger

data class Book(val id: Int, val name: String, val price: Int)

@Service
class BookService {
    private final val nextId = AtomicInteger(0)
    
    val books =
        mutableListOf(
            Book(id = nextId.incrementAndGet(), name = "코틀린 인 액션", price = 30000),
            Book(id = nextId.incrementAndGet(), name = "HTTP 완벽 가이드 ", price = 40000),
        )
    
    fun get(id: Int): Mono<Book> {
        val books = books.find { it.id == id }
        //return Mono.justOrEmpty(books.find { it.id == id })
        return books.toMono()
    }
    
    fun getAll(): Flux<Book> {
        //return Flux.fromIterable(books)
        return books.toFlux()
    }
}
```

### GET /books/{id}

- id를 전달받아 서적을 가져오는 API

> [Controller]
```kotlin
package com.fastcampus.springwebflux.book

import org.springframework.web.bind.annotation.GetMapping
import org.springframework.web.bind.annotation.PathVariable
import org.springframework.web.bind.annotation.RestController
import reactor.core.publisher.Flux
import reactor.core.publisher.Mono

@RestController
class BookController(
    private val bookService: BookService,
) {
    @GetMapping("/books/{id}")
    fun get(@PathVariable id : Int): Mono<Book> {
        return bookService.get(id)
    }
}
```

> [Service]
```kotlin
package com.fastcampus.springwebflux.book

import org.springframework.stereotype.Service
import reactor.core.publisher.Flux
import reactor.core.publisher.Mono
import java.util.concurrent.atomic.AtomicInteger

data class Book(val id: Int, val name: String, val price: Int)

@Service
class BookService {
    fun get(id: Int): Mono<Book> {
        val books = books.find { it.id == id }
        //return Mono.justOrEmpty(books.find { it.id == id })
        return books.toMono()
    }
}
```

### POST /books

- 새로운 서적을 등록하는 API

> [Controller]
```kotlin
package com.fastcampus.springwebflux.book

import org.springframework.web.bind.annotation.GetMapping
import org.springframework.web.bind.annotation.PathVariable
import org.springframework.web.bind.annotation.PostMapping
import org.springframework.web.bind.annotation.RequestBody
import org.springframework.web.bind.annotation.RestController
import reactor.core.publisher.Flux
import reactor.core.publisher.Mono

@RestController
class BookController(
    private val bookService: BookService,
) {
    @PostMapping("/books")
    fun add(@RequestBody request : Map<String,Any>): Mono<Book> {
        return bookService.add(request)
    }
}
```

> [Service]
```kotlin
package com.fastcampus.springwebflux.book
import org.springframework.stereotype.Service
import reactor.core.publisher.Flux
import reactor.core.publisher.Mono
import reactor.kotlin.core.publisher.toFlux
import reactor.kotlin.core.publisher.toMono
import java.util.concurrent.atomic.AtomicInteger

data class Book(val id: Int, val name: String, val price: Int)

@Service
class BookService {
    private final val nextId = AtomicInteger(0)
    
    val books =
        mutableListOf(
            Book(id = nextId.incrementAndGet(), name = "코틀린 인 액션", price = 30000),
            Book(id = nextId.incrementAndGet(), name = "HTTP 완벽 가이드 ", price = 40000),
        )
    
    fun add(request: Map<String, Any>): Mono<Book> {
        return Mono.just(request)
            .map { map ->
                val book = Book(
                    id = nextId.incrementAndGet(),
                    name = map["name"].toString(),
                    price = map["price"] as Int
                )
                books.add(book)
                book
            }
    }
}
```

### DELETE /books/{id}

- 등록된 서적을 삭제하는 API

> [Controller]
```kotlin
package com.fastcampus.springwebflux.book

import org.springframework.web.bind.annotation.DeleteMapping
import org.springframework.web.bind.annotation.GetMapping
import org.springframework.web.bind.annotation.PathVariable
import org.springframework.web.bind.annotation.PostMapping
import org.springframework.web.bind.annotation.RequestBody
import org.springframework.web.bind.annotation.RestController
import reactor.core.publisher.Flux
import reactor.core.publisher.Mono

@RestController
class BookController(
    private val bookService: BookService,
) {
    @DeleteMapping("/books/{id}")
    fun delete(@PathVariable id: Int): Mono<Void> {
        return bookService.delete(id)
    }
}
```

> [Service]
```kotlin
package com.fastcampus.springwebflux.book

import org.springframework.stereotype.Service
import reactor.core.publisher.Flux
import reactor.core.publisher.Mono
import reactor.kotlin.core.publisher.toFlux
import reactor.kotlin.core.publisher.toMono
import java.util.concurrent.atomic.AtomicInteger

data class Book(val id: Int, val name: String, val price: Int)

@Service
class BookService {
    private final val nextId = AtomicInteger(0)
    
    val books =
        mutableListOf(
            Book(id = nextId.incrementAndGet(), name = "코틀린 인 액션", price = 30000),
            Book(id = nextId.incrementAndGet(), name = "HTTP 완벽 가이드 ", price = 40000),
        )
    
    fun delete(id: Int): Mono<Void> {
        return Mono.justOrEmpty(books.find { it.id == id })
            .map { books.remove(it) }
            .then()
    }
}
```