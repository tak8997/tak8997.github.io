
# Android CPU, Compilers, D8 & R8

목표 :
1. JVM과 안드로이드와의 연관성
2. 바이트코드 읽기
3. 안드로이드 빌드 시스템에 대한 아이디어
4. AOT, JIT이 의미하는 바, 그리고 R8 컴파일러와의 연관성



## Inside JVM

#### ClassLoader
컴파일 된 자바파일(.class), linking, 손상된 바이트 코드 감지, 정적 변수, 코드 할당 및 초기화

#### Rumtime Data
모든 프로그램 데이터 : 스택, 메소드 변수들, 힙

#### Execution Engine
컴파일 및 로드 된 코드의 실행, 가비지 정리




## Interpreter & JIT
프로그램을 실행 할 때 마다, Interpreter 바이트코들를 기계 코드로 번역한다. Interpreter의 단점은 어떤 메소드가 호출 되서 번역할 때, 여러번 호출되도 호출 될 떄마다 반복적으로 번역한다는 것이다.

Jit은 이러한 단점을 보완시켜준다. Execution Engine이 Interpreter의 도움으로 바이트코드를 변환시킨다. 하지만, 반복되는 코드가 있다면, Jit 컴파일러를 사용한다.
가능한 많은 바이트코드를 컴파일(임계값 까지)하고 네이티브 코드로 변경한다. 네이티브 코드는 반복적인 메소드 호출에 사용되고, 그래서 시스템 퍼포먼스를 향상 시킬 수 있다. 이러한 반복되는 코드를 'Hot code'라고 부른다(Interpreter는 'Cold Code', Jit은 'Hot Code').


이제 이것이 안드로이드와 어떤 연관성을 갖는지 살펴볼 수 있다.
기존 JVM은 충분한 전력과 제한 없는 저장 공간을 가지고 디자인 되었다. 하지만, 안드로이드 디바이스는 베터리 한계, 프로세스들은 리소스를 두고 경쟁한다. 또한
RAM도 작고, 디바이스 저장공간도 충분치 않다. 그래서, 구글은 JVM구조를 많이 바꿨다 - 바이트코드 컴파일 및 바이트코드 구조. 
아래는 예시이다.

    public int method(int i1, int i2) {
        int i3 = i1 * i2;
        return i3 * 2;
    }

regular java compiler

    .method public method(II)I
    .limit locals 4
    .var 0 is this LTest2; from Label0 to Label1
    .var 1 is arg0 I from Label0 to Label1
    .var 2 is arg1 I from Label0 to Label1
    Label0:
      iload_1
      iload_2
      imul
      istore_3
      iload3
      iconst_2
      imul
    Label1:
      ireturn
    .end method
    
android dex compiler

    .method public method(II)I
    .limit registers 4
    mul-int v0, v2, v3
    mul-int/lit-8 v0,v0,2
    return v0
    .end method
    
기존 자바 바이트코드는 스택 베이스이다(모든 변수들이 스택에 저장), 반면, 덱스 바이트코드는 레지스터 기반이다(모든 변수들이 레지스터에 저장).
덱스 방식이 기존 방식보다 더 효율적이고 더 적은 공간을 필요로 한다. 이러한 덱스 바이트코드는 달빅 이라는 안드로이드 가상 머신에서 실행된다.
**달빅은 덱스 컴파일러에 의해 컴파일 된 바이트코드를 로드하는 것이다.** 실행은 기존 JVM과 비슷하게 JIT & Interpreter를 사용한다.


## Bytecode?
바이트코드는 JVM이 이해할수 있는 언어로 컴파일된 자바 코드이다. 
예를 보자.

    mul-int   v0,v2,v3
    ->opcode  ->registers
     (명령코드)

바이트코드 모든 작업들은 opcode, registers, consonants로 이루어진다. 
모든 타입들은 자바와 같다.

    ● I — int
    ● J — long
    ● Z — boolean
    ● D — double
    ● F — float
    ● S — short
    ● C — char
    ● V — void (when return value)

클래스는 full path로 보여진다

    Ljava/lang/Object
    
Arrays들은 '[' 로 시작하고 타입이 온다.

    [I, [Ljava/lang/Object;, [[I
    
예를 보자.

    obtainStyledAttributes(Landroid/util/AttributeSet;[III)
    
obtainStyledAttributes는 메소드 이름이다. 
Landroid/util/AttributeSet; 은 AttributeSet을 첫 번째 파라미터,
[I 는 integer 배열
I 는 integer 2번
을 의미한다.

    obtainStyledAttributes(AttributeSet set, int[] attrs, int defStyleAttr, int defStyleRes)

자바코드는 위와 같다.

또 다른 예를 보자.

    .method swap([II)V ;swap(int[] array, int i)
     .registers 6
     aget v0, p1, p2 ; v0=p1[p2]
     add-int/lit8 v1, p2, 0x1 ; v1=p2+1
     aget v2, p1, v1 ; v2=p1[v1]
     aput v2, p1, p2 ; p1[p2]=v2
     aput v0, p1, v1 ; p1[v1]=v0
    return-void
    .end method
    
자바코드는 아래와 같다.

    void swap(int array[], int i) {
        int temp = array[i];
        array[i] = array[i+1];
        array[i+1] = temp;
    }
    
여기서 6번째 레지스터는 어딨을까?
메소드가 인스턴스의 메소드면, 디폴트 파라미터 'this'가 레지스터에 항상 저장된다. 그것이 p0 이다.
이는 메소드가 static이면, p0 파라미터는 다른 의미를 가지는 것이다.

또 다른 예를보자.

    const/16 v0, 0x8                      ;int[] size 8
    new-array v0, v0, [I                  ;v0 = new int[]
    fill-array-data v0, :array_12         ;fill data
    nop
    :array_12
    .array-data 4
            0x4
            0x7
            0x1
            0x8
            0xa
            0x2
            0x1
            0x5
    .end array-data

자바코드는 아래와 같다.

    int array[] = {
         4, 7, 1, 8, 10, 2, 1, 5
    };
    
마지막 예시를 보자.

    new-instance p1, Lcom/android/academy/DexExample;
                ;p1 = new DexExample();
    invoke-direct {p1}, Lcom/android/academy/DexExample;-><init>()V
                ;calling to constructor: public DexExample(){ ... }
    const/4 v1, 0x5          ;v1=5
    invoke-virtual {p1, v0, v1}, Lcom/android/academy/DexExample;->swap([II)V .           ;p1.swap(v0,v1)
   
자바코드는 아래와 같다.

    DexExample dex = new DexExample();
    dex.swap(array,5);
    

이제 D8, R8을 알아보자. 그 전에 android jvm 달빅을 다시 한번 볼 것이다.










