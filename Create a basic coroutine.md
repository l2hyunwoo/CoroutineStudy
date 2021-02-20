# Coroutine 공식문서 번역기 - 2 

- 다음 문서는 KotlinLang에서 제공되는 Coroutines 공식문서를 번역(의역)한 내용입니다.
- 최대한 번역체를 줄이고 Reader-Friendly하게 작성하려고 노력을 하였습니다.
- 1탄에 중복되는 내용이 일부 있어 이를 최대한 배제하고 내용을 컴팩트하게 전달하려 했습니다.

## Tutorial

코틀린 1.1에서 새로운 Non-Blocking으로 비동기 처리를 할 수 있는 코루틴을 소개했습니다. 이번 글에서는 코틀린에서 코루틴을 사용법에 대한 기초를 배울 수 있을 것입니다.

이번 Basic Tutorial에서 사용할 개발환경은 다음과 같습니다.

- Build 도구: Gradle Kotlin DSL
- Project SDK: 1.8
- Kotlinx-coroutines version: 1.4.2

## Project Setting

![Create a new project](https://kotlinlang.org/docs/images/new-gradle-project-jvm.png)

IntelliJ에서 다음과 같이 프로젝트를 셋업하시고

![스크린샷 2021-02-20 오후 2.48.15](https://media.vlpt.us/images/l2hyunwoo/post/4ace9489-3eae-45ae-a7f8-e26831e18217/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA%202021-02-20%20%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE%202.48.15.png)

프로젝트에 build.gradle.kts에 repositories에 ``jcenter()``를 추가하시고 dependencies에 ``implementation("org.jetbrains.kotlinx:kotlinx-coroutines-core:1.4.2")``를 추가하시면 됩니다.

## IDE에서 코루틴 작성해보기 with ``launch()``

지난 글에서도 언급했다시피, 코루틴은 매우 가벼운 스레드라고 생각할 수 있습니다. 스레드와 같이 코루틴 역시 병렬적으로 실행될 수 있고 각각의 코루틴끼리 상호 대기/통신할 수 있습니다. 

그러나 스레드와 가장 큰 차이점은 **시스템 자원 소모량**입니다. 코루틴은 아주 적은 코스트로 수 천개의 코루틴을 생산해낼 수 있습니다. 그러나 스레드에서는 아무리 발전된 현대 컴퓨터일지라도 이는 굉장히 큰 부담(?)을 수반할 수 있을 것입니다.

이제 코루틴을 사용해볼까요? launch를 활용하여 새로운 코루틴을 만들어봅시다.

```kotlin
launch {
  ...
}
```

기본적으로, 코루틴은 **스레드 풀에서 실행**이 됩니다. 아무리 코루틴일지라도 기본적으로 스레드가 비동기 처리의 근간이 되어줍니다. 그러나 기존 스레드와는 달리 코루틴은 한개의 스레드에서 다수의 코루틴을 실행시킬 수 있습니다. 따라서 코루틴은 많은 양의 스레드를 요구하지 않습니다.

자 이제 완성된 튜토리얼 프로그램을 돌려볼까요?

```kotlin
import kotlinx.coroutines.*

fun main(args: Array<String>) {
    println("Start")

    // Start a coroutine
    GlobalScope.launch {
        delay(1000)
        println("Hello")
    }

    Thread.sleep(2000) // wait for 2 seconds
    println("Stop")
}
```

```
Start
Hello
Stop
```

## Async: 코루틴에서 return되는 값 받아오기

지난 게시글에서 GlobalScope는 main 함수의 생명주기에 종속되어서 내부 로직이 완성되기도 전에 프로그램이 종료가 되는 부분을 보셨나요? 이 부분을 ``async()``로 해결해보고자 합니다.

async는 launch와 같이 코루틴을 만들 수 있는 코루틴 빌더입니다. 하지만 async는 Deffered<T> 객체를 리턴합니다. 이 객체는 기초적인 Future(Promise) 패턴으로 구성되어 있습니다. 이미 JDK에서는 Future를 통해 이와 비슷한 기능을 제공하고 있지만 지금 저희는 코루틴에 국한시켜 Deffered만 다루도록 하겠습니다.

이제 async를 활용해 이전 게시물에 있었던 코드를 다시 짜보도록 하겠습니다.

```kotlin
import kotlinx.coroutines.*

fun main(args: Array<String>) {
    val deffered = (1..1_000_000).map {
        GlobalScope.async { it }
    }
    val sum = deffered.sumOf { it.await().toLong() }
}
```

이렇게 작성을 하면 await 부분에서 오류가 발생합니다. await 함수가 **suspend 함수**이기 때문입니다. 따라서 다음과 이를 실행시킬 수 있는 Coroutine Scope을 따로 만드셔서 실행시키셔야 합니다.

```kotlin
import kotlinx.coroutines.*

fun main(args: Array<String>) {
    val deffered = (1..1_000_000).map {
        GlobalScope.async { it }
    }

    runBlocking {
        val sum = deffered.sumOf { it.await().toLong() }
        println("Sum: $sum")
    }
}
```

그렇다면 아래와 같이 코드를 수정시키고 다시 프로그램을 실행시켜 보세요. 프로그램의 예상 실행시간은 어느정도로 예상하셨나요? 그리고 실제 실행시간은 어느정도 되었나요?

```kotlin
import kotlinx.coroutines.*

fun main(args: Array<String>) {
    val deffered = (1..1_000_000).map {
        GlobalScope.async { 
            delay(1000)
            it 
        }
    }

    runBlocking {
        val sum = deffered.sumOf { it.await().toLong() }
        println("Sum: $sum")
    }
}
```

혹시 1,000,000 초를 예상하셨나요? 코루틴은 **병렬적으로** 실행됩니다. 따라서 모든 코루틴에 1초 delay 함수를 걸었더라도 실제로는 여러 코루틴이 동시에 시작되기 때문에 1 * (1,000,000) 초에 걸쳐서 실행되지는 않습니다.
