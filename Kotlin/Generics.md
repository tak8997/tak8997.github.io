## Generics

설명하기전에 배경지식으로 
1. class vs type
2. subclass vs subtype
에 대한 배경지식을 알아야 한다.
별로 어려운 내용이 아니므로 매우 간략히 정리한다.

1. class vs type
 
 
    ╔═════════════╦═════════════════╦
    ║             ║ Class  ║ Type   ║
    ╠═════════════╬════════╬════════╬
    ║ String      ║ Yes    ║ Yes    ║
    ║ String?     ║ No     ║ Yes    ║
    ║ List        ║ Yes    ║ Yes    ║
    ║ List<String>║ No     ║ Yes    ║
    ╚═════════════╩════════╩════════╩
 
 
2. subclass vs subtype
subclass가 성립하기 위해서는, 어떤 클래스를 상속해야한다. 예를 들어, Integer, Number가 있따고
가정해보자. Integer은 Number를 상속한다. 그러므로 Integer는 Number의 서브클래스 이다.

    val integer: Int = 1
    val number: Number = integer
    
integer은 number의 하위 타입이기 때문에 이것이 성립한다.

subtype의 예를 보자. Int는 Int?의 하위 타입이다. 그래서 Int?에 Int를 할당할 수 있다.


## Variance(Invariance, Covariance, Contravariance)

변성이란 타입 간의 범위 지정을 일컫는 말이다.

1. Invariance(무공변)

invariance를 살펴보자. 기본적으로 자바, 코틀린에서 따로 지정하지 않으면 모든 제네릭 클래스는 무공변이다. 
무공변이라는 것은 제네릭타입을 인스턴스화 할 때, 서로 다른 타입 인자가 들어가는 경우 인스턴스 타입 사이의 하위 관계가 성립하지 안흐면
그 제네릭 타입을 무공변이라고 한다.

자바에서 


