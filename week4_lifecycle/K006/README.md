# Lifecycle Aware Component

Lifecycle은 안드로이드 개발에서 가장 중요하고 핵심이 되는 개념이라고 할 수 있다. 애플리케이션은 항상 lifecycle을 고려해야 하며, 그렇지 않으면 메모리 누수 또는 애플리케이션의 비정상 종료가 발생할 수 있다. 

## Lifecycle Aware Component

Lifecycle aware component 는 직역하면 생명주기 인식 컴포넌트를 의미하는데, 이는 말 그대로 lifecycle을 관찰하고 그에 따라서 동작하는 컴포넌트로 해석할 수 있다. 우리가 항상 당연하게 사용해왔던 lifecycle callback 또한 내부적으로 이 개념으로 동작하게 된다.

그렇다면 우리는 항상 당연하게 사용했던 개념을 가능하게 하는 원리는 무엇인지 한번 알아볼 필요가 있을 것이다. 모르고 쓰는 것과 알고 쓰는 것은 엄청난 차이니까 말이다. 지금까지는 당연히 lifecycle callback 메서드에 원하는 동작을 정의하고, 해당 lifecycle state가 되면 우리가 정의한 동작이 알아서 잘 작동했는데, 이것이 어떻게 가능한 것인지 적당히 한번 알아보려고 한다.

## Lifecycle

가장 기본이 되는 것은 당연히 Lifecycle 클래스이다. 공식문서에서는 Lifecycle 클래스를 다음과 같이 정의하고 있다. 

> Lifecycle은 액티비티나 프래그먼트와 같은 구성요소의 생명주기 상태 관련 정보를 포함하며 다른 객체가 이 상태를 관찰할 수 있게 하는 클래스입니다.

즉, 액티비티 나 프래그먼트의 생명주기 정보를 가지고 있는 클래스라고 생각하면 된다. Lifecycle 클래스 내부에는 생명주기에 따른 상태(State) 와 상태 전환시 발생할 수 있는 이벤트(Event)를 정의하고 있다.

<img src="https://s3.us-west-2.amazonaws.com/secure.notion-static.com/f021c813-76ca-4483-a217-c3178475f5cd/Untitled.svg?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAT73L2G45O3KS52Y5%2F20210926%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20210926T154949Z&X-Amz-Expires=86400&X-Amz-Signature=906e6e4ee7de951dc5f2234b4e5cff3b4a75ee3b9f7b73d94d41418473ae53ba&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22Untitled.svg%22" width="600">


<img src="https://s3.us-west-2.amazonaws.com/secure.notion-static.com/3f7edb00-abe3-450b-a470-c66a936f34ff/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAT73L2G45O3KS52Y5%2F20210926%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20210926T155025Z&X-Amz-Expires=86400&X-Amz-Signature=d64efb67a9615334aa07a926b7f5597c239660074d405541fea68f0d80d73e9f&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22Untitled.png%22" width="600">

<img src="https://s3.us-west-2.amazonaws.com/secure.notion-static.com/cbf21c66-a867-4047-8122-28131c6c0ee0/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAT73L2G45O3KS52Y5%2F20210926%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20210926T155041Z&X-Amz-Expires=86400&X-Amz-Signature=2dc44af2c431f19e81b0a7a12ea7ab42d728eaebee12f913e83f837e55e61a97&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22Untitled.png%22" width="600">


Lifecycle 클래스 내부에는 이외에도 상태를 변환하는 메서드 등이 존재한다.

그렇다면 결국 액티비티나 프래그먼트는 lifecycle에 따라 동작을 해야한다면, lifecycle 정보를 가지고 있어야할 것인데, 이는 어떻게 구현이 되어 있는지 살펴보도록 하자.

## LifecycleOwner

액티비티나 프래그먼트는 어떤식으로 lifecycle 정보 즉, Lifecycle 객체를 가지고 있는 것일까?

이는 LifecycleOwner를 통해 구현이 되어있다. 

