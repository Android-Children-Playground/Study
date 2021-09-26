# Lifecycle Component

# 들어가며

왜 수명 주기 인식 구성요소를 쓰는가?

일반적으로 액티비티나 프래그먼트의 수명 주기 콜백 메서드에서 작업하는 것과 차이점은 무엇인가?

수명 주기 인식 구성요소를 쓰면 어떻게 바뀌는가?

Lifecycle 라이브러리와 이를 사용하는 구성요소의 메커니즘을 알아보자.

# androidx.lifecycle:lifecycle-common

## LifecycleOwner

> A class that has an Android lifecycle. These events can be used by custom components to
> handle lifecycle changes without implementing any code inside the Activity or the Fragment.

`LifecycleOwner` 인터페이스는 `Lifecycle` 타입 객체를 반환하는 `getLifecycle` 메소드 하나만 가지는 인터페이스로 라이프사이클 가지는 커스텀 클래스라면 이 인터페이스를 구현해야 한다.

```java
public interface LifecycleOwner {
    @NonNull
    Lifecycle getLifecycle();
}
```

안드로이드의 액티비티나 프래그먼트는 이미 `LifecycleOwner`를 구현하는 방식으로 설계되어있다.

## Lifecycle

> Defines an object that has an Android Lifecycle.

한 가지 애매하다고 생각되는게 Lifecycle 클래스가 안드로이드의 라이프사이클과 동일한 의미를 가지는것은 아닌듯 싶다. `Lifecycle` 클래스를 액티비티나 프래그먼트 또는 자신만의 라이프사이클을 가지는 컴포넌트의 표현하기 위해 사용되는 클래스라고 보는것이 좋을 것 같다.

이후의 내용에서 Lifecycle 클래스에 대한 지칭은 영문으로, 안드로이드 라이프사이클에 대한 지칭은 국문으로 표기한다.

클래스 자체는 추상 클래스이며 인터페이스의 구현은 `LifecycleRegistry` 클래스에서 이루어진다.

`Lifecycle` 클래스의 인터페이스를 살펴보자.

```java
public abstract class Lifecycle {

    ...

    @MainThread
    public abstract void addObserver(@NonNull LifecycleObserver observer);

    @MainThread
    public abstract void removeObserver(@NonNull LifecycleObserver observer);

    @MainThread
    @NonNull
    public abstract State getCurrentState();

    public enum Event {
        ON_CREATE,
        ON_START,
        ON_RESUME,
        ON_PAUSE,
        ON_STOP,
        ON_DESTROY,
        ON_ANY;

        ...
    }

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
}
```

`Lifecycle` 클래스의 인터페이스로부터 알 수 있는 것은 `Lifecycle` 인스턴스에 대해 `addObserver` 또는 `removeObserver` 등으로 라이프사이클 이벤트를 관찰하는 옵저버를 등록할 수 있는 것과, `Lifecycle` 클래스는 안드로이드의 라이프사이클을 `Event`와 `State`라는 두 개의 `enum class`로 표현한다는 점이다.

여기서 `Event`는 라이프사이클을 그래프로 나타냈을 때 그래프의 간선(edge)에 해당하고, `State`는 그래프의 정점(node)에 해당한다.

라이프사이클 그래프는 다음과 같이 표현할 수 있다.

