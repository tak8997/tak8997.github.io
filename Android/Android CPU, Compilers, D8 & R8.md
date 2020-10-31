
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
가능한 많은 바이트코드를 컴파일(임계값 까지)하고 네이티브 코드로 변경한다. 네이티브 코드는 반복적인 메소드 호출에 사용되고, 그래서 시스템 퍼포먼스를 향상 시킬 수 있다. 이러한 **반복되는 코드를 'Hot code'라고 부른다(Interpreter는 'Cold Code', Jit은 'Hot Code').**


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


## Android build process

![android-build-process](https://user-images.githubusercontent.com/19990905/95824354-1bdb8d80-0d6a-11eb-997c-c407f635be35.png)

**.java, .kt 파일들은 자바, 코틀린 컴파일러에 의해 .class파일을 만든다. .class파일들은 덱스 컴파일러에 의해 .dex로 컴파일 된다. 마지막으로, 
.apk 파일로 패키지 된다. **

플레이스토어에서 앱을 다운로드하면, apk 파일과 함께 모든 리소스들(images, icons, layouts) 그리고 컴파일된 자바코드를 다운로드 하게 된다.
앱을 실행하면, 달빅 프로세스르 시작한다. 즉, dex코드가 로드 되고 Interpreter에 의해 번역되고 JIT에 의해 런타임에 컴파일 된다. 

![dalvik-process](https://user-images.githubusercontent.com/19990905/96329147-26a56380-1085-11eb-9009-28baf118f856.png)



## ART

하지만, davlik은 한계가(배터리 소모, 램 소모 높음) 있다. 그래서 구글은 ART라고 불리는 JVM을 소개했다. 주요 차이점은 **ART가 런타임!에 
Interpreter, JIT을 실행시키지 않는다는 것이다.** 즉, .oat 라는 바이너리로 부터 미리 컴파일 된 코드를 실행시킨다. 결과적으로 더 빠른 런타임을 자랑한다. 코드를 .oat 바이너리로 컴파일 하기 위해선, ART는 AOT(Ahead of Time) 컴파일을 쓴다.

.oat 바이너리란??

![image](https://user-images.githubusercontent.com/19990905/96329333-da5b2300-1086-11eb-8599-f53db6aed683.png)

플레이 스토어에서 앱을 설치하면, apk파일 압축해제 이외에도, **.dex파일을 .oat바이너리로 컴파일하는 과정을 시작한다.**
앱을 실행하면, ART는 .oat바이너리를 로드하고 interpretation이나 jit 컴파일 과정 없이 즉시 실행시킨다.

하지만 업데이트에 있어서 단점도 있다.

* .dex를 .oat바이너리로 컴파일하는 과정은 설치 과정의 일부이다. 이는 설치, 업그레이드 시간에 큰 영향을 미친다. 또한, 모든 안드로이드 os
업데이트는 1,2 시간의 앱 최적화 프로세스를 시작한다. 이는 매달 보안 업데이트 해야하는 nexus 사용자한테는 큰 고통이다.

* 저장 공간 : 전체 .dex파일이 .oat로 컴파일 됐다. 심지어 사용자가 별로 사용하지 않는 부분도 말이다(예를 들어, 앱 셋팅, 로그인 화면, 보통 
한, 두번 들어가고 잘 사용하지 않는). 그래서 기본적으로, disk공간 낭비가 있고 이는 제한된 저장 공간을 가지는 low-end 다비이스들에게 있어서는
문제가 된다.


이런 문제들 역시 구글 엔지니어들에 의해 개선이 됐다. jit, interpreter, aot를 합치는것이다.
1. 처음에는 .oat 바이너리가 없다. 앱을 처음 실행하면, art가 인터프리터를 이용해서 코드를 실행시킨다.
2. Hot code가 발견되면, Jit컴파일러를 이용해 컴파일한다.
3. Jit 컴파일된 코드는 캐시에 저장된다. 이후 모든 실행에서 이 캐시를 사용한다.
4. 디바이스가 우휴 상태(스크린이 꺼지거나 충전중)가 되면, Hot code는 Aot컴파일러에 의해 다시 컴파일 된다. 
5. 앱을 실행하면, .oat 바이너리에 있는 코드가 더 좋은 퍼포먼스로 즉시 실행된다. .oat 바이너리에 코드가 없다면, 다시 1의 과정을 반복한다.

![image](https://user-images.githubusercontent.com/19990905/97287268-b918e480-1887-11eb-9cd8-b823c1077aa5.png)

평균적으로 앱의 80프로를 최적화하는데 약 8번의 앱 실행이 필요했다.
더 나은 최적화가 있다. 유사한 다바이스들 사이 컴파일 과정을 공유하는 것이다.
디바이스가 유휴 상태이거나 와이파이에 연결되어 있다면, 구글플레이 서비스들을 통해 컴파일 과정 파일들을 공유하는 것이다.
나중에, 똑같은 디바이스에 다른유저가 플레이 스토어에서 앱을 다운로드하면, 디바이스는 이러한 컴파일 과정들을 받아서 aot가 컴파일을 수행하도록 한다. 결과적으로 처음부터 최적화된 앱을 사용한다.

## 그래서 r8과 어떻게 연결되는 것일까?

구글은 런타임 컴파일 과정에 많은 개선을 이뤘다. 하지만 여전히 Dalvik/ART 컴파일러에 의해 지원하는 제한된 수의 opcode를 가지는 .dex세상에 살고 있다(달빅은 jvm이 아니고 달빅 바이트코드는 jvm바이트코드와는 다르기 때문에).

자바 7,8,9에서 소개된 새로운 언어 기능들은 안드로이드에서 실현되지 않았다. 비교적 최근까지 안드로이드는 자바6 세상에 살았다.
자바8을 마침내 실현시키기 위해, 구글은 desugaring(=변환)을 사용했다. 즉, 컴파일 시간 동안 자바8 기능을 Davik/ART가 인식할 수 있는 무엇인가로 변환했다 **(바이트 코드 바꿔치기)**. 하지만 이는 더 긴 빌드 시간을 가져왔다.

![image](https://user-images.githubusercontent.com/19990905/97774888-f5f21d80-1b9e-11eb-93a3-35150383babd.png)

위는 java .class desugar transformation 과정이다.


## Dope8

이러한 문제 해결을 위해, 안스 3.2에서 D8 이라고 불리는 덱스 컴파일러로 교체했다. desugaring transformation을 제거하고 .class2dex 컴파일 과정의 일부로 만들어서 더 빠른 빌드 시간을 만드는 이루는것이다.


![image](https://user-images.githubusercontent.com/19990905/97774962-a2cc9a80-1b9f-11eb-8247-7864ebee1861.png)

이게 끝이아니다.

# android.enableR8 = true

r8은 d8의 스핀오프다. 둘은 똑같은 코드베이스를 공유한다. 그러나 r8 추가적인 문제들을 해결했다. r8은 d8과 같이 자바8 기능들을 Dalvik/ART에서 사용할 수 있게 했다. 뿐만 아니라...

# R8 helps to use correct opcodes

안드로이드의 가장 큰 페인은 파편화다. 많은 안드로이드 디바이스들이 있다. 최근 체크해봤을 때 2만개의 디바이스들이 있었다. 그것들 중 Jit 컴파일러 작동을 바꾸는 디바이스/제조사들도 있다. 결국, 그것들은 이상하게 작동한다.
r8의 가장큰 장점은 .dex의 최적화다. 특정 디바이스/api 지원에 필요한 opcodes만 남긴다. 아래와 같이

![image](https://user-images.githubusercontent.com/19990905/97775122-7dd92700-1ba1-11eb-9561-74379befe2d3.png)

# R8 replace Proguard?

프로가드는 빌드가 돌아가는 시간 동안 영향을 미치는 또 다른 변환이다. 이를 해결하기 위해, R8은 .class가 .dex 컴파일 되는 과정에서 프로가드가 하는 비슷한 작업들을 할 것이다(optimization, obfuscation, removing unused classes). 

![image](https://user-images.githubusercontent.com/19990905/97775168-f63fe800-1ba1-11eb-8d53-04c271ffa568.png)

r8은 프로가드가 아니다. 프로가드가 하는 일의 일부분만 지원하는 새로운 실험 도구다.
(나중에 읽어보기 : https://www.guardsquare.com/en/blog/proguard-and-r8)

# R8 is more kotlin friendly

불행히도, 코틀린이 만드는 바이트코드는 더 많은 instructions 들을 만들어 낸다 자바보다.

    fun mathLambda() {
        doSomething { n: Int -> n % 2 == 0 }
    }

    fun doSomething(numericTest: (Int) -> Boolean) {
        numericTest(10)
    }
    
    
![image](https://user-images.githubusercontent.com/19990905/97775260-a4e42880-1ba2-11eb-80b8-f81f9429a583.png)

그러므로, 코틀린은 자바보다 cpu소모를 더 많이 사용한다. 그러나 r8은 ART가 실행시키는데 필요한 instructions 수를 감소시켜준다.

![image](https://user-images.githubusercontent.com/19990905/97775287-d3fa9a00-1ba2-11eb-8c3b-8848c485a45a.png)

끝.


https://proandroiddev.com/android-cpu-compilers-d8-r8-a3aa2bfbc109








