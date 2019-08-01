# android databinding viewmodel livedata


첫 블로그 글 입니다. 부스트코스 온라인 교육을 진행하며 학습한 내용을 공유하기 위해 시작하게 되었습니다. 이번 기회에, 블로그를 통해 학습한 것을 정리하고 공유하는 습관을 기르고자 합니다.



mvvm 아키텍처는 모바일 개발에서 가장 대중적으로 사용되는 아키텍처입니다. 안드로이드에서 mvvm을 구현하는 방법으로 databinding을 활용할 수 있습니다.

예전에 mvvm을 구현하는데 databinding을 사용하지 않고 대체자로 livedata, rx을 활용하여 구현하였었습니다. 최근에 databinding을 활용하게 되어서 공부하게 된 내용을 기록하고 차이점을 학습하기 위해 글을 쓰게 되었습니다.



최소사항

- Android 2.1(API레벨 7) 이상

- Android Plugin for Gradle 1.5.0-alpha1 이상

- Android Studio 1.3 이상


 먼저, 빌드 환경구성을 해보자.

    android {
        .......
        dataBinding {
             enabled = true
        }
    }

그 다음으로, 해당 뷰(여기서는 R.layout.activity_main)의 xml코드로 가본다.


보시는 바와 같이 최상단에 layout태그로 감싸게 되고 data와 view의 요소가 나온다. data태그 영역을 보면, viewmodel을 활용할 것이기 때문에, 위와 같이 선언되어 있다. 

viewmodel에 이벤트를 주면, 바인딩 된 xml에서 값을 읽어서 뷰를 업데이트 하게 된다. 코드를 보도록 하자.


위 코드를 보면 app:viewSelected="@{viewModel.like}", app:text="@{viewModel.like}" 부분을 볼 수 있다. 이 부분은 바인딩 된 뷰 모델로 부터 이벤트가 발생하면(viewModel.like, viewModel.hate),  그 값을 읽어들여 ui를 갱신 하는 것이다. 

 어떻게 갱신할 것인지 알아보기 위해 app:viewSelected를 조금 더 구체적으로 살펴보자. 


뷰에 셀력션을 주기 위해 이렇게 정의하였다. 이제 viewModel.like에 이벤트가 발생하면, 그 값을 읽어들여와서 뷰에 상태를 가해주는 것이다. 이 과정을 통해 추측해보면, BindingAdapter를 통해 내가 원하는 방식으로 뷰에 어떤 값을 전달해 줄 있는 것이다.

public MutableLiveData<Boolean> like = new MutableLiveData<>();


public void onClick(View view) {

        switch (view.getId()) {

                case R.id.img_like:

                like.setValue(!view.selected);

                break;

        }

}

뷰모델에서는 LiveData를 활용하여 구성하였다. 기존의 ObservableField를 사용하지 않은 이유는 LiveData를 활용하면 라이프싸이클 감지하는 효과를 볼 수 있기 때문이다. 그래서 얻게 되는 장점은 여기서 설명하지는 않겠습니다..

마지막으로, MainActivity를 가보자.


뷰모델과 데이터 바인딩이 제대로 동작하기 위한 코드이다. 위와 같이 xml에 layout 태그를 달아주고 다시 빌드를 해주면 위와 같이 ActivityMainBinding이 생성됩니다. 생성 방식은 layout file 이름에 기초합니다. 이렇게 함으로써, layout태그에 정의 된 변수들에 접근할 수 있습니다. 여기서 binding.setVariable()을 통해 뷰와 뷰모델을 set해주었는데, 이럴 필요 없이 binding.setView(), binding.setViewModel()을 통해 set해주셔도 됩니다. 

 또한, binding.setLifecycleOwner(this)를 잘 작성하셔야 뷰모델에서 이벤트를 줬을 때 값을 잘 받아옵니다(이걸 놓쳐서 삽질했던 기억이 좀 있어서..). 왜냐하면 저희는 ObservableField가 아닌 LiveData를 활용하고 있으니깐.



 이상으로 데이터 바인딩을 쓰면서 느꼈던 장점을 말해보자면

- 뷰쪽 자바(코틀린) 코드가 줄어들고 깔끔해진다. xml에 정의하므로

- findViewById 그만(코틀린 쓰면 이것도 사실 장점은 아닌거같지만..)

- 사용하기 어렵지 않다

반면 단점으로는

- xml쪽 코드가 많아진다

- 종종 디버깅하기 힘들어질 수 있다. (이것도 안드로이드 스튜디오가 많이 발전하면서 좋아진 것 같지만)

- 데이터 바인딩을 구현하기 위한 보일러 플레이트 코드들



정도로 정리해본다.





