# Coroutine

### Coroutine이란?

co + routine => cooperative + function
하나의 쓰레드 안에서 함수들이 서로 협력한다.
Coroutine 은 Thread 의 대안이 아니라 기존의 Thread를 더 잘게 쪼개어 사용하기위한 개념임.

Coroutines allow you to achieve concurrent behavior without switching threads, which results in more efficient code. Therefore, coroutines are often called “lightweight threads”.

it is possible to achieve concurrency without creating expensive threads

A framework to manage concurrency in a more performant and simple way with its lightweight thread which is written on top of the actual threading framework to get the most out of it by taking the advantage of cooperative nature of functions.


### Continuations?

함수가 suspend되면, suspend된 코루틴의 정보, 상태가 있다. 코루틴이 suspend될 때마다, continuation에 상태를 저장한다. 코루틴이 resume되면, continuation은 남은 코루틴 실행을 계속하기에 충분한 정보를 가지고 있다. continuation은 코루틴 컨텍스트와 코루틴의 성공 혹은 실패를 보고할 완료 콜백으로 이루어져 있다.

The Continuation interface consists of a CoroutineContext and a completion callback used to report the success or failure of the coroutine.

### Structured Concurruncy

nested 코루틴 스코프를 관리하는 기법. 코루틴 안에 또 다른 코루틴 스코프를 만들어서 관리하는 기법.
outer coroutine does not complete until all the coroutines launched in its scope complete. 
https://kotlinlang.org/docs/reference/coroutines/basics.html#structured-concurrency

1. Cancel work when it is no longer needed. ->  When a scope cancels, all of its coroutines cancel
2. Keep track of work while it’s running. ->  When a suspend fun returns, all of its work is done.
3. Signal errors when a coroutine fails. -> When a coroutine errors, its caller or scope is notified.


### coroutineScope, supervisorScope 

coroutineScope and supervisorScope will wait for child coroutines to complete.
coroutineScope, supervisorScope 차이 : child코루틴 들 간에 에러 전파 여부.
coroutineScope는 에러 던져지면 즉시 취소시키고 child, parent 모두 전파.
supervisorScope는 에러 던져지면 child들에게만 전파(단, 다른 코루틴들을 취소시키지 않고 끝날때까지 대기). 부모는 cancecl안됨.
https://tourspace.tistory.com/155
https://blog.mindorks.com/mastering-kotlin-coroutines-in-android-step-by-step-guide
https://stackoverflow.com/questions/53577907/when-to-use-coroutinescope-vs-supervisorscope
https://proandroiddev.com/managing-exceptions-in-nested-coroutine-scopes-9f23fd85e61


### Coroutine Scope, Coroutine Context

CoroutineScope -
코루틴 생성
CoroutineScope는 coroutineContext를 가지고있는 interface이다.
all coroutines run inside a CoroutineScope. A scope controls the lifetime of coroutines through its job -> coroutine 상태 추적
https://medium.com/androiddevelopers/coroutines-on-android-part-ii-getting-started-3bff117176dd

CoroutineContext -
코루틴 정보
Job, CoroutineDispatcher, CoroutineName, CoroutineExceptionHandler(포착되지 않은 예외 처리)

There is also an interface called CoroutineScope that consists of a sole property — val coroutineContext: CoroutineContext. It has nothing else but a context. So, why it exists and how is it different from a context itself? The difference between a context and a scope is in their intended purpose.
->Every coroutine builder is an extension on CoroutineScope and inherits its coroutineContext to automatically propagate both context elements and cancellation.
즉, 스코프를 자동으로 전파하려고. 스코프 hierarchy만들려고. 결국, job을 통해 cancel.
https://medium.com/@elizarov/coroutine-context-and-scope-c8b255d59055
https://stackoverflow.com/questions/54416840/kotlin-coroutines-scope-vs-coroutine-context
https://proandroiddev.com/demystifying-coroutinecontext-1ce5b68407ad


### Job
A coroutine itself is represented by a Job. It is responsible for coroutine’s lifecycle, cancellation, and parent-child relations. A current job can be retrieved from a current coroutine’s context:
즉, 코루틴의 수명 주기를 제어

- 자식이 실패하면, 다른 자식들에게 실패 전파
- 실패를 통보받으면, 부모에게 전달

### SupervisorJob

- 자식이 실패해도 다른 자식들에게 영향을 미치지 않음
- 실패를 통보받아도, 그 스코프는 아무것도 하지 않음. 위로 전달x


