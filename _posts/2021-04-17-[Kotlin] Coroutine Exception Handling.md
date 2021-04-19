# Coroutine Exception Handling


<br/>
<br/>


- ### 일반적으로 CoroutineScope(Job())일 경우, 예외 처리


![image](https://user-images.githubusercontent.com/19990905/115106123-f7bd6b00-9f9d-11eb-961a-7b221430abbd.png)

코루틴에서 예외가 발생하면, 부모한테 예외를 전달한다. 그러면
1) 부모는 자식들을 취소한다
2) 자기 자신을 취소한다
3) 부모한테 예외를 전달한다.

하지만, 예외가 발생해도 다른 자식 코루틴, 혹은 부모 코루틴은 취소하기 싫은 수도 있다.
그럴 경우??

</br>
</br>

- ### SupervisorJob의 예외처리

![image](https://user-images.githubusercontent.com/19990905/115106289-dc069480-9f9e-11eb-8f29-b612b2002564.png)

SupervisorJob을 쓰면, 부모, 혹은 다른 자식들에게 자신의 예외가 영향을 미치지 않는다. 자식들은 자기들 스스로 예외를 처리하게 된다.
예를 보자


    val scope = CoroutineScope(SupervisorJob())
    scope.launch {
        // Child 1
    }
    scope.launch {
        // Child 2
    }


child1에서 예외가 발생하면, child2는 예외가 발생하지 않는다.
또 다른 예를 보자


    val scope = CoroutineScope(Job())
    scope.launch {
        supervisorScope {
            launch {
                // Child 1
            }
            launch {
                // Child 2
            }
        }
    }

마찬가지로 child1이 예외가 발생해도, child2는 예외가 발생하지 않는다.
supervisorScope는 SupervisorJob을 가지고 스코프를 만든다. 당연히 coroutineScope를 쓰면 모두 취소된다.


- ### Who's my parent?


</br>


    val scope = CoroutineScope(Job())
    scope.launch(SupervisorJob()) {
        // new coroutine -> can suspend
       launch {
            // Child 1
        }
        launch {
            // Child 2
        }
    }


child1부모의 Job종류는 무엇일까??
정답은 Job이다. 언뜻보기에는 SupervisorJob 처럼 보이지만 그렇지 않다. 왜냐면 새 코루틴은 언제나 Job을 할당받기 떄문이다.
SupervisorJob은 scope.launch의 부모다(이 경우에는 SupervisorJob은 아무 의미가 없다).


![image](https://user-images.githubusercontent.com/19990905/115107159-16befb80-9fa4-11eb-8c64-36e4ca2306c0.png)


그렇기에 child1에서 예외가 발생하면, 전부 취소된다.

**결론은 SupervisorJob은 supervisorScope 또는 CoroutineScope(SupervisorJob())를 사용하여 생성 된 범위의 일부인 경우에만 제대로 작동한다.**
SupervisorJob을 코루틴 빌더의 파라미터로 넘기면 예상한대로 취소가 되지 않을 것이다.


</br>
</br>


- ### Launch, Async에 따른 Exception Handling 비교

</br>

#### Launch 

</br>

    scope.launch {
        try {
            codeThatCanThrowExceptions()
        } catch(e: Exception) {
            // Handle exception
        }
    }

예외가 바로 발생하고 catch문에 잡힌다.

</br>

#### Async 

</br>

async가 root 코루틴으로 사용될 경우(CoroutineScope 인스턴스, 혹은 supervisorScope의 직계 자식 코루틴일 경우)
예외는 자동으로 던져지지 않는다. 대신 .await()를 호출해야만 던져진다.


    supervisorScope {
        val deferred = async {
            codeThatCanThrowExceptions()
        }
        try {
            deferred.await()
        } catch(e: Exception) {
            // Handle exception thrown in async
        }
    }


.await()를 try ~ catch로 감싸야만 한다. async만으로는 예외를 던지지 않는다.
하지만, Job을 쓰면 어떻게될까?


    coroutineScope {
        try {
            val deferred = async {
                codeThatCanThrowExceptions()
            }
            deferred.await()
        } catch(e: Exception) {
            // Exception thrown in async WILL NOT be caught here 
            // but propagated up to the scope
        }
    }


Job은 부모로 예외를 전파하기 때문에 catch에 걸리지 않는다.
더 나아가, 다른 코루틴에 의해 생성 된 코루틴에서 발생하는 예외는 코루틴 빌더와 상관없이 항상 전파된다.


    val scope = CoroutineScope(Job())
    scope.launch {
        async {
            // If async throws, launch throws without calling .await()
        }
    }
    
    
    
**이 경우, await()를 하지 않아도 바로 예외가 발생한다. 왜냐하면, async코루틴은 launch의 직계 자식이기 떄문이다. 즉, CoroutineScope가 아니다.**
그 예로 아래와 같은 코드가 있다면 어떨까??


    CoroutineScope(Job()).async {
        exception()
    }
    //.await()


위 코드는 exception이 발생하지 않는다. async가 루트 코루틴, 즉, CoroutineScope의 직계 이기 때문이다. 
이 같은 경우, await()를 꼭 호출해야 exception()이 발생한다.


**결론적으로 async를 사용할때는 **
**1) coroutineScope를 사용할 경우**
**2) 다른 코루틴에 의해 생성되었을 경우(직계가 아닐 경우)**
**try ~ catch에 걸리지 않으므로 조심해야 한다!**



마지막으로, CoroutineExceptionHandler에 대해 알아보자.


</br>
</br>


- ### CoroutineExceptionHandler

</br>


CoroutineExceptionHandler는 'handle uncaught exceptions' 이다.


    val handler = CoroutineExceptionHandler {
        context, exception -> println("Caught $exception")
    }


예외가 발생하면, 그 exception과 CoroutineContext가 있다.
예를 보자.


    val scope = CoroutineScope(Job())
    scope.launch(handler) {
        launch {
            throw Exception("Failed coroutine")
        }
    }


이렇게 정의했을 때, 예외는 handler에 의해 잘 잡히게 된다.
하지만, 


    val scope = CoroutineScope(Job())
    scope.launch {
        launch(handler) {
            throw Exception("Failed coroutine")
        }
    }


이렇게 정의 됐을 경우, 예외는 handler에 의해 잡히지 않는다.
왜냐하면, 올바른 CoroutineContext에 놓여있지 않기 때문이다. 
앞서 반복했듯이, 예외는 부모에게 전파되는데, 부모에는 어떠한 exception handling을 하고 있지 않기 때문에 예외를 캐치할 수 없다.


</br>
</br>


참고 자료 :

https://medium.com/androiddevelopers/exceptions-in-coroutines-ce8da1ec060c

https://www.lukaslechner.com/why-exception-handling-with-kotlin-coroutines-is-so-hard-and-how-to-successfully-master-it/