![https://developer.android.com/images/topic/libraries/architecture/lifecycle-states.svg?hl=ko](https://developer.android.com/images/topic/libraries/architecture/lifecycle-states.svg?hl=ko)

## LifecycleObserver

Lifecycle-aware 컴포넌트는 라이프사이클의 이벤트를 관찰하며 변화하는 이벤트에 맞춰 자신의 상태를 변경한다.

어떤 클래스가 라이프사이클을 관찰하려면 해당 클래스가 `LifecycleObserver` 인터페이스를 구현해야 한다.

`LifecycleObserver` 클래스 자체는 메소드가 정의되지 않은 마커 인터페이스로 단순히 해당 클래스가 라이프사이클을 인지할 수 있다는 것을 나타낸다.

```java
@SuppressWarnings("WeakerAccess")
public interface LifecycleObserver {

}
```

여기서 라이프사이클의 이벤트마다 수행할 옵저버를 설정하기 위해서는 옵저버에 해당하는 메소드에 어노테이션을 사용하여 관찰할 이벤트를 명시한다.

```java
class MyObserver : LifecycleObserver {
    @OnLifecycleEvent(Lifecycle.Event.ON_START)
    fun start() {
        Log.d("MyObserver", "start")
    }

    @OnLifecycleEvent(Lifecycle.Event.ON_STOP)
    fun stop() {
        Log.d("MyObserver", "stop")
    }
}
```

안드로이드의 공식 문서에서도 동일한 방식으로 설명하고 있지만 [레퍼런스](https://developer.android.com/reference/androidx/lifecycle/LifecycleObserver)에서는 조금 다르게 이야기한다.

레퍼런스에 따르면 `LifecycleObserver` 인터페이스를 직접적으로 구현하는 방식으로 사용해선 안되며, 대신 `LifecycleEventObserver` 또는 `DefaultLifecycleObserver` 인터페이스를 사용하라고 말한다.

여기서 만약 프로젝트가 Java 8 이상을 사용하고 있다면 `DefaultLifecycleObserver`가 권장된다고 말한다. 하지만 `DefaultLifecycleObserver`는 `lifecycle`의 기본 라이브러리에서는 제공하지 않기 때문에 사용을 위해선 다음 dependency를 추가해야 한다.

`implementation "androidx.lifecycle:lifecycle-common-java8:$lifecycle_version"`

라이프사이클에 대해 `DefaultLifecycleObserver`를 사용하는 예제를 찾기 어려운 것으로 보아 아직까지는 특별한 이유가 없다면 `LifecycleEventObserver`로도 충분하지 않을까?

`LifecycleEventObserver`와 `DefualtLifecycleObserver` 인터페이스의 차이는 라이프사이클의 이벤트마다 호출되는 콜백 함수가 구분되어 있는가 이다.

`LifecycleEventObserver`의 경우 `onStateChanged` 메소드 하나만 존재하며, 라이프사이클 이벤트에 따른 동작의 분기가 필요하다면 메소드의 `event` 파라미터를 사용하여 분기 처리를 해야한다.

```java
public interface LifecycleEventObserver extends LifecycleObserver {
    void onStateChanged(@NonNull LifecycleOwner source, @NonNull Lifecycle.Event event);
}
```

반면에 DefaultLifecycleObserver의 경우 라이프사이클 이벤트마다 호출되는 콜백 함수가 분리되어 있다.

```java
public interface DefaultLifecycleObserver extends FullLifecycleObserver {
    @Override
    default void onCreate(@NonNull LifecycleOwner owner) {
    }

    @Override
    default void onStart(@NonNull LifecycleOwner owner) {
    }

    @Override
    default void onResume(@NonNull LifecycleOwner owner) {
    }

    @Override
    default void onPause(@NonNull LifecycleOwner owner) {
    }

    @Override
    default void onStop(@NonNull LifecycleOwner owner) {
    }

    @Override
    default void onDestroy(@NonNull LifecycleOwner owner) {
    }
}
```

사용에 있어서 `LifecycleEventObserver`와 `DefaultLifecycleObserver` 두 인터페이스 중 어느 것을 구현하든 클래스 내에서 기존의 방식대로 어노테이션을 사용하는 함수는 동작이 무시된다. 다만 두 인터페이스가 함께 사용된다면 `DefaultLifecycleEvent`의 함수가 먼저 호출되고 이후에 `LifecycleEventObserver`의 `onStateChanged` 함수가 호출된다.

## Custom LifecycleOwner

`Lifecycle` 라이브러리를 사용하면 액티비티나 프래그먼트 외에도 자체적인 라이프사이클을 가지는 컴포넌트를 만들 수 있다. 다만 이 경우는 해당 클래스의 상태를 상황에 맞게 변경할 필요가 있다.

자체적인 라이프사이클을 가지는 클래스 또한 마찬가지로 `LifecycleOwner` 인터페이스를 구현한다.

`LifecycleOwner`가 반환하는 Lifecycle 타입의 인스턴스는 일반적으로 `Lifecycle` 클래스를 구현하는 `LifecycleRegistry` 클래스를 사용한다.

```kotlin
class MyOwner : LifecycleOwner {
    private val lifecycle = LifecycleRegistry(this)

    override fun getLifecycle(): Lifecycle {
        return lifecycle
    }
}
```

커스텀 클래스는 외부에는 `Lifecycle` 타입만 공개하여 옵저버를 등록, 해제하고, 내부적으로는 `LifecycleRegistry` 클래스를 사용하여 상태를 변경하며 옵저버의 콜백을 호출할 수 있다.

# androidx.lifecycle:lifecycle-livedata-core

## LiveData

`LiveData`의 개념 자체는 단순한 옵저버 패턴의 구현이다. 다만 라이프사이클을 인식한다는 점에서 일반적인 옵저버 패턴과 차이가 있다.

`LiveData`의 옵저버는 등록이 될 때 `LifecycleOwner`와 함께 등록이된다. 이후 LiveData의 데이터가 변경되면 함께 등록된 `LifecycleOwner`의 상태를 확인하고 active state인 경우에만 변경을 통지한다.

라이프사이클을 인식하고 있기 때문에 옵저버의 제거 또한 `LifecycleOwner`의 상태가 `DESTROYED`로 전환될 때 자동으로 제거한다.

`LiveData`의 동작 과정을 알아보기 전에 먼저 LiveData에서 사용하는 몇 가지 인터페이스와 클래스를 알아보자.

## Observer

`LiveData`의 데이터가 변경되었을 때 호출하는 콜백 메소드 하나만 정의된 인터페이스.

```java
public interface Observer<T> {
    void onChanged(T t);
}
```

## ObserverWrapper

`Observer` 인터페이스의 인스턴스에 대한 래퍼 클래스이자 옵저버의 상태와 관련된 몇 가지 기능을 가진다.

`activeStateChanged` 메소드에서는 옵저버의 활성 상태 여부를 변경하며, 비활성 상태에서 활성 상태로 전환되는 경우에만 옵저버에게 값의 변경을 전파하도록 요청한다.

```java
private abstract class ObserverWrapper {    final Observer<? super T> mObserver;    boolean mActive;    int mLastVersion = START_VERSION;    ObserverWrapper(Observer<? super T> observer) {        mObserver = observer;    }    abstract boolean shouldBeActive();    boolean isAttachedTo(LifecycleOwner owner) {        return false;    }    void detachObserver() {    }    void activeStateChanged(boolean newActive) {        if (newActive == mActive) {            return;        }        // immediately set active state, so we'd never dispatch anything to inactive        // owner        mActive = newActive;        changeActiveCounter(mActive ? 1 : -1);        if (mActive) {            dispatchingValue(this);        }    }}
```

## LifecycleBoundObserver

`ObserverWrapper`를 구현하는 클래스로 `LifecycleEventObserver`를 구현한다. 따라서 `LifecycleBoundObserver`는 `LiveData`의 변경을 관찰하는 옵저버이자, 라이프사이클 이벤트를 관찰하는 옵저버가 된다.

일반적으로 `LiveData`를 사용할 때 `observe` 메소드에 전달하는 값들이 `LifecycleBoundObserver`의 생성자 파라미터에 전달된다.

옵저버와 함께 전달된 `LifecycleOwner`의 라이프사이클을 관찰하며, 상태가 `DESTROYED`로 전환되면 옵저버를 제거한다. 그 외의 이벤트에 대해서는 활성 상태인지 체크한 값으로 `activeStateChanged` 메소드를 호출한다.

```java
class LifecycleBoundObserver extends ObserverWrapper implements LifecycleEventObserver {    @NonNull    final LifecycleOwner mOwner;    LifecycleBoundObserver(@NonNull LifecycleOwner owner, Observer<? super T> observer) {        super(observer);        mOwner = owner;    }    @Override    boolean shouldBeActive() {        return mOwner.getLifecycle().getCurrentState().isAtLeast(STARTED);    }    @Override    public void onStateChanged(@NonNull LifecycleOwner source,            @NonNull Lifecycle.Event event) {        Lifecycle.State currentState = mOwner.getLifecycle().getCurrentState();        if (currentState == DESTROYED) {            removeObserver(mObserver);            return;        }        Lifecycle.State prevState = null;        while (prevState != currentState) {            prevState = currentState;            activeStateChanged(shouldBeActive());            currentState = mOwner.getLifecycle().getCurrentState();        }    }    @Override    boolean isAttachedTo(LifecycleOwner owner) {        return mOwner == owner;    }    @Override    void detachObserver() {        mOwner.getLifecycle().removeObserver(this);    }}
```

## LiveData의 동작 과정

`LiveData`의 동작 과정을 설명하기에 앞서 여기서 설명하는 동작 과정은 `LiveData`의 `postValue`, `observeForever`, `onActivty`, `onInactive` 등 부가적인 기능은 제외하고, `LiveData`의 데이터를 변경했을 때 어떤 과정을 거쳐 옵저버에게 변경이 전달되는지에 대해서만 다루는 것을 밝힌다.

각각의 동작마다 어떤 작업이 호출되는지 살펴보자.

### 데이터의 변경

`LiveData`에서 데이터의 변경은 `setValue` 메소드를 통해 요청된다.

```java
public abstract class LiveData<T> {    ...    static final int START_VERSION = -1;    private volatile Object mData;    private int mVersion;    public LiveData(T value) {        mData = value;        mVersion = START_VERSION + 1;    }    public LiveData() {        mData = NOT_SET;        mVersion = START_VERSION;    }    @MainThread    protected void setValue(T value) {        assertMainThread("setValue");        mVersion++;        mData = value;        dispatchingValue(null);    }}
```

`LiveData`는 자체적으로 `mVersion`이라는 `int` 타입 필드를 관리한다. `mVersion`은 `LiveData` 객체가 생성될 때 -1로 초기화되며, 이후 LiveD`a`ta의 데이터가 변경될 때 마다 1씩 증가한다.

변경된 `mVersion`의 값은 옵저버에게 데이터를 전달하기 전에 사용된다.

`LiveData`의 옵저버 또한 `mLastVersion`이라는 필드를 가지고 있으며, `LiveData`는 옵저버에게 값을 전달하기 전에 옵저버의 `mLastVersion`과 자신의 `mVersion`을 비교하여 `mVersion`이 더 큰 경우에만 옵저버에게 변경된 값을 전달한다.

```java
public abstract class LiveData<T> {    ...    private void considerNotify(ObserverWrapper observer) {        // 먼저 옵저버가 active 상태인지 확인        ...        if (observer.mLastVersion >= mVersion) {            return;        }        observer.mLastVersion = mVersion;        observer.mObserver.onChanged((T) mData);    }}
```

이로 인해 옵저버가 활성, 비활성 상태를 반복하더라도 데이터의 변경이 없었다면 비활성 상태에서 활성 상태로 전환된 최초 1회만 데이터의 변경을 전달받을 수 있고, 이후의 전환에선 불필요한 전달을 방지할 수 있다.

### 옵저버 등록

`LiveData`에 대한 옵저버를 등록하는 과정은 `LifecycleOwner`와 옵저버를 `observe` 메소드에 전달하는 과정으로 요청된다.

```java
public abstract class LiveData<T> {    ...    private SafeIterableMap<Observer<? super T>, ObserverWrapper> mObservers =            new SafeIterableMap<>();    private volatile Object mData;    @MainThread    public void observe(@NonNull LifecycleOwner owner, @NonNull Observer<? super T> observer) {        assertMainThread("observe");        if (owner.getLifecycle().getCurrentState() == DESTROYED) {            // ignore            return;        }        LifecycleBoundObserver wrapper = new LifecycleBoundObserver(owner, observer);        ObserverWrapper existing = mObservers.putIfAbsent(observer, wrapper);        if (existing != null && !existing.isAttachedTo(owner)) {            throw new IllegalArgumentException("Cannot add the same observer"                    + " with different lifecycles");        }        if (existing != null) {            return;        }        owner.getLifecycle().addObserver(wrapper);    }}
```

`observe` 메소드가 호출되면 메소드에 전달된 파라미터를 사용하여 `LifecycleBoundObserver` 인스턴스를 생성한다. 이후 옵저버에 대한 validation을 수행하여 동일한 옵저버가 이미 등록되지 않은 경우에만 자체적으로 관리하는 옵저버 `Map`에 엔트리를 추가한다. 

최종적으로 `LifecycleOwner`의 라이프사이클에 같은 옵저버를 등록하는 것으로 메소드가 종료된다.

### 데이터 변경의 전달

`LiveData`에서 옵저버에 대한 알림은 두 가지 경우에 의해 트리거된다.

- 데이터의 변경
- 라이프사이클의 변경(비활성 상태에서 활성 상태로 진입하는 경우)

이때 데이터의 변경은 `LiveData`의 `setValue` 메소드에서 트리거되며, 라이프사이클의 변경은 `LifecycleEventObserver`를 구현하는 `LifecycleBoundObserver`의 `onStateChanged` 메소드에서 트리거된다.

두 경우 모두 변경을 전달하기 위한 진입점으로 `dispatchingValue` 메소드를 호출한다.

```java
public abstract class LiveData<T> {    private SafeIterableMap<Observer<? super T>, ObserverWrapper> mObservers =            new SafeIterableMap<>();    private boolean mDispatchingValue;    @SuppressWarnings("FieldCanBeLocal")    private boolean mDispatchInvalidated;    @SuppressWarnings("WeakerAccess") /* synthetic access */    void dispatchingValue(@Nullable ObserverWrapper initiator) {        if (mDispatchingValue) {            mDispatchInvalidated = true;            return;        }        mDispatchingValue = true;        do {            mDispatchInvalidated = false;            if (initiator != null) {                considerNotify(initiator);                initiator = null;            } else {                for (Iterator<Map.Entry<Observer<? super T>, ObserverWrapper>> iterator =                        mObservers.iteratorWithAdditions(); iterator.hasNext(); ) {                    considerNotify(iterator.next().getValue());                    if (mDispatchInvalidated) {                        break;                    }                }            }        } while (mDispatchInvalidated);        mDispatchingValue = false;    }}
```

메소드 내에서는 `mDispatchingValue`와 `mDispatchInvalidated` 필드를 사용하여 변경 전파 도중 다른 트리거가 발생했을 때 빠르게 변경을 전파한다.

이 부분에 대해서는 의문이 생기는 것이 데이터의 변경이나 라이프사이클의 변경이 어짜피 메인 스레드에서 강제되기 때문에 동시에 `dispachingValue` 메소드가 호출되는 경우가 있을까 싶다.

어찌됐든 `while`문 내에서는 파라미터에 따라 `if`문으로 분기하여 변경을 전파한다.

`initiator`가 `null`이 아닌 경우는 `LifecycleBoundObserver`에서 관찰중인 `Lifecycle`이 변경된 경우로, 이때는 해당 `Lifecycle`에 연관된 옵저버에 대해서만 변경을 전파한다.

`else`문은 `LiveData`의 데이터가 변경된 경우로 `LiveData`에 등록된 모든 옵저버에 대해 변경을 전파한다.

변경을 전파하기 위해 호출되는 `considerNotify` 메소드에서는 옵저버와 `Lifecycle`의 상태 및 `version`을 비교한 후 옵저버의 `onChanged` 콜백을 호출한다.

```java
private void considerNotify(ObserverWrapper observer) {    if (!observer.mActive) {        return;    }       if (!observer.shouldBeActive()) {        observer.activeStateChanged(false);        return;    }    if (observer.mLastVersion >= mVersion) {        return;    }    observer.mLastVersion = mVersion;    observer.mObserver.onChanged((T) mData);}
```

# ViewModel