<img src="https://s3.us-west-2.amazonaws.com/secure.notion-static.com/3be609c6-cb10-457b-aad0-870947c2a5f4/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAT73L2G45O3KS52Y5%2F20210926%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20210926T155056Z&X-Amz-Expires=86400&X-Amz-Signature=17e0d5944872b71b797ed262c860043d9fc1f849fe4eab64c0ae2471eb4f5601&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22Untitled.png%22" width="600">


LifecycleOwner는 getLifecycle() 라는 단일 메서드를 가지고 있는 인터페이스이다. 

문서에서도 알 수 있듯이 이 인터페이스를 구현한 클래스는 독자적인 생명주기를 가지게 되고, 생명주기의 변화에 따라서 동작할 수 있는 컴포넌트가 될 수 있다고 한다.

결국 액티비티와 프래그먼트 또한 LifecycleOwner 인터페이스를 구현했기 때문에 독자적인 생명주기를 가지고 동작할 수  있는 것이다.

<img src="https://s3.us-west-2.amazonaws.com/secure.notion-static.com/67f63b8e-f1d0-48c8-a1d0-ddff18832217/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAT73L2G45O3KS52Y5%2F20210926%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20210926T155111Z&X-Amz-Expires=86400&X-Amz-Signature=6ad155dfffd064c0c57fa3f9f76d7032cc24d200f4263f36fd82a70ff94d4489&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22Untitled.png%22" width="600">

<img src="https://s3.us-west-2.amazonaws.com/secure.notion-static.com/5632c7cc-1e37-4f29-a1c8-2109b0a66be6/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAT73L2G45O3KS52Y5%2F20210926%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20210926T155126Z&X-Amz-Expires=86400&X-Amz-Signature=4944e8703afd510a9dbe53b8c1758b29d47bfab12891eaf515a64ac02a293a0c&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22Untitled.png%22" width="600">


LifecycleOwner에는 getLifecycle() 이라는 단일 메서드가 존재했었는데 액티비티의 실제 구현체를 확인해보았다.

<img src="https://s3.us-west-2.amazonaws.com/secure.notion-static.com/ad46f846-9297-458f-ba29-3bf9df28e27e/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAT73L2G45O3KS52Y5%2F20210926%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20210926T155145Z&X-Amz-Expires=86400&X-Amz-Signature=ac6fd9b819f5e8a444a31da72ebe4fd3f91d3362a03bea196af0a10d60e81aa5&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22Untitled.png%22" width="600">


```java
private final LifecycleRegistry mLifecycleRegistry = new LifecycleRegistry(this);
```

여기서는 LifecycleRegistry 이라는 타입을 리턴하고 있었고, 이는 당연히 Lifecycle 클래스를 상속한 클래스였다. 여기만 더 따라가보면 뭔가를 알 수 있을 것 같다.

```java
public class LifecycleRegistry extends Lifecycle {

	public void markState(@NonNull State state) {...}
	public void setCurrentState(@NonNull State state) {...}
	public void handleLifecycleEvent(@NonNull Lifecycle.Event event) {...}
	private void moveToState(State next) {..}
	public void addObserver(@NonNull LifecycleObserver observer) {...}
	public void removeObserver(@NonNull LifecycleObserver observer) {...}
	...
}
```

LifecycleRegistry 은 Lifecycle 추상 클래스를 구현한 클래스로 생명주기에 따라 상태변경, 이벤트 동작, 옵저버 관리 등의 실질적인 동작을 하는 클래스이다. 

메서드 이름으로 유추해보았을 때 여기서 컴포넌트의 상태가 변경되고, 이벤트가 발생되는 것을 알 수 있다. 여기서 남아있는 옵저버가 우리가 추가적으로 알아봐야할 부분이다. LifecycleRegistry(Lifecycle) 에서 옵저버를 등록하고 취소하는 기능이 있다는 것을 알고 넘어가자.

## LifecycleObserver

안드로이드를 하면서 가장 먼저 접했던 디자인 패턴은 옵저버 패턴이었다. 여기서 등장하는 옵저버 또한 옵저버 패턴에서의 역할과 동일하다. 옵저버는 말 그대로 무언가를 관찰하게 되는데, LifecycleObserver는 이름 그대로 생명주기를 관찰하게 된다.

