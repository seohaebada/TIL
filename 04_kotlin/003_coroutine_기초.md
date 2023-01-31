# 코루틴 기초

## runBlocking

- runBlocking은 코루틴을 생성하는 코루틴 빌더 이다
- runBlocking으로 감싼 코드는 코루틴 내부의 코드가 수행이 끝날때 까지 스레드가 블로킹된다

```kotlin
import kotlinx.coroutines.*

fun main() {
    runBlocking {
        println("Hello")
    }
    println("World")
}

// Hello
// World
```

- 실행옵션에 -Dkotlinx.coroutines.debug 을 붙여주면 코루틴에서 수행되는 스레드는 이름 뒤에 @coroutine#1 이 붙어있는 것을 볼 수 있다

```kotlin
import kotlinx.coroutines.*

fun main() {
    runBlocking {
        println("Hello")
        println(Thread.currentThread().name)
    }
    println("World")
    println(Thread.currentThread().name)
}

// Hello
// main @coroutine#1
// World
// main
```

- 일반적으로 코루틴은 스레드를 차단하지 않고 사용해야하므로 runBlocking을 사용하는 것은 좋지 않지만 꼭 사용해야하는 경우가 있다
  - 코루틴을 지원하지 않는 경우 예) 테스트코드, 스프링 배치 등

# 

## launch

- launch 는 스레드 차단 없이 새 코루틴을 시작하고 결과로 job을 반환하는 코루틴 빌더이다
- launch는 결과를 만들어내지 않는 비동기 작업에 적합하기 때문에 인자로 Unit을 반환하는 람다를 인자로 받는다

```kotlin
fun main() = runBlocking<Unit> {
    launch {
        delay(500L)
        println("World!")
    }
    println("Hello")
}

// Hello
// World
```
- delay() 함수는 코루틴 라이브러리에 정의된 일시 중단 함수이며, Thread.sleep() 과 유사하지만 현재 스레드를 차단하지 않고 일시 중단 시킨다. 
- 이때 일시 중단 된 스레드는 코루틴 내에서 다른 일시 중단 함수를 수행한다

### launch를 사용해서 여러개의 작업을 병렬적으로 수행할 수 있다

```kotlin
import kotlinx.coroutines.*
import kotlin.system.measureTimeMillis

fun main() {
    runBlocking {
        launch {
            val timeMillis = measureTimeMillis {
                delay(150)
            }
            println("async task-1 $timeMillis ms")
        }
        
        launch {
            val timeMillis = measureTimeMillis {
                delay(100)
            }
            println("async task-2 $timeMillis ms")
        }
    }
}
```

### launch가 반환하는 Job 을 사용해 현재 코루틴의 상태를 확인하거나 실행 또는 취소도 가능하다

```kotlin
import kotlinx.coroutines.*
import kotlin.system.measureTimeMillis

fun main() = runBlocking<Unit> {
    val job1: Job = launch {
        val timeMillis = measureTimeMillis {
            delay(150)
        }
        println("async task-1 $timeMillis ms")
    }
    
    job1.cancel() // 취소
    
    val job2: Job = launch(start = CoroutineStart.LAZY) {
        val timeMillis = measureTimeMillis {
            delay(100)
        }
        println("async task-2 $timeMillis ms")
    }
    
    println("start task-2")
    
    job2.start()
}
```

- job1.cancel() 을 호출해 코루틴을 취소할 수 있다
- launch(start = CoroutineStart.LAZY) 를 사용해서 start 함수를 호출하는 시점에 코루틴을 동작시킬 수 있다
  - start 함수를 주석처리하면 launch가 동작하지 않는다

# 

## async

- async 빌더는 비동기 작업을 통해 결과를 만들어내는 경우에 적합하다

```kotlin
import kotlinx.coroutines.*

fun sum(a: Int, b: Int) = a + b

fun main()= runBlocking<Unit> {
    val result1: Deferred<Int> = async {
        delay(100)
        sum(1, 3)
    }
    
    println("result1 : ${result1.await()}")
    
    val result2: Deferred<Int> = async {
        delay(100)
        delay(100)
        sum(2, 5)
    }
    
    println("result2 : ${result2.await()}")
}
```

- async는 비동기 작업의 결과로 Deferred 라는 특별한 인스턴스를 반환하는데 await 이라는 함수를 통해 async로 수행한 비동기 작업의 결과를 받아올 수 있다
- 자바 스크립트나 다른 언어의 async-await은 키워드인 경우가 보통이지만 코틀린의 코루틴은 async-await이 함수인 점이 차이점이다

# 

## suspend 함수

- suspend 함수는 코루틴의 핵심 요소로써 일시 중단이 가능한 함수를 말한다

```kotlin
suspend fun doSomething() {
}
```

- suspend는 키워드이다
- suspend 함수는 일반 함수를 마음껏 호출할 수 있지만 일반 함수에선 suspend 함수를 호출할 수 없다

```kotlin
fun main() {
    doSomething() // 컴파일 에러
}

fun printHello() = println("hello")
suspend fun doSomething() {
    printHello()
}
```

### 호출하는 함수에 suspend 키워드를 붙여주면 된다

```kotlin
suspend fun main() {
    doSomething()
}

fun printHello() = println("hello")

suspend fun doSomething() {
    printHello()
}
```

### suspend 함수에서 앞서 학습한 async, launch와 같은 코루틴 빌더를 사용하려면 코루틴 스코프(coroutineScope) 를 사용한다

- coroutineScope를 사용하면 runBlocking과 다르게 현재 스레드가 차단되지 않고 코루틴이 동작한다

```kotlin
import kotlinx.coroutines.coroutineScope
import kotlinx.coroutines.delay
import kotlinx.coroutines.launch

suspend fun main() {
    doSomething()
}

fun printHello() = println("hello")

suspend fun doSomething() = coroutineScope {
    launch {
        delay(200)
        println("world!")
    }
    
    launch {
        printHello()
    }
}
```

# 

## Flow

- Flow 는 코루틴에서 리액티브 프로그래밍 스타일로 작성할 수 있도록 만들어진 API이다
- 코루틴의 suspend 함수는 단일 값을 비동기로 반환하지만 Flow를 사용하면 여러개의 값을 반환할 수 있다

```kotlin
import kotlinx.coroutines.delay
import kotlinx.coroutines.flow.Flow
import kotlinx.coroutines.flow.flow
import kotlinx.coroutines.runBlocking

fun main() = runBlocking<Unit> {
    val flow = simple()
    flow.collect { value -> println(value) }
}

fun simple(): Flow<Int> = flow {
    println("Flow started")
    for (i in 1..3) {
        delay(100)
        emit(i)
    }
}

// Flow started
// 1
// 2
// 3
```

- 리액티브 스트림과 같이 Terminal Operator(최종 연산자) 인 collect 를 호출하지 않으면 아무런 일도 일어나지 않는다



