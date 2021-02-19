## Volatile

volatile 키워드는 변수를 '메인 메모리에 저장하라'를 의미합니다.
좀 더 구체적으로, 모든 read/write가 cpu cache가 아니라 메인 메모리에 저장되어야 합니다. 

이는 멀티 쓰레드 환경에서 가시성을 보장 합니다.
만약 cpu가 하나 이상이고 각각의 쓰레드가 그 cpu에서 실행되고 있을 수 있습니다.
그 의미는 각각의 쓰레드가 각각의 cpu cache에 변수를 카피할 수 있습니다.
이럴 경우 데이터 불일치를 겪을 수 있겠죠.
volatile을 써서 이러한 불일치를 해결하는 것 입니다.

하지만, volatile이 멀티쓰레드 환경에서 변수 접근에 대한 원자성을 보장 하지는 않습니다.
여전히 크리티컬 섹션에 synchronize 같은 키워드 사용을 통해 배타적 사용을 보장해야 합니다.

또한, volatile은 메인 메모리에서 read/write하기 때문에, cpu cache에서 read/write할 때 보다 상대적으로 더 오래 걸립니다.
이러한 측면을 잘 고려해서 사용해야 할 것 입니다.


참고자료 : 
http://tutorials.jenkov.com/java-concurrency/volatile.html
