# Crossinline

inline을 쓸 때 주의점이 있다.
'non-local returns'를 초래한다는 것이다.
그렇다면 non-local returns 란 무엇일까?

## non-local returns

아래와 같은 코드가 있다.

    fun doSomething() {
        print("doSomething start")
        doSomethingElse {
            print("doSomethingElse")
            return // notice this return
        }
        print("doSomething end")
    }

    inline fun doSomethingElse(abc: () -> Unit) {
        // I can take function
        // do something else here
        // execute the function
        abc()
    }
    
위 결과는 
doSomething start
doSomethingElse
만 찍힌다. 즉, doSomething end는 찍히지 않는 것이다.
decompile을 해봐도 doSomething end는 보이지 않는다.
이것이 non-local returns의 개념이다. (inline돤)lamda 내부의 리턴이 아니라 외부 doSomething() 을 리턴한다.

그렇다면 crossinline이 뭔지를 살펴보자

## crossinline

crossinline은 이러한 원치 않는 결과가 일어나지 않도록 강제해주는 역할을 한다.
즉, 'non-local returns'를 피하게 해준다.

다시 코드를 보자.

    fun doSomething() {
        print("doSomething start")
        doSomethingElse {
            print("doSomethingElse")
            // return is not allowed here
        }
        print("doSomething end")
    }

    inline fun doSomethingElse(crossinline abc: () -> Unit) {
        // I can take function
        // do something else here
        // execute the function
        abc()
    }

crossinline이 추가 되었다. 그리고 return 자리의 주석을 살펴볼 수 있다.
return이 허용되지 않는다는 것이다. return을 쓰면 컴파일 에러를 낸다.
이제 실행해보면 결과는

doSomething start
doSomethingElse
doSomething end

모두 잘 찍힌다.

참고자료 : 
https://medium.com/@tferreirap/kotlin-quick-look-at-inline-noinline-and-crossinline-e62e8833db1f
https://blog.mindorks.com/understanding-inline-noinline-and-crossinline-in-kotlin
