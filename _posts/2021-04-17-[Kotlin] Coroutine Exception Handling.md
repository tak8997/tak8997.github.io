# Coroutine Exception Handling


### 일반적으로 CoroutineScope(Job())일 경우, 예외 처리


![image](https://user-images.githubusercontent.com/19990905/115106123-f7bd6b00-9f9d-11eb-961a-7b221430abbd.png)

코루틴에서 예외가 발생하면, 부모한테 예외를 전달한다. 그러면
1) 부모는 자식들을 취소한다
2) 자기 자신을 취소한다
3) 부모한테 예외를 전달한다.

하지만, 예외가 발생해도 다른 자식 코루틴, 혹은 부모 코루틴은 취소하기 싫은 수도 있다.
그럴 경우??

</br>
</br>

### SupervisorJob의 예외처리

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


### Who's my parent?


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


### Launch, Async에 따른 Exception Handling 비교










