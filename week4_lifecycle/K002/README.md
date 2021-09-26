# LifeCycle:

## LifeCycle Owner:

![Untitled](https://user-images.githubusercontent.com/51209390/134810507-4a4ff76a-81dc-43ce-b9aa-1cdc37cedf25.png)


- LifeCycleOwner에대해 안드로이드 공식 문서를 확인하면 다음과 같이 명시되어있다.
- LifeCycleOwner은 안드로이드 생명주기를 인식하고 있는 클래스로 Activity나 Fragment 내부에 별로 코드를 작성하지 않고 생명주기 변경에 대응할수 있게 해준다.
- LifeCycleOwner은 클래스에 LifeCycle이 있음을 나타내는 단일메서드 인터페이스이다. 이 인터페이스에는 클래스에서 구현해야하는 getLifeCycle() 메서드가 하나 있다.

```kotlin
public interface LifecycleOwner {
    /**
     * Returns the Lifecycle of the provider.
     *
     * @return The lifecycle of the provider.
     */
    @NonNull
    Lifecycle getLifecycle();
}
```

- 지원 라이브러리 26.1.0 이상의 Activity 및 Fragment 에서는 이미 LifecycleOwner 인턴페이스가 구현되어있다.

```kotlin
@SuppressWarnings("WeakerAccess")
    public enum Event {
        ON_CREATE,
        ON_START,
        ON_RESUME,
        ON_PAUSE,
        ON_STOP,
        ON_DESTROY,
        ON_ANY;
}
```

```kotlin
@SuppressWarnings("WeakerAccess")
    public enum State {

        DESTROYED,
        INITIALIZED,
        CREATED,
        STARTED,
        RESUMED;
        public boolean isAtLeast(@NonNull State state) {
            return compareTo(state) >= 0;
        }
    }
```

- LifeCycle은 Activity나 Fragment와 같은 구성요소의 수명 주기 상태 관련 정보를 포함하며 다른 객체가 이 상태를 관찰할수 있게 하는 클래스이다. LifeCycle은 두가지 기본 열거를 사용하여 연결된 구성요소의 수명 주기 상태를 추적한다.

![Untitled 1](https://user-images.githubusercontent.com/51209390/134810508-1d49b6e6-363b-48c4-849b-7cb20bcc6caa.png)


### 사용 예제:

```kotlin
class MainActivity : AppCompatActivity() {
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        lifecycle.addObserver(MyObserver(this))
        Log.d("Observer", "OnCreate Function of MainActivity")
    }

    override fun onResume() {
        super.onResume()
        Log.d("Observer", "OnResume Function of MainActivity")
    }
}
```

```kotlin
class MyObserver(var context: Context) : LifecycleObserver {

    //onCreate() 메서드와 연결해준다.
    @OnLifecycleEvent(Lifecycle.Event.ON_CREATE)
    fun onCreate(){
        Log.d("Observer", "OnCreate Function of Observer")
    }

    //onResume() 메서드와 연결해준다.
    @OnLifecycleEvent(Lifecycle.Event.ON_RESUME)
    fun onResume(){
        Log.d("Observer", "OnResume Function of Observer")
    }
}
```

- 위의 코드들처럼 LifecycleOwner에 이벤트를 링크해주는 것으로 LifecycleObserver를 통해 생명주기 변화를 감지할수 있다.

# ViewModel:

## ViewModel이란?

- ViewModel이란 Android Jetpack의 구성요소 중 하나로, 본래 ViewModel이라는 개발 디자인 패턴중 하나인 MVVM(Model, View, ViewModel) 디자인 페턴으로 부터 파생된것이다.
- Android Jetpack의 ViewModel은 수명 주기를 고려하여 UI관련 데이터를 저장하고 관리하도록 설계되어 있다. 예를 들어 화면 회전과 같이 구성을 변경할 때도 데이터를 유지하는 것이 가능하다.
- 즉, ViewModel은 View로 부터 독릭적이며, View(UI)에 필요한 데이터들만을 소유한다. 이를 통해 Activity나 Fragment 같은 UI Controller가 과도한 책임을 분담하여 클래스가 거대해지는 것을 방지할수있다. 이는 Controller의 유지보수, 재사용성 그리고 테스트 등을 용이하게 만들어 준다.

![Untitled 2](https://user-images.githubusercontent.com/51209390/134810509-6398f404-01e3-40bc-9529-bfcec5e37bb3.png)


- 위의 사진에서 확인 할수 있듯이 ViewModel이 View에 필요한 데이터들을 소유하여 화면의 구성이 변경되어도 데이터를 유지할수 있는이유는 ViewModel의 생명주기는 LifeCycle과 관계되어 있기 때문이다.
- ViewModel은 Activity가 최초 생성될 때 ViewModel을 인스턴스화 하여 생명주기를 함께 시작하게된다. 그리고 Activity과 완전히 종료되 기전까지 메모리에 남아있게된다.
- ViewModel의LifeCycle을 Fragment의 LifeCycle과 함께하도록 하는것도 가능하다.

## ViewModel의 생성 과정:

1) ViewModelProvider를 통해 ViewModel 인스턴스를 요청한다.

2) ViewModelProvider 내부에서는 ViewModelStoreOwner를 참조하여 ViewModelStore를 가져온다.

3) ViewModelStore에게 이미 생성된(저장된) ViewModel 인스턴스를 요청한다.

