# Java8 lamda expression, method reference

부스트코스 프로젝트 3을 진행하며 , 애매하게 알고있는 개념에 대해 정리해보고자 합니다.
자바8 기능은 여러가지가 있지만, 그 중 람다표현식과 메소드 래퍼런스가 가장 많이쓰입니다.

먼저, 환경을 셋팅을 알아보겠습니다.

    android {
        
        ...
        compileOptions {
            sourceCompatibility JavaVersion.VERSION_1_8
            targetCompatibility JavaVersion.VERSION_1_8
        }
    }
   
이렇게 매우 간단히 설정할 수 있습니다.
java8 기능 중 언급한 2가지는 minSdk상관없이 사용할 수 있습니다.

## 람다식
다음으로, 프로젝트 소스코드를 일부 참조하여 '람다식'부터 설명해보도록 하겠습니다.

    public class MovieCommentViewModel extends ViewModel {

        public MutableLiveData<String> toast = new MutableLiveData<>();
        public MutableLiveData<Comment> comment = new MutableLiveData<>();
        ...
    }
    
이 2가지 livdata는 뷰 단에서 observe하고 있습니다. 이제 어디선가 저 2가지 livedata에 setValue() 할 것입니다. 그럼 observe하고 있는 뷰 단에서 이벤트를 받아서 처리하게 될 것 입니다. 그러면 뷰쪽을 보겠습니다.
    
    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
    
        viewModel.toast.observe(this, new Observer<String>() {
            @Override
            public void onChanged(String message) {
                Toast.makeText(this, message, Toast.LENGTH_SHORT).show();
            }
        });
    }
    
이렇듯 익명 클래스를 생성하고 매소드의 파라미터로 데이터를 받아서 처리를 해줍니다. 
Toast하나 띄우기 위해 너무 많은 코드들이 들어가 있습니다.
람다식을 이용해서, 이 부분을 줄여 볼 수 있습니다.

    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        
        viewModel.toast.observe(this, message -> {
            Toast.makeText(this, message, Toast.LENGTH_SHORT).show();
        });
        ..
    }

이렇게 표현이 가능합니다. 기존코드 보다 훨씬 간소화 된 것을 볼 수 있습니다. message파라미터를 받아서 void타입의 함수를 실행합니다.
여기서는 Toast메시지 딱 한줄만 실행시키고 있습니다. 그렇다면,

    viewModel.toast.observe(this, message -> Toast.makeText(this, message, Toast.LENGTH_SHORT).show());

이렇게 한 줄로도 표현이 가능합니다.

## 메소드 레퍼런스
이제는 메소드 레퍼런스에 대해서 설명해보겠습니다.
이번에는 결과부터 보고 가겠습니다. 위의 코드를 메소드 레퍼런스를 통해 분리해보면,


    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
    
        viewModel.toast.observe(this, this::showMessage);
        ..
    }
    
    private void showMessage(String message) {
        Toast.makeText(this, message, Toast.LENGTH_SHORT).show();
    }
    
이런식으로 처리해 볼 수 있습니다. 
기본적으로 메서드 명 앞에 ::를 붙여서 사용할 수 있습니다. showMessage메소드는 기존 String타입의 message 파라미터와 똑같이 맞춰줍니다. 그리고 구현체 역시 void타입으로 구현합니다. 즉, (string) -> () 타입입니다.
이렇게 매소드 래퍼런스를 사용하는 법은 간단히 알아봤습니다.

매소드 래퍼런스의 장점으로는
 - 기존 메서드 구현으로 람다 표현식을 만들 수 있다. 
 - 이때 명시적으로 메서드명을 참조 하게 됨으로서 가독성을 높일 수 있는 장점이 있다. 
정도 입니다.

솔직히, 여기서는 굳이 분리하는것이 번거로워 보입니다... 하지만, 실제 업무를 하다보면 매소드 래퍼런스를 통해 상당히 가독성을 향상시킬 수 있었음을 많이 느껴왔습니다.
특히, 데이터바인딩을 쓰면서, viewModel.onSomethingChanged() 이런식으로 



참고 : https://digitalbourgeois.tistory.com/48,
      https://multifrontgarden.tistory.com/126


