# Coroutine 공식문서 번역기 - 1 Coroutines Basics

* 다음 문서는 KotlinLang에서 제공되는 Coroutines 공식문서를 번역(의역)한 내용입니다.
* 최대한 번역체를 줄이고 Reader-Friendly하게 작성하려고 노력을 하였습니다.

## 코루틴, 맨땅에 헤딩하기

아래 주어진 코드를 IntelliJ 혹은 [Kotlin Playground](https://play.kotlinlang.org/)에서 실행시켜봅시다

```kotlin
import kotlinx.coroutines.*

fun main() {
    GlobalScope.launch { // launch a new coroutine in background and continue
        delay(1000L) // non-blocking delay for 1 second (default time unit is ms)
        println("World!") // print after delay
    }
    println("Hello,") // main thread continues while coroutine is delayed
    Thread.sleep(2000L) // block main thread for 2 seconds to keep JVM alive
}
```

코드를 실행하면 다음과 같은 결과가 도출될 것입니다.

```
Hello,
World!
```

어떻게 이런 결과가 나온 것일까요? 코루틴은 **경량화된 스레드(Light-Weight Threads)**입니다. 코루틴은 *CoroutineScope*이라는 *Context*에서 *launch*와 같은 **코루틴 빌더**릍 통해 코루틴을 설치할 수 있습니다. 

위의 코드에서는 *GlobalScope*이라는 Context에서 코루틴을 설치했습니다. 이는 "이 코루틴의 생명주기는 어플리케이션의 전체 생명주기에 종속된다"라는 의미입니다.

GlobalScope.launch와 delay 함수는 우리가 기존에 사용했었던 스레드 함수(thread { } )와 Sleep 함수(Thread.sleep())로 각각 대체가 가능합니다. 그렇다면 다음과 같은 코드가 정상적으로 작동할 수 있을까요?

```kotlin
import kotlin.concurrent.thread
import kotlinx.coroutines.*

fun main() {
    thread { 
        delay(1000L) 
        println("World!") 
    }
    println("Hello,") 
    Thread.sleep(2000L) 
}
```

아...아쉽게도 그러지 않습니다. delay함수는 suspend 함수로 정의되어서 스레드를 블락하지 않습니다. 하지만 delay함수 자체가 코루틴을 멈추기에(suspend 하기에) delay는 **코루틴 내부에서 사용**되어야 합니다.

## 그러면 코루틴으로 스레드를 아예 막을 수는 없는건가요?

위의 예시는 suspend 함수인 delay와 스레드를 아예 블락하는 Thread.sleep 함수가 혼재되어있습니다. 이와 같이 코드를 작성한다면 어떤 부분이 스레드를 막는 부분인지 아닌지를 쉽게 판단할 수 없게 됩니다. 이에 코루틴에서는 코루틴으로 스레드를 블락시킬 수 있는 runBlocking 코루틴 빌더를 대안으로 제시합니다.

```kotlin
import kotlinx.coroutines.*

fun main() { 
    GlobalScope.launch { 
        delay(1000L)
        println("World!")
    }
    println("Hello,") 
    runBlocking {     // 이 코루틴 빌더가 스레드를 블락합니다.
        delay(2000L)  
    } 
}
```

결과는 위와 같지만, 이번에 작성한 코드는 위와는 달리 스레드를 블락하지 않는 함수들만 사용해서 작성되었습니다. runBlocking은 내부의 함수가 끝날때까지 메인 스레드를 블락합니다. 위의 코드는 runBlocking을 main 함수 전체를 감싸는 형식으로 작성할 수도 있습니다.

```kotlin
import kotlinx.coroutines.*

fun main() = runBlocking<Unit> { 
    GlobalScope.launch { 
        delay(1000L)
        println("World!")
    }
    println("Hello,") 
    runBlocking {     // 이 코루틴 빌더가 스레드를 블락합니다.
        delay(2000L)  
    } 
}
```

새롭게 작성된 방식에서는, runBlocking이 Unit(코틀린에서는 main함수가 Unit을 반환하죠? ㅎㅁㅎ)을 반환하는 최상위 단의 코루틴이 됩니다. 이런 방식은 suspend 함수를 Unit 테스트를 작성할 때도 사용됩니다.

```kotlin
class MyTest {
    @Test
    fun testMySuspendingFunction() = runBlocking<Unit> {
        // suspend 함수를 여기서 테스트 합니다.
    }
}
```

## 근데 무조건 메인 스레드를 막으면서 launch를 해야하나요?

다른 코루틴이 작업하는 동안 delay를 시키는 것은 그다지 좋지 못한 방식입니다. 따라서 이번에는 새로운 방식으로 launch 된 코루틴의 작업을 대기해보고자 합니다.

```kotlin
import kotlinx.coroutines.*

fun main() = runBlocking {
    val job = GlobalScope.launch { // 코루틴을 launch 하면 job 객체가 이를 참조합니다.
        delay(1000L)
        println("World!")
    }
    println("Hello,")
    job.join() // 코루틴의 작업이 끝날때까지 기다립니다.   
}
```

Job 객체는 launch 함수의 반환형입니다. 이를 활용하여 메인 코루틴(runBlocking으로 빌드된 코루틴)은 백그라운드 job(GlobalScope.launch로 빌드된 코루틴)을 위하여 delay를 할 필요가 없습니다.

## 구조적 동시성

여태까지 우리는 GlobalScope.launch를 이용하여 최상위 코루틴(앱 전체에서 살아남는 코루틴)을 만드는 것을 배웠습니다. 이런 코루틴들은 아무리 light-weight 하더라도 메모리 자원들을 프로그램이 돌아가는 동안 계속 사용하며 이런 코루틴 객체의 참조를 잃는 순간 메모리 릭이 일어날 수 있습니다. 그렇다면 이런 코루들 객체들의 reference를 다 가지고 있으면서 코루틴을 사용해야하나요? 이는 코루틴 사용에 전형적인 안티패턴이라 볼 수 있습니다.

이제 우리들은 **구조적 동시성**이라는 아주 놀라운 방법을 사용할 수 있습니다. GlobalScope에서 코루틴을 launch하기보다 우리가 수행하는 특정 scope에서 코루틴을 빌드할 수 있습니다.

아래의 예시에서 메인함수는 runBlocking 코루틴을 사용합니다. 모든 코루틴 빌더(runBlocking도 코루틴 빌더죠)들은 고유의 코드 블럭안에서 또다른 CoroutineScope를 추가할 수 있습니다. 이런 방식을 사용하면 join을 명시적으로 사용 안해도 코루틴을 launch 할 수 있습니다. 왜냐면 이 방식에서는 상위 코루틴이 내부 코루틴이 완료될때까지 종료가 안되기 때문이죠.

```kotlin
import kotlinx.coroutines.*

fun main() = runBlocking { // this: CoroutineScope
    launch { 
        delay(1000L)
        println("World!")
    }
    println("Hello,")
}
```

## 스코프 빌더

코루틴 scope은 또 다른 빌더를 제공합니다. 스코프 내에서는 또 다른 스코프를  만들 수 있는 coroutineScope 빌더를 선언할 수 있습니다. 모든 자식 코루틴이 종료될 때까지 종료되지 않는 코루틴 스코프를 만들 수 있습니다.

이런 점들로 볼 때 runBlocking과 coroutineScope는 비슷해 보이지만, 이 둘의 다른 점은 다른 작업이 진행되는 동안  runBlocking은 현재 스레드를 블락하는 것이고, coroutineScope는 현재 스레드가 기다린다는 것이다. 이런 차이점 때문에 coroutineScope는 suspend 함수이고 runBlocking은 일반 함수입니다.

```kotlin
import kotlinx.coroutines.*

fun main() = runBlocking { // this: CoroutineScope
    launch { 
        delay(200L)
        println("Task from runBlocking")
    }
    
    coroutineScope { // Creates a coroutine scope
        launch {
            delay(500L) 
            println("Task from nested launch")
        }
    
        delay(100L)
        println("Task from coroutine scope") // This line will be printed before the nested launch
    }
    
    println("Coroutine scope is over") // This line is not printed until the nested launch completes
}
```

```
Task from coroutine scope
Task from runBlocking
Task from nested launch
Coroutine scope is over
```

## 리팩토링: suspend 함수 작성하기

launch 안에 있는 코드들을 기능분리하고 싶을 때 어떻게 리팩토링을 해야할까요? 코루틴 내부의 함수를 외부로 빼내기 위해서는 suspend modifier가 붙어야합니다. suspend 함수는 코루틴 내부에서 일반적인 함수들처럼 사용할 수 있고 다른 suspend 함수들(delay 같은 함수들)도 사용할 수 있습니다.

```kotlin
import kotlinx.coroutines.*

fun main() = runBlocking {
    launch { doWorld() }
    println("Hello,")
}

suspend fun doWorld() {
    delay(1000L)
    println("World!")
}
```

```kotlin
Hello,
World!
```

만약에 기능분리된 함수에서 main 함수의 scope에서 발생한 scope를 포함하고 있다면 어떻게 할까요? 이런 경우에 기능 분리를 할 때에는 CoroutineScope의 확장함수로 만들어서 기능분리를 할 수도 있겠지만, 코드가 깔끔하게 작성되지 못할 가능성이 높습니다.

해당 함수를 포함하는 CoroutineScope를 클래스 내 변수로 가지거나 CoroutineScope를 상속받아서 클래스 내부적으로 구현하여 이를 사용하는 것이 가장 이상적인 해결책입니다. CoroutineScope(coroutineContext)도 사용할 수 있지만 이 스코프에서 실행할 때는 개발자가 더이상 자체적으로 컨트롤을 할 수 없기에 구조적으로 불안정할 수 있으니 웬만하면 사용을 자제하는 것이 좋습니다.

## 코루틴은 정말로 Light-Weight 합니다

다음 두 가지의 코드를 실행해봅시다

```kotlin
import kotlinx.coroutines.*

fun main() = runBlocking {
    repeat(100_000) { // launch a lot of coroutines
        launch {
            delay(5000L)
            print(".")
        }
    }
}
```

```kotlin
import kotlin.concurrent.thread

fun main() {
    thread { 
        repeat(100_000) { // launch a lot of coroutines
            Thread.sleep(5000L)
            print(".")
    }
}
```

10 만개의 코루틴을 동시에 실행했을 때, 각 코루틴은 정상적으로 점을 찍는 일을 수행할 수 있을 것입니다. 그러나 스레드에서 실행할 때에는 중간에 Out Of Memory를 일으키며 프로세스가 종료될 것입니다.

## Global한 코루틴은 Daemon Thread와 같다

다음 코드를 실행해봅시다

```kotlin
import kotlinx.coroutines.*

fun main() = runBlocking {
    GlobalScope.launch {
        repeat(1000) { i ->
            println("I'm sleeping $i ...")
            delay(500L)
        }
    }
    delay(1300L) // just quit after delay    
}
```

```
I'm sleeping 0 ...
I'm sleeping 1 ...
I'm sleeping 2 ...
```

GlobalScope에서 launch된 코루틴은 프로세스의 생명주기를 지켜주지 않습니다, 즉 GlobalScope에서 작동되는 모든 동작이 프로세스가 실행되는 동안 다 작동되리라는 보장을 못합니다. 마치 Thread에서 Daemon Thread와 같이요.