4) ViewModelStore가 적합한 ViewModel 인스턴스를 가지고 있지 않다면, Factory를 통해 ViewModel 인스턴스를 생성한다.

5) 생성한 ViewModel 인스턴스를 ViewModelStore에 저장하고 만들어진 ViewModel 인스턴스를 클라이언트에게 반환한다.

6)이미 ViewModelStore에 저장된 인스턴스를 요청할경우 1~3 반복.

![Untitled 3](https://user-images.githubusercontent.com/51209390/134810511-c157227d-c97b-4fa9-becc-4437003a34d7.png)

```kotlin
val viewModel = ViewModelProvider(this).get(MainActivityViewModel::class.java)
```

- ViewModel은 자신의 데이터가 유지되어야할 View(Activity, Fragment)를 알아야한다. 즉 뷰모델의 인스턴스를 요청할때, LifeCycle owner와 Class를 전달하는것으로 인스턴스를 얻을수 있다.

```kotlin
public ViewModelProvider(@NonNull ViewModelStoreOwner owner) {
        this(owner.getViewModelStore(), owner instanceof HasDefaultViewModelProviderFactory
                ? ((HasDefaultViewModelProviderFactory) owner).getDefaultViewModelProviderFactory()
                : NewInstanceFactory.getInstance());
    }
```

```kotlin
public interface ViewModelStoreOwner {
    /**
     * Returns owned {@link ViewModelStore}
     *
     * @return a {@code ViewModelStore}
     */
    @NonNull
    ViewModelStore getViewModelStore();
}
```

- ViewModelProvider은 ViewModelStoreOwner 인터페이스를 구현하는 인스턴스를 인자로 받는다. 즉, 이를 통해 Activity와 Fragment가 ViewModelStoreOwner 인터페이스를 구현함을 알 수 있고 View(Activity, Fragment) 인스턴스의 LifeCycle을 이용해 ViewModel을 제공함을 알수 있다.
- 또한 NewInstanceFactory.getInstance() 를 통해 ViewModelProvider가 ViewModelStoreOwner에 대해 singleton으로 만들어진다.

```kotlin
public <T extends ViewModel> T get(@NonNull String key, @NonNull Class<T> modelClass) {
        ViewModel viewModel = mViewModelStore.get(key);

        if (modelClass.isInstance(viewModel)) {
            if (mFactory instanceof OnRequeryFactory) {
                ((OnRequeryFactory) mFactory).onRequery(viewModel);
            }
            return (T) viewModel;
        } else {
            //noinspection StatementWithEmptyBody
            if (viewModel != null) {
                // TODO: log a warning.
            }
        }
        if (mFactory instanceof KeyedFactory) {
            viewModel = ((KeyedFactory) mFactory).create(key, modelClass);
        } else {
            viewModel = mFactory.create(modelClass);
        }
        mViewModelStore.put(key, viewModel);
        return (T) viewModel;
    }
```

- 인자를 하나 받는 get 메서드를 타고 들어가면 인자를 두개 받는 get 메서드가 나오게되는데 코드는 위와 같다.
- class로 부터 받은 canonicalName을 Key로 두고 ViewModel을 저장하는 Map인 ViewModelStore에 해당 key값에 대해 ViewModel 값을 저장한 다음 만들어진 ViewModel을 return 한다.

```kotlin
public class ViewModelStore {

    private final HashMap<String, ViewModel> mMap = new HashMap<>();

    final void put(String key, ViewModel viewModel) {
        ViewModel oldViewModel = mMap.put(key, viewModel);
        if (oldViewModel != null) {
            oldViewModel.onCleared();
        }
    }

    final ViewModel get(String key) {
        return mMap.get(key);
    }

    Set<String> keys() {
        return new HashSet<>(mMap.keySet());
    }

    /**
     *  Clears internal storage and notifies ViewModels that they are no longer used.
     */
    public final void clear() {
        for (ViewModel vm : mMap.values()) {
            vm.clear();
        }
        mMap.clear();
    }
}
```

- ViewModel은 같은 ViewModelStore에 대해 key - value 쌍으로 ViewModel을 저장한다.
- 즉, 같은 ViewModelStoreOwner(Activity, Fragment)에 대해 같은 이름의 ViewModel 클래스를 get 하면 같은 인스턴스가 반환된다.

## 구현:

```kotlin
class MainActivity : AppCompatActivity() {
    val text: TextView = findViewById(R.id.textView)
    val btn: Button = findViewById(R.id.button)
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        val viewModel = ViewModelProvider(this).get(MainActivityViewModel::class.java)

        btn.setOnClickListener {
            viewModel.addNumber()
            text.text = viewModel.number.toString()
        }
    }
}
```

```kotlin
class MainActivityViewModel: ViewModel() {

    var number = 0

    fun addNumber(){
        number++
    }
}
```

[[ViewModel] 2. ViewModel은 어떻게 저장되는가?](https://kotlinworld.com/87)

# LiveData:

## LiveData란?

- LiveData는 일반 클래스와 달리 생명주기를 인식하며 데이터를 담아둘 수 있는 데이터 홀더 클래스 이다.

## LiveData의 작동 방식:

- LiveData는 LifeCycleOwner가 Start나 Resume 상태에 다다르면 활성화 상태가 된다. 이때 활당된 값이 변할 경우 observe 메서드로 인해 참조하고 있는 Observer의 onChanged 메소드가 호출되며 이벤트를 전달하게된다.
- LifeCycleOwner가 Start나 Resume 상태가 아닐경우에는 비활성화 상태가 되어 이베트를 전달하지 않는다.
- LifeCycleOwner가 재시작 즉, 다시활성화 상태가 되면 자신이 들고이쓴 데이터를 바로 넘긴다. 다시 활성화 상태로 돌아왔을 때 특별한 호출을 따로 해주지 않아도, 가장 마지막인 최신 데이터를 갱신 해준다.
- LifeCycleOwner가 destroyed 되명 관찰자 또한 삭제된다.

## LiveData 사용 이점:

1) UI 데이터 상태의 일치 보장 : 

- 앱의 데이터 및 라이프 사이클이 변경될 때 마다 observer을 통해 데이터를 변경할 수있다.

2) 메모리 누수 없음: 

- 연결된 수명 주기가 끝나면 자동으로 삭제됨

3) 중지된 활동으로 인한 비정상 종료 없음: 

- 관찰자의 수명 주기가 비활성화 상태이면 관찰자는 어떤 Live Data 이벤트도 받지 않는다.

4) 수명주기를 수동으로 처리하지 않음: 

- 수명 주기의 변경을 자동으로 인식함으로 수동으로 처리할 필요가 없음

5) 최신 데이터 유지: 

- 수명 주기가 비활성화 일 경우 다시 활성화가 될때 새로운 데이터를 받는다.

추가예정...
