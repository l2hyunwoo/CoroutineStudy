# Asynchronous Programming Techniques

수십년동안 개발자들은 블락 문제를 더 효율적인 방법으로 풀기를 원했다. PC든, 모바일 환경이든 심지어 서버 사이드 제품이더라도 개발자들은 사용자들이 대리를 하게 하는 모든 원인들에서 해방하고 싶어했고 이러한 문제를 예방하고자 했다.

코루틴을 본격적으로 공부하기 전에 이런 문제들을 해결하고자 했던 개발자들의 방법들을 먼저 탐구해보자.

## Threading(스레드)

스레드는 이런 문제를 해결하기 위한 가장 잘 알려진 방법일 것이다. 다음 예시를 보자

```kotlin
fun postItem(item: Item) {
    val token = preparePost()
    val post = submitPost(token, item)
    processPost(post)
}

fun preparePost(): Token {
    // makes a request and consequently blocks the main thread
    return token
}
```

``preparePost()``함수의 실행시간이 매우 오래 걸리고 이로 인해 UI가 블락될 수 있다고 가정해보자. 가장 먼저 생각이 나는 방법은 이 기능을 다른 스레드로 분리하는 것이다. 이는 가장 보편적인 방법인 반면 수 많은 단점이 수반된다.

- 스레드는 절대로 **가볍지 않다**. 스레드는 Context Switch를 일으킨다.
- 스레드는 무한하지 않다. 스레드는 운영체제에 의해 실행할 수 있는 수가 제한되어 있다. 이로 인해 병목현상이 많이 일어난다.
- 어떤 플랫폼에서는 스레드 자체를 이용이 블가능하다. 대표적으로 JavaScript에서는 스레드 자체를 지원해주지 않는다.
- 스레드는 이용하기가 어렵다. 디버깅, Race Condition으로 인해 멀티 스레드 프로그래밍의 러닝커브가 매우 높다.



## Callbacks(콜백)

콜백(함수 자체를 다른 함수의 패러미터로 넘기는 방식) 역시 한 프로세스가 끝나고 이어서 나오는 행동을 결정짓는 다는 점에서 비동기 처리 방식으로 많이 사용되고 있다. 이는 전보다 나은 방식처엄 보이지만 사용성과 관련되어 몇 가지 이슈들이 있다.

- **Callback Hell**

  - 콜백을 사용하는 함수들은 대부분 또 다른 콜백을 부르는 것으로 종료를 시키는 경우가 많다. 콜백을 연속적으로 불러 일으켜서 코드 자체를 이해할 수 없게 만드는 경우가 많다. 

  ```javascript
  step1(function (err, value1) {
      if (err) {
          console.log(err);
          return;
      }
      step2(function (err, value2) {
          if (err) {
              console.log(err);
              return;
          }
          step3(function (err, value3) {
              if (err) {
                  console.log(err);
                  return;
              }
              step4(function (err, value4) {
                  // 정신 건강을 위해 생략
              });
          });
      });
  });
  ```

- **에러 핸들링**

  - 위와 같이 중첩 콜백 구조를 가질 경우 에러 핸들링/함수 전개가 굉장히 복잡해진다.

이런 콜백 구조는 JavaScript에서 Event-Loop 구조에 사용되었는데, 최근에는 이런 문제점들로 인해 Promise나 Rx로 많이 옮겨가고 있다.

## Futures, Promises, and others

- Future/Promise는 언어/플랫폼에 따라서 그 이름이 바뀐다(Java는 Future, JS는 Promise와 같이)

Future(Promise)의 대전제는 다음과 같다

- 만약에 어떤 함수를 Call을 할 때, 그 함수는 **미래(Future)**에 특정 객체가 반환될 것이라 **약속(Promise)**한다.

```kotlin
fun postItem(item: Item) {
    preparePostAsync()
        .thenCompose { token ->
            submitPostAsync(token, item)
        }
        .thenAccept { post ->
            processPost(post)
        }

}

fun preparePostAsync(): Promise<Token> {
    // makes request an returns a promise that is completed later
    return promise
}
```

이 방식(패턴)은 우리가 프로그램을 작성하는 방식을 바꿔놓는다.

- 프로그램의 구조 측면에서
  - 절차지향적/명령적인 콜백 구조와는 달리 Future를 사용하면 구조지향적(Compositional Model)인 방향으로 함수를 구성해야한다.
