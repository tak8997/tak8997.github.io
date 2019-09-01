# Android 'LaunchMode'

안드로이드 런치 모드에 대해서 정리해보자

### 그 전에 Task란
Task는 어플리케이션에서 실행되는 액티비티를 보관하고 관리하며 Stack형태의 연속된 Activity로 이루어짐.
각 앱마다 현재 사용하고 있는 컴포넌트들을 그룹화하여 관리하는 것이 Task이다.
여기서의 컴포넌트는 해당 앱에 속한 컴포넌트 일수도 있고, 다른 앱에 속한 컴포넌트 일수도 있다!!
안드로이드는 다른 안드로이드 컴포넌트에 접근할 수 있음. 현재 태스크에 끌어온다.


## 1.singleTop (Activity D에 적용)
1) Activity Stack 맨 위에 D가 있다고 가정
- D Activity 호출 시 onNewIntent 호출. 즉, 새로 호출 되지 않음

2) Activity Stack 맨 위에 C가 있다고 가정
 - D Activity 호출 시 onCreate 호출. 즉, 그냥 새로운 엑티비티 호출

3) Activity Stack 맨 위부터 C -> D .. 있다고 가정
 - D Activity 호출 시 onCreate호 호출. 즉, 그냥 새로운 엑티비티 호출
 
결국, singleTop은 호출하고자 하는 Activity가 Stack내에서 반드시 맨 위에 존재해야만 onNewIntent가 호출됨.
그렇지 않으면, 그냥 새로운 생성.


## 2.singleTask (Activity C에 적용)
1) Activity Stack 맨 위부터 D -> C .. 있다고 가정
 - C Activity 호출 시 onNewIntent 호출. 이 때, 위에 있는 D Activity를 Destroy시켜버린다.
 
2) Activity Stack 맨 위부터 B -> A .. 있다고 가정
 - C Activity 호출 시 onCreate 호출.
 
결국, singleTask에서는 Acitivty Stack내에 호출하고자 하는 Acitivty가 있으면, 무조건 onNewIntent가 호출


## 3.singleInstance (Activity E에 적용)
1) Acitivty Stack 맨 위부터 D -> C .. 있다고 가정
 - E Activity 호출 시 새로운 task를 하나 만듬.

2) 이어서, D Activity에서 F Activity 호출 가정
 - D Activity 위에 새로운 F Activity 쌓임
 - E Activity 여전히 새로운 태스크에 본인만 존재함.
 - E Activity 호출 시, onNewIntent호출 됨.
 
 결국, 가장중요한 것은 새로운 task를 할당해서 존재한다는 것. singleTask와의 차이임.
 
 
## 4.standard (Activity B에 적용)
그냥 새로 생성



글을 보면서 아래 블로그 그림을 되새기자
 
참조 : https://medium.com/@iammert/android-launchmode-visualized-8843fc833dbe