<img src="https://s3.us-west-2.amazonaws.com/secure.notion-static.com/dbd9b494-5f34-4c38-8ac9-9d5b34a35a24/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAT73L2G45O3KS52Y5%2F20210926%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20210926T155205Z&X-Amz-Expires=86400&X-Amz-Signature=8b680528a6c10dfc370af9bdac673f6c54dcb94d6becb01c5fd7707074de0de2&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22Untitled.png%22" width="600">


코드상에서도 알 수 있듯이 LifecycleObserver는 마커 인터페이스로, 메서드도, 구현해야할 추상 메서드도 존재하지 않는다. 하지만 문서에서도 나와 있듯이 OnLifecycleEvent 어노테이션과 함께 무슨 역할을 한다고 나와 있다. 

그렇다면 다시 Lifecycle 쪽에서는 LifecycleObserver를 어떻게 다루는지 한번 살펴보자.

<img src="https://s3.us-west-2.amazonaws.com/secure.notion-static.com/1035e386-1474-4d0c-82d6-c0cc0daefe38/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAT73L2G45O3KS52Y5%2F20210926%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20210926T155218Z&X-Amz-Expires=86400&X-Amz-Signature=ac6b5b802d1851a2ee4b79cfd3c72e53674c748c02f9638fc7db576fe77dff31&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22Untitled.png%22" width="600">

LifecycleRegistry의 addObserver 메서드의 일부이다. 여기서 LifecycleObserver 객체를 map에 등록해서 관리하고 있다. 그리고 removeObserver에서는 반대로 observer객체를 map에서 삭제한다. 

map에 등록하는 ObserverWithState 객체는 컴포넌트의 상태와 이벤트에 대한 옵저버를 동시에 가지는 객체이다.

<img src="https://s3.us-west-2.amazonaws.com/secure.notion-static.com/088545cf-65b6-45b0-b9c6-071847b11bba/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAT73L2G45O3KS52Y5%2F20210926%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20210926T155231Z&X-Amz-Expires=86400&X-Amz-Signature=0506665fda9a350b3f73f8a6df1ca006081450254382305bb5a68feda0e393cf&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22Untitled.png%22" width="600">

이렇게 Lifecycle 객체에서 옵저버를 등록 / 제거하고 사용하는 것이 옵저버 패턴을 그대로 사용한다고 볼 수 있다.

그렇다면 이렇게 옵저버를 등록해서 어디다가 써먹는 것일까?? 

이미 알고 있듯이, 특정 이벤트 동작을 관찰하다가 그에 해당하는 동작을 수행할 것이다. 그렇다면 이를 한번 확인해보자!

<img src="https://s3.us-west-2.amazonaws.com/secure.notion-static.com/9dc120b7-e2b3-41ea-b45d-dccbe2070adc/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAT73L2G45O3KS52Y5%2F20210926%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20210926T155249Z&X-Amz-Expires=86400&X-Amz-Signature=d8c72ecea2ca547fae18aa8c1cb77944621f888bd83433bf1cca3b4a19d946ec&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22Untitled.png%22" width="600">

 

문서에서도 나와있듯이 이벤트가 발생하면, 이벤트에 따라 적절하게 컴포넌트의 상태를 변경하고 알맞은 옵저버를 찾는다. 

<img src="https://s3.us-west-2.amazonaws.com/secure.notion-static.com/2ca749d5-3db7-451e-9aba-bb26d5190329/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAT73L2G45O3KS52Y5%2F20210926%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20210926T155304Z&X-Amz-Expires=86400&X-Amz-Signature=967a892bde41f526aa6892d58db8281688aa5f72e1a4fc4782a1495123183ad3&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22Untitled.png%22" width="600">

<img src="https://s3.us-west-2.amazonaws.com/secure.notion-static.com/2d914266-6336-4a27-bf60-bfcee8895f27/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAT73L2G45O3KS52Y5%2F20210926%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20210926T155323Z&X-Amz-Expires=86400&X-Amz-Signature=f1a54b02f511f86b217b1c8ee91122d81c0595be5be5b443d872ead850740222&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22Untitled.png%22" width="600">


