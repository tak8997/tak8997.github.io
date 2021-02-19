# Dagger2 Multibindings

부스트코스 프로젝트 개발을 하면서 dagger2를 적용해 보았습니다. 본래 알고있었지만 어려웠던 부분이나 애매한 부분을 중심으로 다시 정리해보고자 글을 쓰게 됬습니다. 
특히, dagger2 multibindings부분을 중심으로 글을 정리해보겠습니다. 또한, Architecture Components ViewModel과 함께 정리할 것입니다.

본격적인 내용에 앞서 사전지식으로 aac viewmodel을 사용하여 생성자에 데이터를 전달하려면, ViewModelProvider.Factory의 create() method를 활용하여 전달해야 합니다.

    viewModel = ViewModelProviders.of(this, object : ViewModelProvider.Factory {
        override fun <T : ViewModel?> create(modelClass: Class<T>): T {
            return MovieViewModel(movieRepository) as T
        }
    }).get(getModelClass())
        
요런식으로요.


<br/>
<br/>


이제 dagger2를 활용해 뷰모델을 초기화하는 과정부터 살펴보겠습니다. 
제 BaseActivity 에는 요런 코드가 있습니다.

    abstract BaseActivity<VB : ViewDataBinding, VM : ViewModel>: AppCompatActivity() {
    
        @Inject
        lateinit var viewModelFactory: ViewModelProvider.Factory
        lateinit var viewModel: VM
    
        override fun onCreate(savedInstanceState: Bundle?) {
            AndroidInjection.inject(this)
            super.onCreate(savedInstanceState)

            viewModel = ViewModelProviders.of(this, viewModelFactory).get(getModelClass())
        }
    }
 
viewModelFactory를 inject해주는 AppModule를 가보겠습니다.
    
    interface AppModule {
        ...
        
        @Binds
        fun bindsViewModelFactory(viewModelFactory: BaseViewModelFactory): ViewModelProvider.Factory
    }

그럼 BaseViewModelFactory가 inject될 것 입니다. 그럼 거기서 BaseActivity를 상속하고 있는 Activity의 ViewModel이 리턴되어야 합니다.
그 원리를 BaseViewModelFactory부분을 통해 알아 볼 수 있습니다.

    class ViewModelFactory @Inject constructor(
            private val creators: Map<Class<out ViewModel>, @JvmSuppressWildcards Provider<ViewModel>>
    ) : ViewModelProvider.Factory {

        override fun <T : ViewModel> create(modelClass: Class<T>): T {
            var creator: Provider<out ViewModel>? = creators[modelClass]
            if (creator == null) {
                for ((key, value) in creators) {
                    if (modelClass.isAssignableFrom(key)) {
                        creator = value
                        break
                    }
                }
            }
            if (creator == null) {
                throw IllegalArgumentException("unknown model class " + modelClass)
            }
            try {
                @Suppress("UNCHECKED_CAST")
                return creator.get() as T
            } catch (e: Exception) {
                throw RuntimeException(e)
            }
        }
    }

먼저, 생성자 쪽의 Map<Class<out ViewModel>, @JvmSuppressWildcards Provider<ViewModel>>부분을 보면, 
ViewModel을 상속하는 Class가 Key,
Provider<ViewModel>(provider : 의존성 주입 클래스를 제공하고 이를 인스턴스화 할 수 있는 Dagger2 특정 클래스)이 Value 형태로 되어있습니다.
이제 create() 메소드를 통해 Activity, Fragment에서 호출한 ViewModel타입이 리턴 됩니다. 여기서 어떻게 ViewModel 타입이 결정 될 수 있을까요??

<br/>

답은 dagger2가 제공하는 Provider 덕분입니다. dagger2는 주어진 키에 Provider를 연결하고 Map에 연결해 줄 수 있습니다. 이게 가능한 이유는
@IntoMap 어노테이션 덕분입니다.
@IntoMap은 키와 연결된 값을 리턴시키는 메소드에 어노테이션을 붙여서 활용됩니다. 즉, MainModule쪽을 봐보면

    @Binds
    @IntoMap
    @ViewModelKey(MainViewModel::class)
    fun bindsMainViewModel(viewModel: MainViewModel): ViewModel
    
이런식으로 되어있습니다. @IntoMap을 통해 MainViewModel을 감싸는 Provider를 Value로써 갖을 수 있습니다.
반면, @MapKey라는 어노테이션도 볼 수 있습니다. 커스텀 어노테이션인 @ViewModelKey 부분을 살펴보면

    @Retention(AnnotationRetention.RUNTIME)
    @Target(AnnotationTarget.FUNCTION)
    @MapKey
    annotation class ViewModelKey(val value: KClass<out ViewModel>)
    
이렇게 되어있는데, @MapKey는 'KClass<out ViewModel> 객체를 키 타입으로 한다' 정도로 알면 되겠습니다.
여기서는 MainViewModel.class가 Key로 Provider<MainViewModel>이 Value가 될 것 입니다.

<br/>

그럼 이제 다시 ViewModelProvider.Factory.create() 메소드를 자세히 관찰해 볼 수 있습니다. create(modelClass: Class<T>) 의 modelClass
를 통해서 우리는 Provider객체를 가져올 수 있습니다.

    var creator: Provider<out ViewModel>? = creators[modelClass]
    
이렇게요. 만약에 Provider가 존재하지 않는다면 해당 ViewModel의 하위 클래스에 Provider객체가 존재하는지를 찾습니다.
그게 아래 부분입니다.

    if (creator == null) {
        for ((key, value) in creators) {
            if (modelClass.isAssignableFrom(key)) {
                creator = value
                break
            }
        }
    }
  
이 과정 후에도 찾지 못한다면

    if (creator == null) {
        throw new IllegalArgumentException("unknown model class " + modelClass);
    }
    
요 부분이 실행되어 exception을 던집니다.
그리고 값이 존재한다면, 마지막으로

    try {
        return (T) creator.get();
    } catch (Exception e) {
        throw new RuntimeException(e);
    }
    
이 부분으로 와서 Provder의 get()을 통해 우리가 원하는 ViewModel 타입을 얻을 수 있습니다.

여기까지 설명을 마무리합니다. 내용이 쉽지 않아서 블로그 글도 리팩토링이 좀 더 필요할 거 같습니다.

















