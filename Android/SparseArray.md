# SparseArray

종종 안드로이드 스튜디오에서 HashMap말고 SparseArray를 쓰라고 권장을 한다. 왜 그럴까?
SparseArray는 HashMap보다 메모리 효율적으로 설계되었기 때문이다.

기본적으로 Map은 '키' 가 객체가 되어야 한다. 좀 더 과정을 자세히 살펴보면,
'키-값'쌍이 삽입 될 때, 키의 해시코드가 계산이 된다(그 이유인 맵 기본자료 구조에 관한 설명은 생략..). 
그리고 그 해시코드가 HashMap.Entry 클래스에 저장 된다.

내부를 살펴보면 HashMap.Entry 라는 innerClass를 갖는다.
그리고 그 안에는 Entry<K, V>[] table; 가 있다. 즉, Key값에는 객체만 들어간다.

여기서 그 차이를 볼 수 있다. Map은 객체가 '키'로 들어가긴 때문에, auto-boxing이 들어간다. 
그 다음 키 값이 객체가 된다든 것은 더 많은 메모리를 소요한다.
ex) Integer -> 16bytes
    int -> 4bytes
  
SparseArray는 이러한 과정이 없다.  
그래서 장점을 보자면
1. 더 적은 메모리소요(primitive '키'를 쓰기 때문에)
2. auto-boxing 연산 과정 없음
3. 더 적은 Garbage Collection과정. auto-boxing이 없기 때문에, 중복적인 객체 타입의 키(Integer같은), Entry 객체가 없다.

장점을 먼저 적었는데, 그럼 어떻게 저장을 시키길래 map과 다른 저런 장점이 있을까?
위에서 설명은 안했지만, 내부적으로 Map은 해시 버킷을 위한 Array를 하나만 가지고 있다. 
반면, SparseArray는 해시를 위한 Array와 '키-값'쌍을 위한 Array 두개를 가지고 있다.
그리고 이 첫 번째 Array가 해시를 정렬된 상태로 저장한다. 그리고 두 번재 Array는 첫 번쨰 Array의 정렬된 상태를 따른다.
그리고, 우리가 아이템을 가져오려고 할 때, 내부적으로 binearySearch를 진행한다.

그래서 단점은
1. binearySearch를 진행한다는 것이다. 시간복잡도 O(logn)이라는 것이다.
2. SparseArray는 안드로이드에서만 사용 가능하다. 기존 Java SE에서는 사용 불가능.

참조 : 
https://android.jlelse.eu/app-optimization-with-arraymap-sparsearray-in-android-c0b7de22541a
https://gunhansancar.com/sparsearray-vs-hashmap/
https://riptutorial.com/android/topic/8824/how-to-use-sparsearray
