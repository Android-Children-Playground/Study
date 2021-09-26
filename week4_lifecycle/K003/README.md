- LifecycleOwner와 LifecycleObserver의 동작 메커니즘
- ViewModel
  - ViewModel의 생성 과정
  - ViewModel의 생명주기가 더 긴 이유
  - ViewModel 관련 클래스들의 역할(ViewModelProvider, Factory, Store, ... )
- LiveData



### LifeCycleOwner

`LifecyclerOwner`는 `ComponentActivity`에서 구현하고 있는 인터페이스이다.

각종 라이브러리에서 `lifecycle`처리 할때 우리는 `LifecycleOwner`를 넘기고 `lifecycle` 처리를 한다.

```kotlin
@MainThread
public void observe(@NonNull LifecycleOwner owner, @NonNull Observer<? super T> observer) {
    assertMainThread("observe");
    if (owner.getLifecycle().getCurrentState() == DESTROYED) {
        // ignore
        return;
    }
    ...중략
```

```kotlin
public interface LifecycleOwner {
    @NonNull
    Lifecycle getLifecycle();
}
```

`LifecycleOwner`는 단순히 `Lifecycle`을 가져오는 메소드가지고 있는 인터페이스이다.

실질적으로 `lifecycle`관련된 처리는 `LifecycleRegistry`에서 처리하게 된다

그러면 `LifecyclerRegistry`를 살펴보자

```kotlin
LifecyclerRegistry.class
public LifecycleRegistry(@NonNull LifecycleOwner provider) {
    mLifecycleOwner = new WeakReference<>(provider);
    mState = INITIALIZED;
}
```

`lifecycleOwner`를 구현하는 객체를 `weekReferece`로 가지고 자신의 상태를 `INITIALIZED`로만들며 생성한.

그리고 외부에서 `lifecycle`에대한 콜백이 필요할때 `addObserver`를 통해 추가하는데 이 `LifecyclerObserver`를 상속한 클래스들은 나중에 `lifecycle`에 변경이 생길때마다 변경을 수신 받는다.

이때 변경을 받는 메소드를 지정하는 `OnLifecycleEvent` 어노테이션을 달아주는데

```kotlin
class LifecycleTest : LifecycleObserver {
    @OnLifecycleEvent(Lifecycle.Event.ON_CREATE)
    fun onCreate() {
    }
}
```

이렇게 지정한 메소드에 lifecycle이 변경될때마다 해당 메소드를 호출해준다.



### VIewModel의 생성과정

`ViewModel`도 `LifecycleOwner`와 비슷하게 `ViewModelStoreOwner`라는 인터페이스를 구현해야 사용할 수 있다.

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

`VIewModelStore`는 `ViewModel`을 `Map`형태로 가지고 있는데 특정한 `Key`를 가지고 `VIewModel`을 생성하고 가져올수 있다. 

![img](https://miro.medium.com/max/875/0*Io9CAKKPaZbZH1Q0.png)

이때 우리가 평소 `ActivityViewModel`을 생성해 `Fragment`에서 `ViewModel`을 공유하는 방식을 살펴 볼수 있는데 `ViewModelStore`에 접근 `ViewModel`을 가져올때 `Key`값으로 `class`의 `canonicalName`을 사용해 `ViewModel`을 가져오는 것을 볼수 있다

```kotlin
@MainThread
public <T extends ViewModel> T get(@NonNull Class<T> modelClass) {
    String canonicalName = modelClass.getCanonicalName();
    if (canonicalName == null) {
        throw new IllegalArgumentException("Local and anonymous classes can not be ViewModels");
    }
    return get(DEFAULT_KEY + ":" + canonicalName, modelClass);
}
```



`ViewModel`은 일반적인 객체의 생성과 달리 `ViewModelProvider`를 통해 생성하는데 

그 이유는 `ViewModel`은 `lifecycle`과 같이 동작해야하는데 임의로 `ViewModel`을 생성하게 된다면 `ViewModel`이 중복으로 생성되거나 데이터의 손실이 발생할 수 있기 떄문에 Google은 `ViewModelProvider`같은 방식으로 생성을 제한하고 관리하도록 설계 한 것이다.

그래서 `ViewModel`을 생성할 때 `Fragment`때와 비슷하게 기본 생성자만 사용해야 하는 것이고 내부적으로 리플렉션을 통해 `ViewModel`을 생성한다

그래서 다양한 생성자를 가지는 `ViewModel`을 만들기 위해서는 이를 자동화 해주는 Koin을 사용하거나  `ViewModelFactory`같은 클래스를 상속해 이를 처리해주는 로직이 필요하게 된 것이다.



 ### ViewModel 수명주기

![활동 상태 변경에 따라 ViewModel의 수명 주기를 설명합니다.](https://developer.android.com/images/topic/libraries/architecture/viewmodel-lifecycle.png?hl=ko)

일반적은 `ViewModel`의 생명주기는 `lifecycle`을 따라 간다고 한다, 그런데 `configChange`의 경우 `Activity`의 `onDestroy`까지 호출되는데 `ViewModel`은 이떄는 초기화 되지 않는다 어떻게 구분하는 걸까?

이는 간단하게 `onDestroy`가 호출될 때 `isChangingConfigurations`라는 메소드를 통해 `onDestroy`가 호출 될때 `configchange`인지 확인하고 실제 `Activity`의 종료인 `onDestroy`일때만 `viewModel`의 `를 호출하고 `ViewModel`은 메모리에서 제거된다.

```kotlin
getLifecycle().addObserver(new LifecycleEventObserver() {
    @Override
    public void onStateChanged(@NonNull LifecycleOwner source,
            @NonNull Lifecycle.Event event) {
        if (event == Lifecycle.Event.ON_DESTROY) {
            if (!isChangingConfigurations()) {
                getViewModelStore().clear();
            }
        }
    }
});
```

