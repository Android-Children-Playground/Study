# Lifecycle

Activity나 Fragment와 같은 구성요소의 수명 주기 상태 관련 정보를 포함하며 다른 객체가 이 상태를 관찰할 수 있도록 지원하는 클래스

다른 객체가 상태를 observing 할 수 있도록 한다.

현재 상태를 나타내는 State와 상태를 변하게하는 Event로 이루어져 있다.

![Untitled](https://user-images.githubusercontent.com/50517813/134809779-44fd0ed6-5f70-471c-b0c8-d4e861024fba.png)

Lifecycle로부터 상태를 받아올 수 있는 메소드를 제공한다.

```java
abstract Lifecycle.State getCurrentState()
```

또한 새로운 생명 주기 옵저버를 추가하고 제거할 수 있는 메소드도 제공한다.

```java
abstract void addObserver(LifecycleObserver observer)

abstract void removeObserver(LifecycleObserver observer)
```

# LifecycleOwner

클래스가 Lifecycle를 가지고 있다는 것을 나타내는 단일 메서드 인터페이스

Fragment 및 AppCompatActivity와 같은 개별 클래스에서 Lifecycle의 소유권을 추출하고, 함께 작동하는 구성요소를 작성할 수 있게 한다.

관찰자가 관찰을 위해 등록할 수 있는 수명 주기를 소유자가 제공할 수 있으므로, LifecycleObserver를 구현하는 구성요소는 LifecycleOwner를 구현하는 구성요소와 원활하게 작동한다.

`getLifecycle()` 메소드를 제공하는데, Lifecycle 을 반환한다.

```java
public abstract Lifecycle getLifecycle()
```

# LifecycleObserver

Activity의 Lifecycle 메소드를 별도로 분리하여 Activity 코드를 간결하게 유지할 수 있도록 해주는 인터페이스

사용하는 Java 버전에 따라 Observer 사용법이 다르다.

Java 7를 사용한다면 annotation 방식을 사용한다.

@OnLifecycleEvent 를 사용하여 생명주기에 대응하는 컴포넌트를 관리할 수 있다.

```java
class TestObserver implements LifecycleObserver {
   @OnLifecycleEvent(ON_STOP)
   void onStopped() {}
 }
```

Java 8을 사용한다면 DefaultLifecycleObserver를 사용한다.

```java
class TestObserver implements DefaultLifecycleObserver { // default method 들을 가지고 있다.
     @Override
     public void onCreate(LifecycleOwner owner) {
         // your code
     }
 }
```

Android develop에 LifecycleObserver 레퍼런스를 확인하면, LifecycleObserver 인터페이스를 직접 구현하는 대신 `DefaultLifecycleObserver`나 `LifecycleEventObserver`를 대신 구현하라고 적혀있다. 아마도 Java 8이 android의 main stream이기 때문에 나중에 deprecate 될 것이기 때문이다.

## DefaultLifecycleObserver

LifecycleOwner 상태 변경 사항을 받기 위한 콜백 인터페이스

기본적으로 Lifecycle에 대한 콜백 메소드들을 구현하게끔 만들었다.

![Untitled](https://user-images.githubusercontent.com/50517813/134809804-32b42072-48e8-44f8-a5c2-772fe69c4d8a.png)

## LifecycleEventObserver

LifecycleOwner 상태 변경 사항을 받기 위한 인터페이스

`onStateChanged()` 라는 단일 메소드가 존재한다.

![Untitled](https://user-images.githubusercontent.com/50517813/134809821-449be0be-2ff4-48bd-a307-e1d611af4d12.png)

문서에서 DefaultLifecycleObserver와 LifecycleEventObserver를 동시에 구현하면 Default가 먼저 호출된 후 `onStateChange()`가 호출된다고 한다.

둘의 차이로는 생명 주기를 구분해둔 콜백 함수를 제공하느냐에 차이인 것 같은데, 굳이 구분지은 이유는 잘 모르겠다.

# LifecycleObserver 구현 및 등록

## 1. LifecycleObserver 구현

```kotlin
class MyObserver(private val lifeCycle: Lifecycle) : LifecycleObserver {
    companion object {
        const val TAG = "MyObserver"
    }

    @OnLifecycleEvent(Lifecycle.Event.ON_CREATE)
    fun onCreated(source: LifecycleOwner) {
        Log.d(TAG, "onCreated")
    }

    @OnLifecycleEvent(Lifecycle.Event.ON_START)
    fun onStart() {
        Log.d(TAG, "onStart")
        if (lifeCycle.currentState.isAtLeast(Lifecycle.State.INITIALIZED)) {
            Log.d(TAG, "currentState is greater or equal to INITIALIZED")
        }
        if (lifeCycle.currentState.isAtLeast(Lifecycle.State.CREATED)) {
            Log.d(TAG, "currentState is greater or equal to CREATED")
        }
        if (lifeCycle.currentState.isAtLeast(Lifecycle.State.STARTED)) {
            Log.d(TAG, "currentState is greater or equal to STARTED")
        }
        if (lifeCycle.currentState.isAtLeast(Lifecycle.State.RESUMED)) {
            Log.d(TAG, "currentState is greater or equal to RESUMED")
        }
        if (lifeCycle.currentState.isAtLeast(Lifecycle.State.DESTROYED)) {
            Log.d(TAG, "currentState is greater or equal to DESTROYED")
        }
    }

    @OnLifecycleEvent(Lifecycle.Event.ON_RESUME)
    fun onResume() {
        Log.d(TAG, "onResume")
    }

    @OnLifecycleEvent(Lifecycle.Event.ON_PAUSE)
    fun onPause() {
        Log.d(TAG, "onPause")
    }

    @OnLifecycleEvent(Lifecycle.Event.ON_STOP)
    fun onStop() {
        Log.d(TAG, "onStop")
    }

    @OnLifecycleEvent(Lifecycle.Event.ON_DESTROY)
    fun onDestroy() {
        Log.d(TAG, "onDestroy")
    }
}
```

기존에 Activity에서 사용하는 `onCreate()` 처럼 각 상태에 따른 메소드들을 만들고, Annotation을 붙여주었다.

Lifecycle 객체는 상태변화를 알려줄 때 annotation이 붙은 함수를 호출해준다. 코드가 컴파일될 때 annotation을 보고 호출해주는 코드가 자동으로 생성된다.

`State.isAtLeast` 는 상태를 점수로 표현하고 현재의 위치가 어떤 상태보다 큰지 Boolean으로 리턴해주는 함수이다.

## 2. 옵저버 등록

AppCompatActivity는 내부적으로 LifecycleOwner를 구현한다. 그렇기 때문에 Activity는 lifecycle 객체를 직접 참조할 수 있다.

```kotlin
class LifeCycleActivity : AppCompatActivity() {

    private var observer = MyObserver(lifecycle)

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_lifecycle)

        lifecycle.addObserver(observer)
    }
}
```

옵저버 생성시 lifecycle 객체를 인자로 넘겨준다. lifecycle을 전달해주면 옵저버에서 lifecycle에 접근하여 현재 상태를 알 수 있다.

옵저버를 생성하고, `lifecycle.addObserver` 로 옵저버 추가가 가능하다.

내부적으로 Activity 상태가 변경되면 Lifecycle로 전달되고 Lifecycle은 옵저버들에게 상태를 알려준다.

## 3. 결과 확인

### 1) Activity 실행

```
MyObserver: onCreated
MyObserver: onStart
MyObserver: currentState is greater or equal to INITIALIZED
MyObserver: currentState is greater or equal to CREATED
MyObserver: currentState is greater or equal to STARTED
MyObserver: currentState is greater or equal to DESTROYED
MyObserver: onResume
```

### 2) Activity 종료

```
MyObserver: onPause
MyObserver: onStop
```

# ViewModel

Activity나 Fragment의 UI 데이터를 준비하고 관리하는 클래스다. 또한 Activity, Fragment와 다른 응용프로그램과의 통신을 처리한다.

Android는 모바일 OS이기때문에 리소스에 대한 제약이 많다. 모바일 OS에서는 리소스를 제거해야만 하는, 제어될 수 없는 이벤트가 발생하는데 Android 프레임워크에서는 이러한 이벤트가 발생했을 때 Activity와 Fragment 같은 UI 컨트롤러에 대한 제거와 복구를 수행한다.(예시: 화면 회전)

Lifecycle에 따라 파괴되고 복구되어야 하는 데이터가 있을 때, 데이터를 저장했다가 꺼내와야하는 번거로움이 있다. 하지만 Activity가 살아있는 동안(생명 주기가 더 긴) 인스턴스가 있다면 굳이 파괴 전에 데이터를 저장하고 복원하는 과정을 거치지 않아도 된다.

번거로움을 해결하기 위해 View(Activity 혹은 Fragment)의 Lifecycle에 맞춰 Model(데이터)를 유지시키는 ViewModel이 만들어졌다.

### ViewModel의 사용

ViewModel 인스턴스를 만들기 위해서는 ViewModelProvider를 사용해야 한다.

```kotlin
class MainActivity: AppCompatActivity(), View.OnClickListener {
	val viewModel = ViewModelProvider(this@MainActivity).get(MainViewModel::class.java)
}
```

코드를 해석해보자면 ViewModelProvider의 파라미터로 MainActivity(View)를 전달하고 그로부터 ViewModel Class를 넣어 ViewModel을 get한다.

그럼 ViewModelProvider가 뭔지 더 알아보자

```java
public ViewModelProvider(@NonNull ViewModelStoreOwner owner) {
	this(owner.getViewModelStore(), owner instanceof HasDefaultViewModelProviderFactory 
		? ((HasDefaultViewModelProviderFactory) owner).getDefaultViewModelProviderFactory()
 : NewInstanceFactory.getInstance()); 
}
```

```java
public interface ViewModelStoreOwner { 
	.. 
	@NonNull 
	ViewModelStore getViewModelStore();
}
```

ViewModelProvider는 ViewModelStoreOwner 인터페이스를 구현하는 인스턴스를 인자로 받는다. 이를 통해 Acitivity와 Fragment가 ViewModelStoreOwner 인터페이스를 구현함을 알 수 있고 View(Activity, Fragment) 인스턴스의 Lifecycle을 이용해 ViewModel을 제공함을 알 수 있다.

```java
public <T extends ViewModel> T get(@NonNull String key, @NonNull Class<T> modelClass) { 
	ViewModel viewModel = mViewModelStore.get(key);
		..
	mViewModelStore.put(key, viewModel);
	return (T) viewModel; 
}

```

get 메소드는 class로부터 받은 canonicalName(정식이름)을 key로 두고 ViewModel을 저장하는 Map인 ViewModelStore에 해당 Key값에 대해 viewModel 값을 저장한 다음 만들어진 viewModel을 return한다. ViewModel은 같은 ViewModelStore에 대해 Key-Value 쌍으로 ViewModel을 저장한다.

### ViewModel의 생명주기

ViewModel 인스턴스의 Scope는 ViewModel을 가져올 때 ViewModelProvider에 전달되는 객체의 Lifecycle로 지정된다.

예를 들어 ViewModelProvider에 다음과 같이 MainActivity가 전달된다면, MainViewModel의 데이터는 MainActivity의 Lifecycle을 따른다.

![Untitled](https://user-images.githubusercontent.com/50517813/134809843-7c61956c-df11-47d1-b16a-12d4116a31af.png)

# 참고
https://codechacha.com/ko/android-jetpack-lifecycle/

https://kotlinworld.com/89?category=918952
