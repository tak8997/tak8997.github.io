# AAC ViewModel Internal

이 글은 screen rotations과 같은 configuration change상황에서 어떻게 ViewModel이 data를 유지시키고 있는지 알아보는 글입니다.
(보시게 된다면, 안드로이드 스튜디오를 켜고 코드를 추적해가면서 보시길 추천드립니다.)

먼저 ViewModel 생성을 위한 ViewModelProvider의 생성자 코드를 살펴보자

![스크린샷 2021-02-05 오후 5 31 15](https://user-images.githubusercontent.com/19990905/108984058-da250100-76d2-11eb-9a3b-9d603b982a09.png)


파라미터를 보면 ViewModelStoreOwner라는 인터페이스가 있고, getViewModelStore()를 통해, ViewModelStore클래스를 리턴하는 것을 볼 수 있다.

그럼 ViewModelStore를 보자.

![image](https://user-images.githubusercontent.com/19990905/108984227-0771af00-76d3-11eb-9348-1af85a5cc67f.png)

해시 맵을 가지고 있는 클래스이다. 키는 DEFAULT_KEY + ":" + canonicalName이다. DEFAULT_KEY는 밸류는 ViewModel이다. 
**즉, Activity, Fragment는 각각 ViewModel를 유지하는 ViewModelStore를 가지고 있다는 의미이다.
(주석까지 꼼꼼히 읽어보기를 권장한다.)**


put()을 통해 ViewModel을 저장하는 부분이 있을 것 같다. put()을 호출하는 쪽을 보자.

![image](https://user-images.githubusercontent.com/19990905/108984399-37b94d80-76d3-11eb-9a96-e3a56f77e987.png)
(in ViewModelProvider.java)

mViewModelStore.put(key, viewModel) 코드를 볼 수 있다. 이 get() 메서드는 ViewModel 생성 방법인 ViewModelProvider(this)[SomeViewModel::class.java] 할 때 호출되는 코드이다. 

코드를 보면, ViewModelStore로 부터 주어진 키에 대한 ViewModel이 존재하는지 체크한다. 존재하면 리턴하고, 없다면 Factory인스턴스를 통해 새로운 ViewModel을 만든다. 그리고 마지막으로 put()을 통해 ViewModelStore에 저장한다.


주석에서도 알 수 있듯이, **ViewModelStore이 configuration change상황에서 살아남기 때문에 데이터를 유지하고 있을 수 있는 것인데,** 그게 어떻게 가능한지 이제 살펴보도록 하자. 


ViewModelProvider 생성자 코드에서 봤던, owner.getViewModelStore()의 구현체를 보자(아래는 ComponentActivity 쪽의 구현체이다).
![image](https://user-images.githubusercontent.com/19990905/108984634-764f0800-76d3-11eb-9629-2c1cd306eb95.png)
(in ComponentActivity.java)

NonConfigurationInstances클래스를 볼 수 있는데, **이 클래스가 configuration change상황에서 살아남아 이전 ViewModelStore클래스를 전달해 줄 수 있는 역할을 한다.** configuration change상황에서 Activity가 재생성될 때, nc는 이전 ViewModelStore를 가지고 있는 것이다. 물론 Activity가 처음 생성될 때는 nc는 null이고 새로운 ViewModelStore이 생성된다.

</br>
</br>

이제, NonConfigurationInstances가 무엇인지? 어떻게 configuration change 상황에서도 살아남는지를 살펴보자.
![image](https://user-images.githubusercontent.com/19990905/108984739-9d0d3e80-76d3-11eb-9dd5-4fcbb605c3fe.png)
(in ComponentActivity.java)


![image](https://user-images.githubusercontent.com/19990905/108984834-b31aff00-76d3-11eb-9ff2-36d2bcb7af86.png)
(in Activity.java)

**NonConfigurationInstances(Activity.java)는 Activity가 재생성될 때, 안드로이드 시스템에 의해 유지되는 객체이다. NonConfigurationInstances(Activity.java)는 activity라는 프로퍼티를 가지고 있는데, 이는 NonConfigurationInstances(ComponentActivity.java) 인스턴스이다.**
그리고 NonConfigurationInstances(ComponentActivity.java)가 ViewModelStore를 가지고 있는 것을 볼 수 있다.

</br>
</br>

![image](https://user-images.githubusercontent.com/19990905/108984981-d9409f00-76d3-11eb-821e-f35baa839383.png)
(in Activity.java)

![image](https://user-images.githubusercontent.com/19990905/108985031-eb224200-76d3-11eb-80b4-e529e4444b09.png)
(in ComponentActivity.java)

Activity가 재생성될 때마다, onRetainNonConfigurationInstance()가 호출되는데, **이전의 NonConfigurationInstances(ComponentActivity.java) 인스턴스를 리턴한다(getLastNonConfigurationInstance()의 mLastNonConfigurationInstances.activity).** 


</br>
</br>


NonConfigurationInstances(Activity.java)인 mLastNonConfigurationInstances이 어떻게 Activity재성 시에 전달받는지에 대한 대답은 Activity.java의 attach() 메서드에 있다. 

![image](https://user-images.githubusercontent.com/19990905/108985138-09883d80-76d4-11eb-8c4f-b910c0d2fe84.png)
(in Activity.java)

![image](https://user-images.githubusercontent.com/19990905/108985202-1efd6780-76d4-11eb-8d06-91b0067c6335.png)
(in Activity.java)

Activity가 재생성 후에 attach() 메서드를 통해 NonConfigurationInstances(Activity.java)를 전달받는 것이다.

</br>
</br>


마지막으로, 하나만 더 알아보고 글을 마치자.
그렇다면, 왜 configuration change상황에서는 ViewModel이 살아남고 low memory 혹은 finish() 일 때는 못 살아남는 것일까?
ViewModelStore의 clear()를 추적해보면 알 수 있다. 

![image](https://user-images.githubusercontent.com/19990905/108985285-39cfdc00-76d4-11eb-991d-32e08a1dbd03.png)

이 clear()는 ComponentActivity 생성자의

![image](https://user-images.githubusercontent.com/19990905/108985330-45bb9e00-76d4-11eb-8044-d80973dcbc19.png)

이 부분을 통해서 호출된다. 여기 isChangingConfigurations()을 통해서 확인할 수 있다. **isChangingConfigurations() 본문을 보면, mChangingConfigurations(Activity.java) 프로퍼티가 있는데, 이를 통해 configuration change 상황인지 아닌지 판단할 수 있다.**

</br>
</br>


참고 : 

proandroiddev.com/the-curious-case-of-survival-of-viewmodel-afe074992fbc

blog.mindorks.com/android-viewmodels-under-the-hood