옵저버를 등록 해놓은 map 을 돌면서 알맞은 옵저버를 찾고, 이벤트에 맞는 동작을 수행하는 것으로 보인다. 이벤트와 알맞은 동작을 연결하는 부분에 대해서는 더 깊게 들어가지는 못했지만 이 내용으로도 충분히 유추해 볼 수 있었다. 

옵저버가 이벤트에 따라서 적절한 콜백을 정의하는 부분은 공식 문서를 참고하였다. **@OnLifecycleEvent** 어노테이션을 부여한 메서드가 해당 이벤트에 대한 콜백이 되는 구조이다.  

```kotlin
class MyObserver : LifecycleObserver {

    private val TAG = "MyObserver"

    @OnLifecycleEvent(Lifecycle.Event.ON_CREATE)
    fun onCreateEventCallback() {
        Log.i(TAG, "onCreateEventCallback")
    }

    @OnLifecycleEvent(Lifecycle.Event.ON_START)
    fun onStartEventCallback() {
        Log.i(TAG, "onStartEventCallback")
    }
		...
}
```

```kotlin
class MainActivity : AppCompatActivity() {
    private val TAG = "MainActivity"

    override fun onCreate( savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        lifecycle.addObserver(MyObserver())
    }
}
```

액티비티의 lifecycle에 커스텀 observer를 등록해주면 된다. 이런식으로 커스텀 observer를 구현하게 되면 액티비티의 lifecycle을 따르지만, 콜백은 액티비티에 정의하지 않을 수 있다. 

이런 식의 구조를 사용하면 액티비티와 프래그먼트의 생명주기 콜백의 경량화가 가능하고 유지보수가 용이해진다는 장점도 있다. 

**REFERENCE**

