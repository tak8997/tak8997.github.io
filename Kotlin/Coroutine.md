# Coroutine

## Coroutine이란?

co + routine => cooperative + function
하나의 쓰레드 안에서 함수들이 서로 협력한다.

Coroutines allow you to achieve concurrent behavior without switching threads, which results in more efficient code. Therefore, coroutines are often called “lightweight threads”.
it is possible to achieve concurrency without creating expensive threads
A framework to manage concurrency in a more performant and simple way with its lightweight thread which is written on top of the actual threading framework to get the most out of it by taking the advantage of cooperative nature of functions.

## Structured Concurruncy

nested 코루틴 스코프를 관리하는 기법. 코루틴 안에 또 다른 코루틴 스코프를 만들어서 관리하는 기법.
outer coroutine does not complete until all the coroutines launched in its scope complete. 
https://kotlinlang.org/docs/reference/coroutines/basics.html#structured-concurrency

1. Cancel work when it is no longer needed. ->  When a scope cancels, all of its coroutines cancel
2. Keep track of work while it’s running. ->  When a suspend fun returns, all of its work is done.
3. Signal errors when a coroutine fails. -> When a coroutine errors, its caller or scope is notified.


## coroutineScope, supervisorScope 

coroutineScope and supervisorScope will wait for child coroutines to complete.
coroutineScope, supervisorScope 차이 : child코루틴 들 간에 에러 전파 여부.
coroutineScope는 에러 던져지면 즉시 child, parent 모두 전파.
supervisorScope는 에러 던져지면 child들에게만 전파(단, 다른 코루틴이 끝날때까지 대기). 부모는 cancecl안됨.
https://tourspace.tistory.com/155
https://blog.mindorks.com/mastering-kotlin-coroutines-in-android-step-by-step-guide
https://stackoverflow.com/questions/53577907/when-to-use-coroutinescope-vs-supervisorscope
