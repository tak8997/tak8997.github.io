## Generics

설명하기전에 배경지식으로 

1. class vs type
2. subclass vs subtype

에 대한 배경지식을 알아야 한다.
별로 어려운 내용이 아니므로 매우 간략히 정리한다.


## class vs type

    ╔═════════════╦═════════════════╦
    ║             ║ Class  ║ Type   ║
    ╠═════════════╬════════╬════════╬
    ║ String      ║ Yes    ║ Yes    ║
    ║ String?     ║ No     ║ Yes    ║
    ║ List        ║ Yes    ║ Yes    ║
    ║ List<String>║ No     ║ Yes    ║
    ╚═════════════╩════════╩════════╩

 
## subclass vs subtype
subclass가 성립하기 위해서는, 어떤 클래스를 상속해야한다. 예를 들어, Integer, Number가 있따고
가정해보자. Integer은 Number를 상속한다. 그러므로 Integer는 Number의 서브클래스 이다.

    val integer: Int = 1
    val number: Number = integer
    
integer은 number의 하위 타입이기 때문에 이것이 성립한다.

subtype의 예를 보자. Int는 Int?의 하위 타입이다. 그래서 Int?에 Int를 할당할 수 있다.


## Variance(Invariance, Covariance, Contravariance)

변성이란 타입 간의 범위 지정을 일컫는 말이다.
설명을 위해

    abstract class Animal(val size: Int)
    class Dog(val cuteness: Int): Animal(100)
    class Spider(val terrorFactor: Int): Animal(1)
    
이런 클래스가 존재한다고 가정한다.

1. Invariance(무공변)

invariance를 살펴보자. 기본적으로 자바, 코틀린에서 타입 범위를 따로 지정하지 않으면 모든 제네릭 클래스는 무공변이다. 
무공변이라는 것은 제네릭타입을 인스턴스화 할 때, 서로 다른 타입 인자가 들어가는 경우 인스턴스 타입 사이의 하위 관계가 성립하지 않으면
그 제네릭 타입을 무공변이라고 한다.

자바코드를 보자 

    List<Dog> dogList = new ArrayList<>();
    List<Animal> animalList = dogList; // compile error
    
컴파일 에러가 나오는 것을 볼 수 있다. 이렇듯 기본적으로 성립하지 않는다.
죽, List<Dog>을 List<Animal>에 할당 할 수 없다. 이를 무공변이라 한다.

Dog -> Animal
List<Dog> ->x Animal
    
    
2. Convariance(공변)

설명하기 전에 코틀린의 List는 immutable하다. 즉, 변경(추가, 삭제) 불가능 하다.
아래 코드를 보자.

    val dogList: List<Dog> = listOf(Dog(10), Dog(20))
    val animalList: List<Animal> = dogList
  
List가 변경 불가능 하기 때문에, 위의 코드는 컴파일 에러가 나지 않는다.
Dog은 Animal의 하위 타입이고,
List<Dog>은 List<Animal>의 하위 타입이다.
이렇게 타입 관계가 보존되는 것을 covariance(공변) 이라고 한다.
    
Dog -> Animal
List<Dog> -> List><Animal>
    

3. Contravariance(반공변)

동물들을 비교하고 싶은 상황이 생겼다고 가정하자.
interface Compare<T>와 그 메소드 compare(T item1, T item2)가 있다.
알다시피 어떤 아이템이 먼저 오게할지 결정해주는 메소드이다.
    
    val dogCompare: Compare<Dog> = object: Compare<Dog> {
      override fun compare(first: Dog, second: Dog): Int {
        return first.cuteness - second.cuteness
      }
    }
    
위와 같은 코드가 있고

    val animalCompare: Compare<Animal> = dogCompare // Compiler error
    
이런 코드를 만들었을 때, 컴파일 에러가 난다.
이 코드가 왜 성립하지 하면 안돼는지에 대한 매우 좋은 이유가 있다.
animalCompare에 spiders를 전달한다고 가정해보자. 에러가 날 것이다. 
dogCompare은 오직 dog의 귀여움을 측정하기 위한 클래스이기 때문이다. 


반면, 만약에 동물들을 전부 비교할 수 있는 매커니즘이 있다면,
dogs, spiders 모두에게 효과적일 것이다.
그런 것이 있다.

    val animalCompare: Compare<Animal> = object: Compare<Animal> {
      override fun compare(first: Animal, second: Animal): Int {
        return first.size - second.size
      }
    }
    val spiderCompare: Compare<Spider> = animalCompare // Works nicely!

컴파일 에러 없이 잘 돌아간다.
그리고 이를 통해 볼 수 있는 것은
Spider가 Animal의 하위 타입인데, 
Compare<Animal>이 Compare<Spider>의 하위 타입 이라는 것이다.
즉, 타입 관계가 역전 되었다!
이것을 contravariance(반공변) 이라고 한다.

Spider -> Animal
Compare<Animal> -> Compare<Spider>
    

    