- 패턴을 구성하는 API 측면에서
  - 각 언어/플랫폼마다 이를 구성하는 멤버 함수가 다 다르기에 각 언어마다 다르게 공부해야한다.
- Retrun Type
  - 일반적으로 우리가 생각하는 것처럼 Class Type, Primitive Type이 오는 것이 아니라 Future(Promise)가 Wrapping되어 리턴이 된다.
- 에러 핸들링
  - 이런 패턴에서 에러의 전파와 처리가 항상 직접적으로 일치하지 않는다.

## Reactive Extensions(Rx)

Rx는 [Erik Meijer](https://en.wikipedia.org/wiki/Erik_Meijer_(computer_scientist))에 의해 C#에 도입되었다. 이는 C#(.NET 플랫폼)에서는 지속적으로 사용하고 있었지만 다른 플랫폼에서는 사용이 안되었다가 Netflix가 이를 본따 RxJava를 만들고 나서 대부분의 팀이 이를 사용하고 더욱 많은 플랫폼(예를 들어서 RxJS)에서 이를 사용하게 되었다.

Rx는 데이터를 observe(옵저버 패턴의 그 옵저브의 의미가 더 가까이 다가와 원어를 사용하였다) 할 수 있는 스트림으로 여겼고, Observer Pattern과 여러 확장함수들을 이용하여 데이터를 스트림(자바 8에 등장하는 그 스트림과 비슷한 개념이다)의 형식으로 받아올 수 있게 했다.

이는 Future와 비슷해 보이지만, Future는 데이터를 하나의 요쇼로 봤고 Rx는 스트림으로 바라봤다. 또한 이는 Furure와 또다른 프로그래밍 방식(**"Everything is a stream, and it's observable"**)을 내놓았다.

Rx는 문제를 접근하는 방식과 동기화 코드를 작성하는 방식에서 많은 변화를 불러일으켰다. 또한, Rx는 C#, Java, JS와 같은 다양한 플랫폼에서 동일한 함수를 제공하여 Future 패턴과는 달리 플랫폼 의존성을 많이 약화시켰고 에러 핸들링 방식도 굉장히 우수하여 갭라자가 사용하기에 더욱 편하게 설계하였다.

## Coroutines(코루틴)

코틀린은 코루틴을 활용하여 비동기 코드를 처리하는 것을 지향한다. 코루틴은 비동기 코드를 마치 일반적인 코드처럼 작성할 수 있게 해주는 엄청난 장점을 가져다 줄 수 있다. 따라서 개발자에게 프로그래밍 방식의 변화를 요구하지 않게 된다.

```kotlin
fun postItem(item: Item) {
    launch {
        val token = preparePost()
        val post = submitPost(token, item)
        processPost(post)
    }
}

suspend fun preparePost(): Token {
    // makes a request and suspends the coroutine
    return suspendCoroutine { /* ... */ }
}
```

위의 코드는 Main 스레드를 블럭하지 않고 오래 걸리는 작업을 수행하는 코드이다. ``preparePost``함수는 앞에 suspend 키워드가 붙는 suspend 함수이다. 이 키워드가 붙은 함수는 말 그대로 기능이 실행되고 잠시 중지된 다음에 일정 시점에서 다시 함수가 재개된다. 

- 함수 앞에 suspend 키워드를 붙이는 것만 제외하면 모든 코드들이 다 형태를 유지하고 기능을 유지하면서 작동한다.
- 코드의 큰 변화를 일으키지 않고 기존에 개발자가 작성했던 대로 순서대로 작성해주기만 하면 된다. 다만 이 기능을 동작시키 위해서는 launch(코루틴을 동작시키기 위한 함수) 함수를 사용하면 된다.
- 프로그래밍 방식, 비동기 처리방식에서 요구하는 특정 세부 함수들이 없다.
- 플랫폼 독립적이어서 JVM 상에서 돌아가든, JS 상에서 돌아가든 그 어떤 플랫폼에서 돌리더라도 동일한 기능을 보장한다.

코루틴은 코틀린에서 제작한 새로운 개념이 아니다. 이 역시 수십년 동안 존재해왔던 개념이고 이미 Go와 같은 언어에서는 이 개념을 차용하여 사용하고 있다. Suspend 키워드를 제외하면 다른 키워드들은 추가된 것이 없고, 문법 자체에 async, await가 추가되어 있는 C#과 달리 Kotlin은 이를 라이브러리 함수로 구현하였다.

