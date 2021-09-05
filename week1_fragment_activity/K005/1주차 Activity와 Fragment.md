

# 1주차 Activity와 Fragment

# Activity

안드로이드 애플리케이션의 구성요소(Activity, BroadcastReceiver, Service, ContentProvider) 중 하나.

일반적으로 하나의 Activity는 하나의 화면을 나타낸다.

사용자가 애플리케이션과 상호작용하기 위한 entry point. 콘솔 애플리케이션 처럼 `main` 함수를 사용하지 않는 대신 lifecycle에 해당하는 함수 마다 수행할 작업을 정의한다. lifecycle 함수는 Android System에 의해 호출된다. 즉, Activity 객체는 Android System에 의해 관리된다.

## 상호작용

안드로이드에서 Activity 사이의 상호작용은 일반적인 코틀린 객체처럼 할 수 없다.

```kotlin
val activity = FooActivity()
foo.doSomething()
```

Activity(안드로이드 구성요소) 사이의 상호작용은 `Intent`를 사용하여 이루어진다.

<img src=".\res\Untitled.png" alt="Untitled" style="zoom:80%;" />

출처: [https://developer.android.com/guide/components/intents-filters?hl=ko](https://developer.android.com/guide/components/intents-filters?hl=ko)

**Intent**: 안드로이드 구성요소 사이의 상호작용(메시지 전달)을 위해 사용되는 메시지 객체.

## Context

<img src=".\res\Untitled 1.png" alt="Untitled 1" />

Activity class hierarchy

공식 문서에서는 Context에 대해 다음과 같이 설명한다.

> 애플리케이션 환경의 전역적인 정보에 대한 인터페이스. Context 클래스 자체는 추상 클래스이며 구현 클래스는 안드로이드 시스템에 의해 제공된다. Context는 애플리케이션의 리소스와 클래스에 대한 접근과, Activity 실행과 Broadcast, intent 수신 등의 애플리케이션 레벨의 명령어를 제공한다.

Activity가 Context를 상속하기 때문에 Activity 내에서 getResource, startActivity, getSystemService 등의 메소드를 호출할 수 있으며, View(Widget) 또는 Toast 메시지를 생성할 때 Context 파라미터에 this를 전달할 수 있다.

하지만 Activity에서는 Context의 참조를 얻기 위해 this 뿐만 아니라 여러 방법을 제공한다. Activity에서 Context를 참조하는 방법은 다음과 같다.

- this
- getBaseContext()
- getApplicationContext()

안정적인 애플리케이션을 위해선 Context가 필요한 시점에 적절한 방법을 사용하여 Context를 제공할 수 있어야 한다. 그렇다면 각각의 메소드를 사용해 얻는 Context는 어떤 차이가 있을까?

BaseContext부터 알아보자.

### BaseContext

Activity에서 호출할 수 있는 getBaseContext 메소드는 Context 클래스가 아닌 ContextWrapper 클래스가 제공하는 메소드이다. 따라서 getBaseContext가 제공하는 Context를 이해하기 위해선 ContextWrapper를 봐야 한다.

앞에서 Activity가 Context를 상속하기 때문에 시스템 리소스나 메소드를 사용할 수 있다고 말했다. 하지만 실제로는 Activity 또는 그 위로 이어지는 부모 클래스에서 Context에서 제공하는 메소드들을 구현하지 않는다.

Context에 대한 구현은 ContextImpl이라는 클래스에서 제공한다. ContextImpl은 Context API의 일반적인 구현 방식을 정의한 클래스로, Activity 또는 다른 애플리케이션 구성요소에 base context object를 제공한다.

ContextWrapper는 바로 이 ContextImpl을 래핑하는 클래스로 Context API의 구현을 ContextImpl 에게 위임한다.

```java
@Override
public void startActivity(Intent intent) {
    mBase.startActivity(intent);
}

/** @hide */
@Override
public void startActivityAsUser(Intent intent, UserHandle user) {
    mBase.startActivityAsUser(intent, user);
}

/** @hide **/
public void startActivityForResult(
        String who, Intent intent, int requestCode, Bundle options) {
    mBase.startActivityForResult(who, intent, requestCode, options);
}
```

ContextImpl 객체는 안드로이드 시스템에 의해 생성되며 ContextWrapper의 생성자 또는 attachBaseContext 메소드에서 mBase 필드에 주입된다. Activity 에서 getBaseContext를 호출하면 ContextWrapper의 mBase가 반환된다.

아래의 다이어그램은 이러한 클래스들의 관계를 보여준다.

<img src=".\res\Untitled 2.png" alt="Untitled 2" style="zoom: 80%;" />

출처: [https://lotuslee.tistory.com/115](https://lotuslee.tistory.com/115)

### ApplicationContext

Context 로서의 Activity와 getApplicationContext를 통해 얻을 수 있는 Context의 차이점을 이해하기 위해선 앞서 말한 Context의 개념을 조금 확장할 필요가 있다.

[여기](https://medium.com/@ali.muzaffar/which-context-should-i-use-in-android-e3133d00772c#:~:text=I%20personally%20like,state%20for%20it.)에 따르면 Context를 다음과 같이 설명한다.

> 애플리케이션에서 특정 시점의 상태. ApplicationContext는 앱에서 전역적이거나 기본이 되는 base Configuration을 나타내며, Activity에서의 Context는 Activity-specific Configuration을 나타낸다.

Context가 단순한 인터페이스 뿐만 아니라 상태이며 범위를 가진다는것은 Context가 일종의 식별자로도 사용된다는 의미이다. Activity에서 View나 Intent를 생성할 때 Context로 this를 넘겨주는 것은 생성하는 View나 Intent가 다른 Context가 아닌 해당 Activity에 속한다는 것을 말한다.

따라서 Activity에서 Context를 사용하거나 전달할때는 Context가 사용되는 범위를 생각하고 사용해야 한다. ViewModel 처럼 Activity보다 생명주기가 긴 객체에서 ActivityContext를 참조하고 있다면 Activity가 종료되더라도 여전히 Context에 대한 참조가 남아있기 때문에 메모리 누수가 발생한다.

Context 사용 목적이 단순히 String이나 Color 등의 시스템 리소스에 대한 접근이라면 ActivityContext 대신 ApplicationContext를 전달할 수도 있다.

Toast의 경우 context 파라미터에 application context를 전달해도 동작한다. 이는 Toast가 특정 Activity의 윈도우에 속하지 않고 자신만의 윈도우를 생성하기 때문이다.

# Fragment

독립적인 라이프사이클과 상태를 가지며 대부분의 경우 UI와 연결된 모듈로, Activity를 분할할 수 있다.

독립적으로 존재할 수 없고 Fragment를 호스팅하는 Activity나 부모 Fragment가 필요하다.

대부분의 경우 FragmentManager에 의해 관리되지만 FragmentManager로부터 Fragment 인스턴스를 참조하여 직접 조작할 수도 있다.

Activity 보다는 좀 더 일반적인 코틀린 객체에 가깝다.

## 상호작용

### 직접 참조

Fragment 내에서 parentFragmentManager 또는 childFragmentManager를 사용하여 다른 Fragment 인스턴스의 참조를 얻을 수 있다.

```kotlin
val fragmentB = parentFragmentManager.findFragmentByTag("FragB") as? FragmentB
fragmentB?.doSomething()
```

이러한 방식은 Fragment를 직접 조작할 수 있지만 권장되진 않는다. 가장 큰 문제는 참조를 얻은 Fragment의 상태를 모른다는 점이다. FragmentB의 doSomething 메소드에서 Fragment의 View를 변경한다고 했을 때 메소드를 호출하는 시점에서 이미 FragmentB의 View가 파괴되었다면 NPE가 발생할 수 있다.

Fragment에서의 안전한 상호작용을 위해 Fragment 1.3.0 부터 Fragment Result API 가 도입되었다.

### Fragment Result API

Fragment 1.3.0-alpha04부터 각 FragmentManager는 FragmentResultOwner를 구현한다. 즉, FragmentManager가 Fragment에서 이벤트 전달을 위한 모더레이터 역할을 수행할 수 있다.

Fragment Result API를 사용한 상호작용은 Key를 사용한 이벤트 옵저빙 방식으로 동작한다.

```kotlin
override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    parentFragmentManager.setFragmentResultListener("requestKey", this) { requestKey, bundle ->
        val result = bundle.getString("bundleKey")
    }
}
```

```kotlin
button.setOnClickListener {
    val result = "result"
    parentFragmentManager.setFragmentResult(
			"requestKey", bundleOf("bundleKey" to result)
		)
}
```

Result 수신을 대기중인 Fragment는 STARTED 상태가 될 때 까지 Result를 수신하지 않는다. STARTED 상태로 변경되었을 때 리스너에 등록한 콜백을 수행한다.

<img src=".\res\Untitled 3.png" alt="Untitled 3" style="zoom:50%;" />

## Context

앞서 Activity의 계층 구조에서 보았듯 Activity는 Context를 상속하기 때문에 Activity == Context가 가능하다. 반면에 Fragment는 Context가 아니다. 대신 Fragment를 호스팅하는 Activity의 Context를 얻을 수 있다.

Fragment에서 Context를 얻는 방법은 두 가지가 있다.

- getContext()
- requireContext()

getContext 메소드와 requireContext 메소드는 반환 객체의 타입이 Nullable이냐에 따라 구분된다.

requireContext의 경우 먼저 getContext로 context 얻은 후 해당 context가 null 이라면 예외를 던지기 때문에 Fragment가 호스팅된 이후 사용해야 한다고 말한다.

하지만 실제로 Fragment의 라이프사이클 콜백마다 requireContext를 사용해 보면 `super.onAttach()` 이전이나. `super.onDetach()` 이후에도 문제없이 사용할 수 있다. 결국 Fragment의 라이프사이클 내에선 문제 없이 requireContext를 사용할 수 있다.

내부 코드를 통해 좀 더 자세히 알아보자.

```java
@Nullable
public Context getContext() {
    return mHost == null ? null : mHost.getContext();
}
```

Fragment의 getContext 메소드는 mHost의 context를 반환한다. mHost는 Fragment 클래스의 필드로 FragmentHostCallback 타입이다. FragmentHostCallback은 추상 클래스로, Activity, Context, Handler, FragmentManager 타입의 필드를 가진다.

```java
public abstract class FragmentHostCallback<E>extends FragmentContainer{
		@Nullable private final Activity mActivity;
    @NonNull private final Context mContext;
    @NonNull private final Handler mHandler;
    private final int mWindowAnimations;
    final FragmentManager mFragmentManager = new FragmentManagerImpl();

		...
}
```

Fragment의 getContext는 mHost의 mContext를 반환하지만, Fragment 객체의 생성 시점에서는 null로 초기화된다. 그렇다면 mHost는 언제 초기화되는 것일까?

mHost가 초기화 되는 코드를 추적해보면 FragmentStateManager의 attach, detach 메소드 내에서 객체가 할당되고 해제된다.

```java
class FragmentStateManager {
		...
		void attach() {
		    ...
		    **mFragment.mHost = mFragment.mFragmentManager.getHost();**
		    mFragment.mParentFragment = mFragment.mFragmentManager.getParent();
		    mDispatcher.dispatchOnFragmentPreAttached(mFragment, false);
		
		    **mFragment.performAttach();**
		
		    mDispatcher.dispatchOnFragmentAttached(mFragment, false);
		}
		
		void detach() {
		    ...
		    **mFragment.performDetach();**
		    mDispatcher.dispatchOnFragmentDetached(
		            mFragment, false);
		    mFragment.mState = Fragment.INITIALIZING;
		
		    **mFragment.mHost = null;**
		
		    ...
		}
		...
}
```

여기서 주의깊게 봐야할 부분은 mHost가 초기화되는 코드와 performXXX 메소드가 호출되는 코드의 순서이다.

Fragment의 onAttach, onDetach 메소드는 각각 performAttach, performDetach 메소드 내에서 호출된다. 하지만 mHost는 performAttach가 실행되기 전과 performDetach가 실행된 후에 초기화된다. 따라서 우리는 Fragment 내에서 onAttach 이전과 onDetach 이후에도 문제 없이 requireContext를 사용할 수 있다.

그렇다면 우리는 언제 requireContext를 사용해선 안될까? 당연한 말이지만 attach가 호출되기 이전과 detach가 호출된 이후이다.

코드를 이렇게 작성할 일은 없겠지만 만약 Fragment 객체가 생성되는 시점에서 필드를 초기화할 때 requireContext를 사용하거나, Fragment가 FragmentManager에 의해 add 또는 replace 되기 전 requireContext를 사용한다면 예외가 발생할 수 있다.

이에 비해 detach된 이후 requireContext를 사용하는 것은 생각보다 가능성 있는 상황이다.

Fragment에서 네트워킹 또는 비동기 작업을 수행하고 콜백 리스너 내에서 Context를 사용해야 하는 상황을 예로 들어보자. 작업의 시작은 Fragment가 호스팅되는 중에 이루어지지만 콜백이 호출되는 시점에도 Fragment가 여전히 호스팅 중이라고 보장할 순 없다.

이런 경우엔 requireContext 대신 getContext를 사용하고 null 처리를 하거나, Fragment가 파괴되는 시점에서 비동기 작업을 취소하는 등의 예외 처리가 필요하다.

# Activity vs Fragment

Fragment는 태블릿 등 다양한 화면 구성에 대응하고 재사용성을 높이기 위해 도입되었다. 하지만 우리는 태블릿용 앱이 아니더라도 단일 Activity에 여러 개의 Fragment를 사용하여 화면과 내비게이션을 구성하거나, 단일 Activity가 아니더라도 흐름상 큰 틀만 Activity로 구분하고 나머지는 Fragment를 사용하여 구성하는 방식을 자주 사용한다.

하지만 Fragment 없이 Activity만 사용하더라도 충분히 애플리케이션을 개발할 수 있다. 복잡한 화면 대응이 필요 없더라도 Fragment를 사용하는 이유는 무엇일까?

## Heavy Activity, Light Fragment

흔히 무거운 Activity에 비해 Fragment는 더 가볍다고 말한다. 물론 무겁고 가볍다는 것은 상대적인 기준이기 때문에 Activity가 무겁다고 한들 시작에 몇 초씩 소요되는것은 아니다.

그렇다면 왜 Fragment를 더 가볍다고 말하는 것일까?

이 차이를 알기 위해선 앞서 말한 Activity와 Fragment의 상호작용 과정을 살펴볼 필요가 있다.

Activity 사이의 상호작용은 항상 시스템이 개입한다. 새로운 Activity의 생성, 전환, 소멸 등의 과정은 상호작용하는 Activity를 직접적으로 참조하지 않고 Intent와 SystemService 등의 중간 과정이 필요하다.

Activity는 설계 단계부터 다른 프로세스에서 실행하는 것이 염두되었고 따라서 각각의 Activity는 서로의 메모리를 공유하지 않는다. 이로 인해 커널 레벨에서 프로세스간 통신이 필요해지고 퍼포먼스를 떨어뜨리는 원인이 된다.

이에 비해 Fragment는 호스팅하는 Activity 내에서 FragmentManager에 의해 관리된다.

FragmentManager를 사용하여 직접적인 객체의 참조를 얻을 수도 있고 메모리 영역을 공유할 수도 있다. 따라서 Activity보다 빠른 상호작용이 가능하다.

또한 Fragment는 객체 생성 과정에서 시스템의 보조가 필요하지 않다. Activity 처럼 상속 구조와 의존성이 복잡하게 꼬여있지 않기 때문에 좀 더 부담 없이 사용할 수 있다.

# Lifecycle

## Activity lifecycle

라이프사이클 각각에 대해선 이미 문서에 정리가 잘 되어있기 때문에 이에대한 내용은 생략하고 이 절에서는 특정 상황에서 라이프사이클이 어떻게 변하는지에 대해 다룬다.

**다른 Activity를 시작할 때**

다른 Activity를 시작하여 기존의 화면이 다른 화면으로 가려진다면 라이프사이클은 어떻게 호출될까?

편의상 기존의 Activity를 Caller Activity, 새로 시작되는 Activity를 Callee Activity라고 하자.

라이프사이클 그래프에 따르면 Caller Activity는 onPause, onStop 이 호출되고 Callee Activity는 onCreate, onStart, onResume이 호출된다. 하지만 이 메소드들이 호출되는 순서는 약간의 차이가 있다.

<img src=".\res\Untitled 4.png" alt="Untitled 4" />

실제로 로그를 찍어보면 알 수 있듯 Caller Activity의 onPause 이후 Callee Activity의 onCreate, onStart, onResume이 호출되고 Caller Activity의 onStop이 호출된다.

왜 이런 순서로 호출되는 것일까?

이를 이해하기 위해선 라이프사이클의 onPause와 onStop을 확인할 필요가 있다.

Activity에서 onPause가 호출되더라도 해당 Activity는 여전히 화면에서 일부라도 볼 수 있다. Activity의 전체 화면이 가려져 더 이상 보이지 않을 때 onStop이 호출된다.

이 말은 Activity의 onStop이 호출되려면 새로 떠오르는 화면이 얼마만큼의 영역을 차지하는지를 먼저 알 수 있어야 한다는 것을 의미한다. 따라서 Callee Activity의 onCreate, onStart, onResume까지 호출된 이후 새 화면의 영역이 정해지고 나서야 Caller Activity의 onStop이 호출된다.

그렇다면 다이얼로그 테마 Activity나 투명한 Activity 처럼 기존의 Activity가 일부라도 보이는 경우에는 라이프사이클이 어떤 순서로 호출될까?

앞에서 말했듯 Caller Activity의 onStop은 호출되지 않는다. 따라서 Caller Activity의 onPause 이후 Caller Activity의 onCreate, onStart, onResume 까지만 호출된다.

**기존의 Activity로 돌아올 때**

백 버튼 등을 눌러 기존의 Activity로 돌아올 때의 라이프사이클은 다른 Activity를 시작했을 때 라이프사이클의 반전된 결과를 보여준다.

<img src=".\res\Untitled 5.png" alt="Untitled 5" />

다른 Activity를 시작했을 때는 Caller Activity의 onPause가 먼저 호출되고 Callee Activity의 라이프사이클 이후 Caller Activity의 onStop이 호출되었지만, Activity를 종료할 때는 Caller와 Callee가 반전되어 Callee Activity의 onPause가 먼저 호출된 이후 Caller Activity의 onStart, onResume이 호출되고 나서야 Callee Activity의 onStop, onDestroy가 호출된다.

# 참고

[https://developer.android.com/guide/components/activities/intro-activities?hl=ko](https://developer.android.com/guide/components/activities/intro-activities?hl=ko)

[https://developer.android.com/guide/components/intents-filters?hl=ko](https://developer.android.com/guide/components/intents-filters?hl=ko)

[https://www.charlezz.com/?p=1080](https://www.charlezz.com/?p=1080)

[https://medium.com/@ali.muzaffar/which-context-should-i-use-in-android-e3133d00772c](https://medium.com/@ali.muzaffar/which-context-should-i-use-in-android-e3133d00772c)

[https://developer.android.com/guide/fragments/communicate#fragment-result](https://developer.android.com/guide/fragments/communicate#fragment-result)

[https://academy.realm.io/kr/posts/michael-yotive-state-of-fragments-2017/](https://academy.realm.io/kr/posts/michael-yotive-state-of-fragments-2017/)

[https://www.charlezz.com/?p=44128](https://www.charlezz.com/?p=44128)

[https://medium.com/박상권의-삽질블로그/지원자-95-가-틀리는-startactivity-라이프사이클-당신도-예외는-아닙니다-ed0947a48d6](https://medium.com/%EB%B0%95%EC%83%81%EA%B6%8C%EC%9D%98-%EC%82%BD%EC%A7%88%EB%B8%94%EB%A1%9C%EA%B7%B8/%EC%A7%80%EC%9B%90%EC%9E%90-95-%EA%B0%80-%ED%8B%80%EB%A6%AC%EB%8A%94-startactivity-%EB%9D%BC%EC%9D%B4%ED%94%84%EC%82%AC%EC%9D%B4%ED%81%B4-%EB%8B%B9%EC%8B%A0%EB%8F%84-%EC%98%88%EC%99%B8%EB%8A%94-%EC%95%84%EB%8B%99%EB%8B%88%EB%8B%A4-ed0947a48d6)

[https://github.com/xxv/android-lifecycle](https://github.com/xxv/android-lifecycle)