[https://developer.android.com/topic/libraries/architecture/lifecycle](https://developer.android.com/topic/libraries/architecture/lifecycle)

# ViewModel

ViewModel 클래스는 액티비티나 프래그먼트 같은 UI 컨트롤러의 생명 주기를 고려하여 UI 관련 데이터를 저장하고 관리하도록 설계되었다. 가장 흔한 ViewModel 의 기능은 화면 회전과 같이 구성을 변경할 때도 데이터를 유지할 수 있다는 것이다.

## 왜 ViewModel 을 사용하는 것인가?

위에서 잠시 언급했지만 ViewModel을 사용하면 화면 회전 등의 구성이 변경되어 액티비티 등이 소멸 후 재 생성될 때 데이터를 유지할 수 있다. 이는 뒤에서도 살펴 보겠지만 ViewModel의 생명주기가 액티비티의 생명주기 보다 길기 때문에 가능한 것이다. 

상태 저장은 이전에도 살펴본 것처럼 onSaveInstanceState() 등의 메서드를 통해서도 가능하지만, 이는 직렬화가 가능한 소량의 데이터에만 적합한 방식이다. 비트맵 같은 대용량의 데이터는 ViewModel을 통해서 상태를 유지할 수 있다.

상태저장 이외에도 ViewModel을 사용해야할 마땅한 이유가 존재한다. ViewModel을 사용하지 않은 경우에는 결국 UI 표시, 사용자 상호작용, 네트워크 / 데이터베이스 처리, 권한 요청 등 다양한 일들을 UI 컨트롤러에서 처리해야하고 당연히 컨트롤러 클래스가 모든 로직을 처리하게 된다. 이는 하나의 클래스만 너무 방대해지고, 유지보수가 힘들어진다는 단점이 존재한다. 즉, 이런 경우에 ViewModel을 사용해서 관심사의 분리가 필요한 것이다. 이렇게 컨트롤러에서 UI 데이터 로직을 분리하는 것은 획기적이고 효율적인 방식인 것이다. 

## ViewModel 프로세스

<img src="https://s3.us-west-2.amazonaws.com/secure.notion-static.com/2a6742b2-1189-463d-8b8f-f452668dc050/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAT73L2G45O3KS52Y5%2F20210926%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20210926T155344Z&X-Amz-Expires=86400&X-Amz-Signature=a764e966a91c62c1db24de17dec17e2ed92379c9d88f6e19f1245f53beb39722&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22Untitled.png%22" width="600">

출처 : [https://charlezz.medium.com/viewmodel이란-무엇인가-viewmodel-초보를-위한-가이드-e1be5dc1ac18](https://charlezz.medium.com/viewmodel%EC%9D%B4%EB%9E%80-%EB%AC%B4%EC%97%87%EC%9D%B8%EA%B0%80-viewmodel-%EC%B4%88%EB%B3%B4%EB%A5%BC-%EC%9C%84%ED%95%9C-%EA%B0%80%EC%9D%B4%EB%93%9C-e1be5dc1ac18)

ViewModel은 ViewModelProvider를 통해서 참조가 가능하다. 액티비티나 프래그먼트에서 ViewModelProvider를 통해 ViewModel을 요청한다. 플로우는 두개로 나뉘게 되는데 기존의 ViewModel이 존재하는지, 아닌지에 따라서 나뉘게 된다. 

ViewModelStore는 ViewModel을 관리하고 있고, ViewModelStoreOwner가 ViewModelStore를 가지고 있다. 여기서 기존에 ViewModel이 존재한다면 ViewModelStore를 통해서 ViewModel을 가져오고, ViewModel이 없다면 ViewModelProvider의 Factory를 통해서 ViewModel 인스턴스를 생성하고 ViewModelStore에 저장한다. 

다음은 counting 앱의 데이터를 저장하기 위한 뷰모델을 생성한 것이다. 

단순하게 counting 데이터를 livedata로 저장하고 있는 뷰모델이다. 그렇다면 이 뷰모델 객체를 어떻게해야 액티비티나 프래그먼트 같은 UI 컨트롤러에서 생명주기와 함께 사용할 수 있을 지 알아보겠다. 단순히 뷰모델 객체를 생성한다고 해서 생명주기를 인식하고 알아서 동작 하지는 않을테니 말이다.

```kotlin
class CountingViewModel : ViewModel() {
    private val counting: MutableLiveData<Int> = MutableLiveData()
    init { counting.value = 0 }
    fun getCounting() = counting
}
```

### ViewModelProvider

액티비티나 프래그먼트에서 ViewModel을 참조하기 위해서는 ViewModelProvider를 사용한다. 

```kotlin
countingViewModel = ViewModelProvider(this).get(CountingViewModel::class.java)
```

<img src="https://s3.us-west-2.amazonaws.com/secure.notion-static.com/be4bf435-33fd-47e6-8a94-17916c8271c0/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAT73L2G45O3KS52Y5%2F20210926%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20210926T155404Z&X-Amz-Expires=86400&X-Amz-Signature=229f25a85071b70e6bbbe484e7cfe5c45cba4547904d957e517fa0ff59e9646b&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22Untitled.png%22" width="600">


첫번째 생성자를 살펴보면 인자의 타입이 ViewModelStoreOwner인 것을 알 수 있다. 이는 말 그대로 ViewModelStore를 가지고 있다는 의미의 단일 메서드 인터페이스이다. 

<img src="https://s3.us-west-2.amazonaws.com/secure.notion-static.com/976f5e53-e8d2-4906-b767-e2136f03bc8c/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAT73L2G45O3KS52Y5%2F20210926%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20210926T155416Z&X-Amz-Expires=86400&X-Amz-Signature=cb37f6c0e98b09996b4103e012967517128a0c19203dfaccea6108ee32fca2d0&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22Untitled.png%22" width="600">


ComponentActivity , Fragment 클래스가 모두 이를 구현하기 때문에 생성자의 인자로 this를 넘길 수 있었던 것이다. 그리고 첫번째 생성자에서 추가적으로 Factory를 설정하고 있는 것을 확인할 수 있다. 대충 DefaultFactory가 없으면 NewInstanceFactory를 생성해서 넘기는 것을 확인할 수 있다. 

```kotlin
countingViewModel = ViewModelProvider(this).get(CountingViewModel::class.java)
```

이제 ViewModelProvider 객체는 생성했으니 get() 메서드에 대해서 살펴보도록하자. ViewModelProvider 의 get 메소드에 클래스 타입을 인자로 넘겨 ViewModel 인스턴스를 가져오는 것이다. 

<img src="https://s3.us-west-2.amazonaws.com/secure.notion-static.com/7a50d42b-4766-430f-87c6-9c0e2a2ce9c9/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAT73L2G45O3KS52Y5%2F20210926%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20210926T155430Z&X-Amz-Expires=86400&X-Amz-Signature=79b6aa9b51214f6a4f85fb95bf57b765138abcc24caefd4bfba0e0700d8d0a8c&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22Untitled.png%22" width="600">

코드를 보면 mViewModelStore에서 ViewModel을 찾고 ViewModel 이 존재하면 이를 리턴하고 , 존재하지 않으면 Factory를 통해 ViewModel을 생성하는 것을 확인할 수 있다.

### ViewModelStore

위에서 잠시 살펴본 것처럼 ViewModelStoreOwner는 ViewModelStore를 가지고 있고, ViewModelStore에서 ViewModel을 관리한다.

<img src="https://s3.us-west-2.amazonaws.com/secure.notion-static.com/87bd0f01-914d-4e88-98da-b23e62edc34e/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAT73L2G45O3KS52Y5%2F20210926%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20210926T155443Z&X-Amz-Expires=86400&X-Amz-Signature=7972cd8fb963cb2ec1cb158be564c97553b5cc02ab4bc40b46683ff685306be6&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22Untitled.png%22" width="600">

내부 코드를 살펴보면 ViewModelStore가 ViewModel을  map으로 관리하고 있는 것을 확인할 수 있다. ViewModelProvider에서 key값을 통해서 ViewModel을 검색하는 것에서 유추할 수 있었다.

## ViewModel 의 생명주기

<img src="https://s3.us-west-2.amazonaws.com/secure.notion-static.com/d79ef3d0-984a-4fcf-8bc5-a85a13d69f18/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAT73L2G45O3KS52Y5%2F20210926%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20210926T155456Z&X-Amz-Expires=86400&X-Amz-Signature=64b3b3cc7e370571e842c061fea526d1168b8c16976a4632aaf25b848c6afd20&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22Untitled.png%22" width="600">


앞에서도 알아봤던 것 처럼 ViewModel의 생명주기는 관련 UI 컴포넌트의 생명주기 보다 길기 때문에 상태정보를 저장할 수 있었다. 위의 그림은 액티비티의 경우를 나타낸 것인데, 액티비티가 destroy 되어도 ViewModel 생명주기는 계속해서 유지되는 것을 알 수 있다.

그리고 최종적으로 finish() 가 호출되어야 ViewModel도 같이 종료된다.

<img src="https://s3.us-west-2.amazonaws.com/secure.notion-static.com/12465fce-645d-49cb-b92f-ec90dd8424eb/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAT73L2G45O3KS52Y5%2F20210926%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20210926T155507Z&X-Amz-Expires=86400&X-Amz-Signature=672ac8525d3395e5e304a8f3960119d52f0cc48bccd7db3ae3dd703c3ecb06dc&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22Untitled.png%22" width="600">


이 예를 보면 확인할 수 있다. 화면 회전을 했을 경우 로그를 확인해보면 액티비티는 소멸되고 재생성 되었지만, ViewModel은 유지되었다. 하지만 finish()를 호출하여 액티비티를 종료했을 때는 마지막에 ViewModel 또한 소멸되는 것을 확인할 수 있었다. 

ViewModel 인스턴스의 생명주기는 ViewModelProvider의 인자로 주어진 ViewModelStoreOwner의 생명주기를 따른다. (이 부분은 코드상에서 확인하지는 못하고 자료를 참고하였다.)

그렇기 때문에 위의 모든 예시는 인자로 액티비티를 넘겨주었지만, 프래그먼트를 넘겨준다면 프래그먼트가 분리될 때까지 ViewModel 이 동작하게 된다.

ViewModelStoreOwner가 완전히 소멸되는 경우에는 ViewModelStore의 clear() 메서드가 자동으로 수행되기 때문에 ViewModel을 관리하고 있던 map을 비워 ViewModel 이 더이상 참조되지 않게하여 가비지 컬렉션의 대상이 되도록 한다.

ViewModel을 응용하는 부분을 이를 제외하고도 다양하지만 이번에는 내부 구현 중심으로 정리를 했다. 활용 부분을 모두 다루기에는 양이 방대해서 이는 다음에 다루도록.

**REFERENCE**

[https://developer.android.com/topic/libraries/architecture/viewmodel?hl=ko](https://developer.android.com/topic/libraries/architecture/viewmodel?hl=ko)

[https://charlezz.medium.com/viewmodel이란-무엇인가-viewmodel-초보를-위한-가이드-e1be5dc1ac18](https://charlezz.medium.com/viewmodel%EC%9D%B4%EB%9E%80-%EB%AC%B4%EC%97%87%EC%9D%B8%EA%B0%80-viewmodel-%EC%B4%88%EB%B3%B4%EB%A5%BC-%EC%9C%84%ED%95%9C-%EA%B0%80%EC%9D%B4%EB%93%9C-e1be5dc1ac18)

[https://choheeis.github.io/newblog//articles/2021-02/viewModel](https://choheeis.github.io/newblog//articles/2021-02/viewModel)

[https://javachoi.tistory.com/138](https://javachoi.tistory.com/138)

# LiveData

LiveData는 안드로이드에서 옵저버 패턴을 사용하는 데이터 홀더 클래스이다. 즉, 관찰 가능한 데이터 홀더 클래스 라고 볼 수 있다. 

LiveData는 일반적인 observable 클래스들과는 다르게 생명주기를 인식한다는 특징이 있다. 다시말해 액티비티, 프래그먼트, 서비스 등의 구성요소의 생명주기를 고려한다.

LiveData는 데이터를 관찰하면서 active한 생명주기 상태를 갖는 observer만 업데이트 한다.

## LiveData 프로세스

사용법은 정말 간단하다. LiveData 는 보통 ViewModel과 함께 사용되기 때문에, ViewModel 내부에 LiveData를 생성하였다. ( 샘플 스니펫은 위에서 사용한 예제와 동일함.)

```kotlin
class CountingViewModel : ViewModel() {
    private val counting: MutableLiveData<Int> = MutableLiveData()
    init { counting.value = 0 }
    fun getCounting() = counting
}
```

ViewModel에 LiveData를 저장하고, 데이터 변경에 대한 옵저버를 UI 컨트롤러에서 등록해주면 된다.

```kotlin
// MainActivity

override fun onCreate( savedInstanceState: Bundle?) {
		countingViewModel.getCounting().observe(this) { // Observer 등록
				...
		}
}
```

이렇게만 해주면 알아서 동작은 하게 된다. 그렇다면 LiveData는 어떻게 생명주기를 인식하게 되는 것일까?

<img src="https://s3.us-west-2.amazonaws.com/secure.notion-static.com/4c56a0b0-55bd-4e17-bac5-57de924d2f27/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAT73L2G45O3KS52Y5%2F20210926%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20210926T155527Z&X-Amz-Expires=86400&X-Amz-Signature=762d7ba2d9ade31675dd8cd7fd4bc9a0b9074705635c165313d671ebaee157a6&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22Untitled.png%22" width="600">

observe 메서드를 보면 인자로 `LifecycleOwner`와 `Observer`를 받는 것을 확인할 수 있다. 여기에서 LifecycleOwner는 컴포넌트의 생명주기를 파악하기 위해서, Observer는 LiveData가 홀딩하고 있는 데이터의 변경을 감지하기 위한 것으로 유추할 수 있다. 

Observer는 onChanged()를 가지고 있는 단일 메서드 인터페이스이다. 이는 LiveData 의 변경사항에 대한 콜백 메서드로 오버라이딩 하면 된다.

<img src="https://s3.us-west-2.amazonaws.com/secure.notion-static.com/2d787352-f75c-45d6-9b50-d33057eeae1a/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAT73L2G45O3KS52Y5%2F20210926%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20210926T155539Z&X-Amz-Expires=86400&X-Amz-Signature=39481ec40cfaa39c7e1ec94b79f8ce38d381db45f88b45cfbacf686896d7ffb8&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22Untitled.png%22" width="600">


## LifecycleBoundObserver & ObserverWrapper

observe 메서드 내부에서는 인자로 받은 객체들을 가지고 `LifecycleBoundObserver` 객체를 생성한다.

LifecycleBoundObserver는 `ObserverWrapper` 를 상속하고 `LifecycleEventObserver`를 구현한 클래스이다. 여기서 알 수 있듯이 이 객체를 통해서 결국 생명주기와 라이브데이터를 둘 다 observing 하는 것을 알 수 있다. 

실제로 코드상에서도 라이브데이터에 대한 observer로 사용하고 관리하기 위해서 이 객체를 map으로 관리하고,

```java
private SafeIterableMap<Observer<? super T>, ObserverWrapper> mObservers =
            new SafeIterableMap<>();
...

public void observe(@NonNull LifecycleOwner owner, @NonNull Observer<? super T> observer) {
		LifecycleBoundObserver wrapper = new LifecycleBoundObserver(owner, observer);
		ObserverWrapper existing = mObservers.putIfAbsent(observer, wrapper);
		...
}
```

그리고 observe 메서드의 마지막에는 lifecycle의 observer로도 등록하고 있다.

```java
owner.getLifecycle().addObserver(wrapper);
```

이런 방식으로 LiveData 의 옵저버는 데이터의 변경과 동시에 생명주기까지 관찰할 수 있는 것이다.

## LiveData Observing

조금만 더 자세히 파헤쳐 보도록 하자. 라이브데이터는 값이 변경되면 이를 감지하여 observer에 의해서 콜백이 호출된다. 

그렇다면 변경의 시작점부터 차근차근 살펴보면 원리를 알 수 있을지도 모른다. 그러면 우선 `setValue()` 로 가보자.

<img src="https://s3.us-west-2.amazonaws.com/secure.notion-static.com/22f6ed75-439c-41c7-8166-ea84487fae24/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAT73L2G45O3KS52Y5%2F20210926%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20210926T155556Z&X-Amz-Expires=86400&X-Amz-Signature=a876d5168566ee3b87a442181532be5fd8c4b20dde83648e75f5a03408d8b9a0&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22Untitled.png%22" width="600">


setValue 에서 눈에 띄는 것이 `mVersion`과 `dispatchingValue()` 가 있다. 우선 mVersion이란 LiveData 내부의 private int 필드고, 이의 사용처를 살펴보았더니 다음과  같은 부분을 발견할 수 있었다.

<img src="https://s3.us-west-2.amazonaws.com/secure.notion-static.com/de9da469-d021-470c-b29a-a0f54a24c1b2/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAT73L2G45O3KS52Y5%2F20210926%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20210926T155650Z&X-Amz-Expires=86400&X-Amz-Signature=5602d348d14559bcb2cd110fd8dfb7838800f3c2c9af2e9a42cd3517078a33c0&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22Untitled.png%22" width="600">

`considerNotify`  메서드를 살펴보면, 현재 version이 이전의 version 보다 큰 경우에는 데이터가 변경 되었다고 간주할 수 있기 때문에(setValue 메서드에서 증가 해주었기 때문),  observer 의 onChanged() 콜백을 호출하게 된다.

그리고 `dispatchingValue()` 메서드 내부 조건에 따라 considerNotify() 메서드를 호출하는 방식으로setValue() 메서드를 구성하게 된다.  

**REFERENCE**  
https://developer.android.com/topic/libraries/architecture/livedata?hl=ko