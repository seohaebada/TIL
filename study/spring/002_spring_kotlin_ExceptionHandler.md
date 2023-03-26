# Spring + Kotlin 프로젝트의 공통 Exception 처리

## 에러 응답 모델 정의
```kotlin
data class ErrorResponse(
  val code: Int,
  val message: String,
)
```
- code로 어떤 에러가 발생했는지 정의한다
  - HTTP status code 만으로 모든 에러에 대한 케이스를 처리하기 어렵다
- 에러가 발생했을때 나타낼 메시지를 표시한다
  - 메시지에는 stacktrace와 같이 언어, 프레임워크 정보, DB 필드 등이 노출되면 절대 안된다

<br/>

## 1) @ExceptionHandler

> ServerException.kt
```kotlin
package com.fastcampus.issueservice.exception

sealed class ServerException(
  val code: Int,
  override val message: String,
) : RuntimeException(message)

/* @ExceptionHandler(ServerException::class) 를 탄다 */
data class NotFoundException(
  override val message: String,
) : ServerException(404, message) // sealed class 상속

data class UnauthorizedException(
  override val message: String = "인증 정보가 잘못되었습니다", // default message
) : ServerException(401, message)
``` 

> GlobalExceptionHandler.kt
```kotlin
package com.fastcampus.issueservice.exception

import com.auth0.jwt.exceptions.TokenExpiredException
import mu.KotlinLogging
import org.springframework.web.bind.annotation.ExceptionHandler
import org.springframework.web.bind.annotation.RestControllerAdvice

@RestControllerAdvice // 모든 예외 감지
class GlobalExceptionHandler {

    private val logger = KotlinLogging.logger {}

    @ExceptionHandler(ServerException::class)
    fun handleServerException(ex: ServerException) : ErrorResponse {
        logger.error { ex.message }

        return ErrorResponse(code = ex.code, message = ex.message)
    }

    @ExceptionHandler(TokenExpiredException::class)
    fun handleTokenExpiredException(ex: TokenExpiredException) : ErrorResponse {
        logger.error { ex.message }

        return ErrorResponse(code = 401, message = "Token Expired Error")
    }

    @ExceptionHandler(Exception::class)
    fun handleException(ex: Exception) : ErrorResponse {
        logger.error { ex.message }

        // ex.message 를 내려주면, 클라이언트에서 특정 서버 오류 로그를 확인할 수 있기 때문에 문구를 쓴다.
        return ErrorResponse(code = 500, message = "Internal Server Error")
    }
}
```

<br/>

## 2) ErrorWebExceptionHandler 구현

- 최상위 ServerException 을 sealed class로 정의하였다

> ServerException.kt
```kotlin
package com.fastcampus.kopring.issueservice.exception

sealed class ServerException(
  val code: Int,
  override val message: String,
) : RuntimeException(message)
```

> GlobalExceptionHandler.kt
- 애플리케이션에서 발생하는 모든 예외를 처리하는 GlobalExceptionHandler를 정의한다
- ExceptionHandler에서 정해진 ErrorResponse 객체로 응답하게 되면 항상 일관성있는 에러 처리를 할 수 있게 된다

```kotlin
package com.fastcampus.userservice.exception

import com.fasterxml.jackson.databind.ObjectMapper
import kotlinx.coroutines.reactor.awaitSingle
import kotlinx.coroutines.reactor.mono
import mu.KotlinLogging
import org.springframework.boot.web.reactive.error.ErrorWebExceptionHandler
import org.springframework.context.annotation.Configuration
import org.springframework.core.io.buffer.DataBuffer
import org.springframework.http.HttpStatus
import org.springframework.http.MediaType
import org.springframework.web.server.ServerWebExchange
import reactor.core.publisher.Mono
import reactor.kotlin.core.publisher.toMono

/**
 * ErrorWebExceptionHandler : reactive 에서 제공해주는 Handler
 */
@Configuration
class GlobalExceptionHandler(private val objectMapper: ObjectMapper) : ErrorWebExceptionHandler {

  private val logger = KotlinLogging.logger {}

  /**
   * 반환타입 Mono<Void>
   * 코루틴에서 코투린을 mono로 바꿔주는 mono 이름의 코루틴 빌더를 제공해준다. (mono : import kotlinx.coroutines.reactor.mono)
   */
  override fun handle(exchange: ServerWebExchange, ex: Throwable): Mono<Void> = mono {
    // 에러 로그 출력
    logger.error { ex.printStackTrace() }

    val errorResponse = if (ex is ServerException) { // 우리가 정의한 오류인 경우
      ErrorResponse(code = ex.code, message = ex.message)
    } else { // 아닌 경우 메시지 지정 필요
      ErrorResponse(code = 500, message = "Internal Server Error")
    }

    // with 로 감싸기
    with(exchange.response) {
      statusCode = HttpStatus.OK
      headers.contentType = MediaType.APPLICATION_JSON

      // to DataBuffer
      val dataBuffer: DataBuffer = bufferFactory().wrap(objectMapper.writeValueAsBytes(errorResponse))

      // ServerHttpResponse 의 writeWith 메서드
      // mono to coroutine
      writeWith(dataBuffer.toMono()).awaitSingle()
    }
  }
}
```
