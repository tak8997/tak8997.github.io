# Reified

## Type Erasure 배경지식 먼저 알아보자.

자바에는 Type Erasure라는게 있다. runtime에 인스턴스의 type을 제거하는 것이다.
예를 들어, List<String>이 있다면, runtime에 List만 남고 type을 제거해버린다.
즉, runtime에 type을 알 수가 없다. 예를 들어보자.
    
    if (value is List<String>) { ... } // Cannot check for instance of erased type
    
이런 에러를 낸다. 실행시점에 List는 확실하지만 그 리스트가 String인지, Int인지 알수가 없기 떄문이다.
또 다른 예다.

    fun listSum(li: List<*>) {
        val intList = li as List<Int> //캐스팅 성공
        println(intList.sum())        //런타임 Exception 발생할 수 있음
    }
    listSum(listOf(1, 2, 3))  //6
    listSum(listOf("a", "b")) //컴파일 에러를 나지 않고, 캐스팅도 성공
    => java.lang.ClassCastException: java.lang.String cannot be cast to java.lang.Number

캐스팅은 성공!!하지만(type erasure때문) exception이 나는걸 볼 수 있다.
여기까지 배경지식으로 알아 놓고. Reified를 알아보자.
다른 예를 통해 보자.

## reified 없이 접근

    fun <T> String.toKotlinObject(): T {
          val mapper = jacksonObjectMapper()                                                 
          return mapper.readValue(this, T::class.java)   //does not compile!
    }
    
readValue 메소드는 파싱해야되는 JsonObject타입을 두번째 인자에서 받는다.
우리가 Class의 타입 T를 얻으려고 하면, "Cannot use 'T' as reified type parameter. Use a class instead"
이런식의 에러를 낸다. 

## Class Type을 써서 해결

    fun <T: Any> String.toKotlinObject(c: KClass<T>): T {
        val mapper = jacksonObjectMapper()
        return mapper.readValue(this, c.java)
    }
    
보통 자바에서 사용하던 해결법이다.
호출하는 쪽을 보면

    data class MyJsonType(val name: String)

    val json = """{"name":"example"}"""
    json.toKotlinObject(MyJsonType::class)
    
이렇게 되어있다.
    
## reified를 써서 코틀린 식으로 해결

inline + reified를 써서 더 간단히 해결가능하다.

    inline fun <reified T: Any> String.toKotlinObject(): T {
        val mapper = jacksonObjectMapper()
        return mapper.readValue(this, T::class.java)
    }
    
    json.toKotlinObject<MyJsonType>()
    
이렇게 일반 함수처럼 사용할 수 있다.

위에서 언급한 Type Erasure의 해결이다. 
즉, 마치 런타임 타입검사가 가능한 것 처럼 말이다.
reified는 타입 파라미터가 실행 시점에 지워지지 않음을 표시하는 것이다.
이게 가능한 이유를 유추해보자면, inline함수는 호출부에 직접 삽입되기 때문이다.
타입인자가 아니라 구체적인 타입 자체를 사용할 수 있는 것이다.

마지막, 주의할점.
inline + reified 함수는 자바코드에서 호출할 수 없다.
