# Higher-order funtions, Inline


인라인 함수란 무엇일까?
인라인 함수는 컴파일러가 함수 호출부에 함수 본문(내용)을 삽입하는 것이다. 바이트코드를 삽입하는 것이다.

그렇다면 왜 쓰는 것일까?
고차 함수를 사용하면 런타임 패널티가 있기때문에 함수 구현 자체를 코드 내부에 넣음으로써 오버헤드를 없앨 수 있다.
Document에 있는 글이다.
즉, 고차 함수의 런타임 패널티를 없애기 위해서 쓴다.
그렇다면 고차 함수가 무엇인지 알아보는게 순서인거 같다.

## Higher-order functions
고차함수는 함수를 파라미터로 받고 함수를 리턴하는 함수이다.
예를 들어,

    fun <T> ArrayList<T>.filterOnCondition(condition: (T) -> Boolean): ArrayList<T>{
        val result = arrayListOf<T>()
        for (item in this){
            if (condition(item)){
                result.add(item)
            }
        }

        return result
    }

이런 코드가 있다.
condition: (T) -> Boolean
부분을 보자.
이 말은 현재 확장하고 있는 주체인 T를 파라미터로 받고 Boolean값을 리턴한다는 뜻이다. 
이게 고차함수 함수이다.

조금 더 예를 들어보자.

    fun isMultipleOf (number: Int, multipleOf : Int): Boolean{
        return number % multipleOf == 0
    }

    var list = arrayListOf<Int>()
    for (number in 1..10){
        list.add(number)
    }
    
    var resultList = list.filterOnCondition { isMultipleOf(it, 5) }

이런 코드가 있다.
이 코드르 아래와 같이 줄일 수 있다.

    var resultList = list.filterOnCondition { it -> it % 5 == 0  } 
    or
    var resultList = list.filterOnCondition { it % 5 == 0 }

간단하게 알아봤다.

그럼 inline함수를 다시 알아보도록 하겠다.

## Inline
위에서 고차함수의 런타임 패널티를 없앤다고 하였다.
그렇다면 Inline이 뭔지 간단히 예를 들어서 보자.

    fun doSomething() {
        print("doSomething start")
        doSomethingElse()
        print("doSomething end")
    }

    fun doSomethingElse() {
        print("doSomethingElse")
    }

이런 코드가 있다. 이 코드의 decompiled된 것을 보자.

    public void doSomething() {
       System.out.print("doSomething start");
       doSomethingElse();
       System.out.print("doSomething end");
    }

    public void doSomethingElse() {
       System.out.print("doSomethingElse");
    }
    
즉, doSomething()함수에서 doSomethingElse를 호출하고 있다.

이번에는 inline을 붙여보자.

    fun doSomething() {
        print("doSomething start")
        doSomethingElse()
        print("doSomething end")
    }

    inline fun doSomethingElse() {
        print("doSomethingElse")
    }
    
이런 코드가 있다. 이 코드의 decompiled된 것을 보자.

    public void doSomething() {
       System.out.print("doSomething start");
       System.out.print("doSomethingElse");
       System.out.print("doSomething end");
    }
    
doSomethingElse의 코드가 doSomething()부분에 카피되어 삽입되어 있는 것을 볼 수 있다.
즉, doSomthing()은 더 이상 doSomethingElse()를 호출하지 않는다.
이게 인라인함수를 붙였을때의 차이이다.

이것이 고차함수와 사용했을 때 왜 이득이 있는지 살펴보자
먼저, inline을 붙이지 말아보자.

    fun doSomething() {
        print("doSomething start")
        doSomethingElse {
            print("doSomethingElse")
        }
        print("doSomething end")
    }

    fun doSomethingElse(abc: () -> Unit) {
        // I can take function
        // do something else here
        // execute the function
        abc()
    }

아래는 decompiled된 코드 이다.

    public void doSomething() {
        System.out.print("doSomething start");
        doSomethingElse(new Function() {
            @Override
            public void invoke() {
                System.out.print("doSomethingElse");
            }
        });
        System.out.print("doSomething end");
    }

    public void doSomethingElse(Function abc) {
        abc.invoke();
    }
    
인라인을 붙여보자.

    fun doSomething() {
        print("doSomething start")
        doSomethingElse {
            print("doSomethingElse")
        }
        print("doSomething end")
    }

    inline fun doSomethingElse(abc: () -> Unit) {
        // I can take function
        // do something else here
        // execute the function
        abc()
    }
    
아래는 decompiled된 코드이다

    public void doSomething() {
        System.out.print("doSomething start");
        System.out.print("doSomethingElse");
        System.out.print("doSomething end");
    }

앞서 살펴본 것과 마찬가지로 호출부에 함수 본문이 삽입되어 있다.
이로써, inline은 function calls를 피할 수 있다. 또한, 새로운 인스턴스 생성도 막을 수 있다.

그래서 inline함수를 쓰면 장점이 뭐가 있을까?
1. Function Call 오버헤드가 발생하지 않는다.
2. 함수가 호출 될 때, 스택에 push/pop 오버헤드를 save한다.
3. 함수의 return call 오버헤드도 save한다.
4. 새로운 인스턴스 생성을 방지한다.

언제쓰면 좋을까?
1. 고차함수가 사용될 때
2. 그러면서, 함수 코드 블럭이 너무 크지 않을때 쓰며 좋을 것 같다.

코드가 너무 커지고 여러곳에 많이 불리면 큰 코드 덩어리가 여러곳에 반복되어 삽입되어지게 된다.

참조 :
https://medium.com/@agrawalsuneet/higher-order-functions-in-kotlin-3d633a86f3d7
https://blog.mindorks.com/understanding-inline-noinline-and-crossinline-in-kotlin

다시 살펴보기 :
https://blog.kotlin-academy.com/effective-kotlin-consider-inline-modifier-for-higher-order-functions-758afcaffc11
