# Fragment Lifecycle

---

**Fragment**에도 **Lifecycle** 이 존재한다. 그러기 위해서는 당연히 **Fragment** 또한 **LifecycleOwner**를 구현해야하고 실제로 이를 구현했다.

<img src="https://user-images.githubusercontent.com/71161576/132982981-4cafbcb3-a1a0-4e08-a96b-02c5769039a2.png"/>

내부적으로 **Lifecycle** 객체가 **lifecycle** **state**를 관리하게 된다. 

**Lifecycle** 객체는 **lifecycle** **state**을 **enum**으로 관리하고, 그 값은 다음과 같다.

- `INITIALIZED`
- `CREATED]`
- `STARTED`
- `RESUMED`
- `DESTROYED`

**Lifecycle**을 가지는 객체가 lifecycle-aware-component와 상호작용을 하는 과정은 다음과 같다. 

Lifecycle 객체에 의해서 lifecycle state 가 변경이 되면, 적절한 단계의 event가 발생하게 된다. ( state 와 event 는 별개의 정보를 나타낸다 ) 그리고 event가 발생하면 등록된 **lifecycle observer**가 알림을 받고 event에 대한 callback 을 실행하는 구조로 동작하게 된다.

<img src="https://user-images.githubusercontent.com/71161576/132983017-e8bbcdfb-c7d8-43f5-99f0-b76ba6663b67.png"/>

Lifecycle 객체의 동작과 LifecycleOwner - LifecycleObserver의 동작에 대해서는 따로 정리를 해야할 듯 하다.

이번에는 위의 내용을 바탕으로 event가 발생했을 때, fragment에서 실행되는 callback과 그 특징에 대해서 간단하게 정리하려고 한다.

  


<img srg="https://user-images.githubusercontent.com/71161576/132983034-b97455cf-79a6-42a1-96be-706398fc3080.png"/>
### onCreate()

이때는 **fragment**의 **Instance** 가 구성된다. 이는 **FragmentManager** 에 add 됐을 때 호출된다. 주의할 점은 **onCreate()** 이전에 **onAttach()** 가 먼저 호출된다는 것이다.

이 시점에는 아직 **Fragment** **View** 가 생성되지 않았기 때문에 **Fragment** 의 **View** 와 관련된 작업을 수행하는 것은 적절하지 않다. 그렇기 때문에 activity 와 다르게 여기서 레이아웃을 **inflate** 하면 안된다.

**Activity**와 마찬가지로 이 때도 `savedInstanceState` 매개 변수를 받는데 이는 **activity** 의 콜백과 기능이 동일하다.

### onCreateView()

**onCreate()** 이후에는 **onCreateView()** 와 **onViewCreated()** 콜백이 이어서 호출된다.  onCreateView() 에서는 보통 **fragment** 뷰의 **layout**을 **inflate** 한다. 그리고 레이아웃이 **inflate** 된 뷰는 **hosting** **activity**에 다시 전달이 된다. 

### onViewCreated()

**onCreateView()** 에 이어서 바로 호출이 된다. **onCreateView()** 에서 반환한 **View**를 인자로 받게 된다. 이 콜백의 실행은 **View** 가 성공적으로 생성 되었다는 것을 보장한다. 그렇기 때문에 **onViewCreated()** 콜백에서 **View**의 초깃값 설정, 리스너 설정 등의 **View** 관련 기능을 초기화 할 수 있다.

### onViewStateRestored()

이는 다소 생소한데, onViewStateRestored() 콜백은 저장해둔 모든 state 값이 Fragment 의 View 계층구조에 복원 됐을 때 호출된다. ( 이는 정확이 무슨 내용인지 잘 이해하지 못했다. )

### onStart()

**onStart()** 콜백은 **fragment** 가 사용자에게 보여질 수 있을 때 호출된다. 이는 주로 **fragment** 가 **attach** 되어있는 **activity** 의 **onStart()** 시점과 유사하다. 

이 시점부터는 **fragment** 의 **childFragmentManager** 통해 **fragment** **transaction** 을 안전하게 수행할 수 있음을 보장할 수 있다.

<img src="https://user-images.githubusercontent.com/71161576/132983067-81105aa8-094b-4595-b07d-12e2ea06ad44.png"/>

### **onResume()**

**Fragment** 가 보이는 상태에서, 사용자와 상호작용할 수 있을 때 **onResume()** 콜백이 호출된다. 이것 또한 마찬가지로 activity 의 **onResume()** 가 호출되는 시기와 유사하다.

**Resumed** 상태가 됐다는 것은 사용자가 **fragment** 와 상호작용 하기에 적절한 상태가 됐다고 했는데, 이는 반대로 **onResume()** 이 **호출되지 않은 시점에서는 입력을 시도하거나 포커스를 설정하는 등의 작업을 임의로 하면 안된다는 것**을 의미한다.

### onPause()

이 또한 **activity** 와 비슷하게 **fragment** 가 사용자와 상호작용 할 수 없는 상태가 되었지만, 여전히 **visible** 한 상태일 때 호출되는 콜백이다. 

### onStop()

**Fragment** 가 더이상 화면에 보여지지 않게 되면 **fragment** 와 **view** 의 **lifecycle** 은 **CREATED** 상태가 되고, **onStop()** 콜백이 호출된다. 이 상태는 **host** **activity** 나 **fragment** 가 중단됐을 때 뿐만 아니라, **host** **activity** 나 **fragment** 의 상태가 저장될 때도 호출된다. 이처럼 **fragment**는 **host activity** 의 **lifecycle**에 영향을 받게 된다. 

### onDestroyView()

모든 e**xit animation** 과 **transition** 이 완료되고, **onDestroy()** 가 호출되기 전에 **Fragment** **View** 의 **Lifecycle** 은 **DESTROYED** 가 된다.

### onDestroy()

**Fragment** 가 제거되거나 **fragmentManager** 가 **destroy** 됐을 경우, **fragment**의 **lifecycle** 은 **DESTROYED** 상태가 되고, **onDestroy()** 콜백 함수가 호출된다. 해당 지점은 **fragment** **lifecycle** 의 마지막에 해당된다.

**Fragment**는 독자적인 **lifecycle**을 가지고 있지만, 이는 **host activity**에 종속적이고, 영향을 받는다. 그렇기 때문에 상태와 콜백 또한 **activity** 와 유사하게 가져가는 것 같다. 

하지만 **fragment** 콜백의 경우, **activity**의 콜백과 비교했을 때 그 개수가 더 많은 것을 알 수 있다. 그리고 **fragment** **lifecycle** 플로우 차트를 확인하면 우측에 **view** **lifecycle** 또한 같이 표현이 되어 있는 것도 확인할 수있다. 

이처럼 **Fragment** 의 콜백은  **fragment** 자체의 상태에 따라서 호출이 되기도 하지만, 바인딩 되어 있는 **view** 의 **life** **cycle**에 따라서도 관련 콜백이 호출이 되는 것 같다. ( 콜백 이름 중 **View** 라는 infix가 포함된 것들이 그러한 역할을 하는 것 같다.) 

**왜** **fragment** 의 콜백에서만 추가적으로 **view**의 **life** **cycle**에 대한 관리까지 할 수 있는 지는 더 학습을 해야 하지만, 일단 결과적으로는 더 상세하게 life cycle에 대한 동작을 처리할 수 있는 것 같다.

# Fragment 의 상태 유지

---

다양한 안드로이드시스템 작업은 **fragment**에 영향을 미칠 수 있다. 이에 따라 **fragment**의 모든 데이터가 유지 되는 지 확인해야한다.

공식 문서에서는 **fragment**에 저장되는 데이터를 **fragment**의 상태라고 설명하고 있다. 공식문서에서는 **fragment**가 상태를 잃게 만드는 작업과, 다양한 상태가 이러한 작업을 통해 유지되는지 그래프를 통해서 간략하게 설명하고 있다.

먼저 공식문서에서 표현하는 fragment의 상태는 네 종류가 있다.

`변수` : 말 그대로 fragment 객체의 로컬 변수를 의미한다.

`뷰 상태(View State)` : fragment의 하나 이상의 뷰에서 소유한 모든 데이터를 의미한다.

`SavedState` : onSaveInstanceState()에 저장되어야 하는 fragment 인스턴스의 고유한 데이터

`NonConfig` : 서버 또는 로컬 저장소와 같은 외부 소스에서 가져온 데이터 또는 커밋된 후 서버에 전송되는 사용자 생성 데이터


<img src="https://user-images.githubusercontent.com/71161576/132983098-1daf9be1-fb25-48ef-8edc-4fa13bfeed8d.png"/>

이 표는 fragment의 상태와 상태에 영향을 끼치는 작업들을 표현한 것이다.

### 변수

그래프에서 알 수 있듯이 변수는 fragment가 백 스택에 추가될 때에는 그 데이터를 유지할 수 있다. 어떻게 보면 당연한 결과다. 하지만 구성이 변경되거나 프로세스가 종료되면 fragment 인스턴스 내부 변수에 저장된 데이터는 그 값을 유지하지 못한다.

### 뷰 상태(View State)

공식 문서에 따르면 뷰도 자체적으로 상태를 저장할 수 있다고한다. 안드로이드 View에는 기본적으로  `onSaveInstanceState()` 및 `onRestoreInstanceState()`의 구현이 있으므로 fragment 내에서 뷰의 상태를 관리하지 않아도 된다.


<img src="https://user-images.githubusercontent.com/71161576/132983108-1fc3abe5-853d-4e82-bbab-a1e5eb24f41e.png"/>
<img src="https://user-images.githubusercontent.com/71161576/132983123-f132a8e5-9871-47dd-acac-b9965e8c2f18.png"/>

실제로 View 클래스에 두 메서드가 구현이 되어 있다. 모든 뷰는 이 메서드를 오버라이딩해서 사용하면 된다.

그리고 뷰 상태를 유지하려면 뷰의 ID가 반드시 필요하다.

### SavedState

Fragment.onSaveInstanceState(Bundle)를 사용하여 소량의 직렬화된 데이터를 쉽게 유지할 수 있다.

그래프에서도 알 수 있듯이 SavedState는 구성 변경과 프로세스 중단 및 재생성에도 상태가 유지된다. 이는 액티비티와 동일하다. 이렇게 번들 객체에 저장된 데이터는 fragment의 `onCreate(Bundle)`, `onCreateView(LayoutInflater, ViewGroup, Bundle)`, `onViewCreated(View, Bundle)` 메소드에서 사용할 수 있다.

그래프를 확인해보면 SavedState 는 백스택에 저장될 때에는 상태가 유지되지 않는다.

아래의 스니펫은 번들 객체에 저장된 데이터를 꺼내오는 작업을 한다. 

```kotlin
override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    isEditing = savedInstanceState?.getBoolean(IS_EDITING_KEY, false)
    randomGoodDeed = savedInstanceState?.getString(RANDOM_GOOD_DEED_KEY)
            ?: viewModel.generateRandomGoodDeed()
}
```

fragment 의 로컨 변수는 백스택에 저장될 때에도 그 값을 유지하기 때문에 번들 객체의 값을 읽어와서 변수에 할당하는 방식으로 상태를 지속해서 유지할 수 있을 것으로 보인다.

### NonConfig

**NoConfig** 데이터는 **fragment**의 외부, 예를 들어 **ViewModel**에 배치 되는 데이터를 말하고 데이터는 **ViewModel** 로직에 의해서 유지가 된다.

**ViewModel** 클래스는 기본적으로 화면 회전과 같은 구성 변경에도 데이터를 유지할 수 있으며 **fragment**가 백 스택에 배치되면 메모리에서 유지된다. 그리고 **SavedState** 모듈을 **ViewModel**에 추가하면 **ViewModel**이 프로세스 중단 및 재생성에서도  간단한 상태를 유지할 수 있다. ( 그래프에서 * 표시가 되어 있는 부분)

이러한 이유에서 **activity** 또는 **fragment**에서 상태를 유지하기 위해서 **ViewModel**을 많이 사용하는 것인가보다.

아직 **ViewModel**에 대해서는 학습하지 않아서 자세한 내용은 이해할 수 없지만, 이런 역할을 한다고는 알아두자.

# JetPack Navigation?

[공식 문서에서도 내비게이션 컴포넌트의 사용에 대해서 권장하고 있다](https://developer.android.com/guide/fragments/fragmentmanager?hl=ko). 해당 프레임워크는 fragment, 백 스택, fragment manager 사용에 관한 권장사항을 잘 따르도록 설계되어있다고 한다. 

먼저 Navigation Component가 도입되기 이전을 살펴보자.  Navigation Component 가 도입되기 이전에는 개발자가 직접 `Fragment Manager`를 통해서 fragment를 관리했다. 사용자의 응답에 따라서 fragment를 추가 삭제 등의 연산을 수행해야했고, 이 일련의 변경 사항은 `FragmentTransaction` 이라는 단위로 관리가 된다.

데이터베이스에서 사용되는 트랜잭션과 개념이 일맥상통한다.

공식 문서에서는 다음과 같이 설명하고있다. 

> Jetpack Navigation 라이브러리를 사용하는 경우 개발자를 대신해 이 라이브러리가 FragmentManager를 사용하기 때문에 개발 시 FragmentManager와 직접 상호작용하지 않을 수 있다.

내가 fragment를 처음 학습하고 도입할 때, FragmentManager의 정체가 코드 상에서 드러나지 않은 이유가 이 라이브러리가 만들어진 이유 중 하나 였던 것이다.

하지만 fragment를 사용하는 앱은 반드시 일정 수준에서 FragmentManager를 사용하고 있기 때문에, 이것이 무엇인지 그리고 어떻게 사용이 되는 지 이해하는 것은 매우 중요하다.

FragmentManager 의 이해 → 왜 Jetpack Navigation Component 가 나왔나  의 순서로 이해하는 것이 바람직할 것이다. 이번에는 FragmentManager의 FragmentTransaction 위주로 알아보려고 한다.

### FragmentTransaction

이는 데이터베이스의 트랜잭션의 개념과 일맥상통한다. FragmentManager를 통해서 Fragment를 `추가`, `삭제` 및 `교체` 하는 작업은 트랜잭션 단위로 묶어서 진행이 된다. 그리고 이는 반드시 `commit` 되어야한다. (데이터베이스와 동일하게 변경사항을 반영하기 위함이다.)

```kotlin
val fragmentManager = ...
val fragmentTransaction = fragmentManager.beginTransaction()
```

FragmentTransaction 인스턴스는 위와 같이 참조가 가능하다.

```kotlin
// The fragment-ktx module provides a commit block that automatically
// calls beginTransaction and commit for you.
fragmentManager.commit { // this: FragmentTransaction 
    // 일련의 작업을 수행
}
```

이와 같이 일련의 작업을 트랜잭션 단위로 수행하고, 커밋을 할 수 있다.

이런 방식으로 FragmentManager는 FragmentTransaction으로 Fragment 추가, 삭제, 갱신 작업을 수행한다.

트랜잭션을 사용해 fragment를 관리하는 부분은 별도로 자세히 정리를 해야겠다. ( 까먹지마..😭 )

지금은 백스택의 관점에서 Navigation Component와 연관지어서 설명할 수 있는 부분만 정리하고 넘어가려고 한다.

### Fragment in Back Stack

Activity에 대해서 공부를 했다면 back stack의 개념에 대해서 알 수 있을 것이다. 안드로이드에서 activity의 전환이 발생하면 기본적으로 이전의 activity는 back stack에 저장이 된다.

<img src="https://user-images.githubusercontent.com/71161576/132983144-33d93fd8-3c64-4c64-8c8b-af878d1b7d89.png"/>
그렇기 때문에 디바이스의 뒤로가기 버튼을 눌렀을 때 back stack에 저장된 구조에 따라서 activity가 pop 되어서 이전 activity가 보이게 되는 것이다.

그렇다면 Fragment에도 이것이 동일하게 적용이 될까? ( Fragment 를 여러개를 만들거나, 순서대로 Fragment를 add & remove & add 했을 때 뒤로가기를 누르면 이전의 Fragment 순서 정보를 기억할까? 라는 질문이다.)

결론부터 말하자면 **No** 이다. 기본적으로 Fragment는 백스택을 통해서 관리가 되지 않는다. 그러면 Fragment는 뒤로가기를 누를때 다 꺼질 수 밖에 없는거야..?!

그것도 아니다. 개발자가 명시적으로 백스택에 `FragmentTransaction`을 등록할 수 있다. 

```kotlin
supportFragmentManager.commit { // this: FragmentTransaction
   replace<ExampleFragment>(R.id.fragment_container)
   setReorderingAllowed(true)
   addToBackStack("name") // name can be null
}
```

이렇게 트랜잭션에서 명시적으로 `addToBackStack()` 을 호출해서 백스택에 등록을 해줘야한다.

여기서 백스택에는 FragmentTransaction을 저장한다는 것을 명심하자! 단순히 Fragment를 순서대로 등록하는게 아니라 일련의 연산 과정의 결과를 백스택에 저장을 한다.


<img src="https://user-images.githubusercontent.com/71161576/132983159-0163d698-7a00-4bc8-90f9-73b9b82ed9a4.png" />

이렇게 백스택을 통해서 Fragment에 대한 정보도 관리를 할 수 있게 된다.

그렇다면 Fragment의 전환이 발생할 때 원하는 flow를 만들기 위해서는 그때 마다 트랜잭션을 백스택에 등록을 해줘야한다. 이것이 좀 많이 번거롭다고 생각할 수 있을 것이다. 

### Navigation Component

Google이 Google I/O 2018에서 Jetpack을 발표하였다. Jetpack은 좋은 안드로이드 앱을 더 쉽게 만들 수 있도록 구글에서 지원하는 라이브러리 패키지를 뜻한다. Jetpack에는 많은 라이브러리가 포함되어 있는데 Navigation Component 또한 여기에 포함 된 라이브러리 중 하나이다.

Navigation Component에 대한 설명은 [공식문서에 아주 자세하게 나와있었다](https://developer.android.com/guide/navigation/navigation-getting-started). 그 내용도 무척 방대하다.

간단하게 설명하자면 Navigation Component는 Fragment간의 연결과 이동 경로를 시작적으로 나타내는 Graph 라는 개념과 그래프에서 각 목적지(Fragment) 의 이동 동작을 설명하는 Action으로 구성이 된다.

<img src="https://user-images.githubusercontent.com/71161576/132983184-0a10c63d-b9e3-4b69-a2a3-298ccb26a746.png"/>
Navigation Component를 사용하면 개발자가 직접 트랜잭션을 백스택에 넣어줘야할 필요도 없고, Fragment 간의 전환을 모두 Navigation Component에 위임할 수 있다. 

이처럼 Fragment 간의 이동을 시각적으로 확인할 수도 있고, 백스택 관리를 라이브러리가 알아서 해주기 때문에 아주 간단하게 Fragment 전환이 가능한 것이다. 

이런 기본 동작을 제외하고도 navigation에 대한 다양한 기능을 제공하고, 공식문서에서도 Navigation Component를 권장하고 있다.

하지만 혹자는 이런 Navigation Component를 권장하지 않는다. 왜 그런 것일까?

(아직까지 나의 수준에서 왜 이러한 의견을 제시했는지 이해는 잘 되지 않지만,, 나도 실제로 사용해보면 비슷한 경우를 느낄 수 있지도 않을까?)

관련 내용을 많이 찾지는 못했지만 [Kaboomba님의 블로그](https://bignerdranch.tistory.com/40)를 참고하여 Navigation Component의 아쉬운점(?)에 대해서 알아봤다. 포스팅의 내용은 주관적일 수 있고, 나의 의견이 반영된 것이 아니라 단순히 참고를 하면 될 것 같다.

- **뷰와 강력한 의존 관계**
    - 대부분 Navigation Component를 뷰에서 사용을 하게 된다. 하지만 이는 뷰의 책임이 아니라는 것이 이 주제의 요점이다. 그렇다고 Presenter나 ViewModel에 이를 포함하는 것도 적합하지는 않아 보인다. Presenter나 ViewModel은 안드로이드 시스템에 대해서 알 필요가 없기 때문이다. 그렇기 때문에 이렇게 Navigation Component가 Activity 나 Fragment에 강한 의존을 가지는 구조에 대해서 의문을 던지는 것 같다.
- **So Complex!**
    - 라이브러리가 너무 방대하고 복잡하다고 한다. 구현 방식을 제대로 이해해서 사용하기엔 너무 복잡한 라이브러리라고 한다. 아주 간단한 내비게이션을 위해서 넣기에는 너무 큰 라이브러리이고 복잡한 앱의 경우에는 정확한 컨트롤이 쉽지 않은 라이브러리 라는 의견이 있었다. 그래서 현재는 Navigation 보다는 백스택 관리 라이브러리라는 인식도 있는 것 같다.
- **제한적인 구조?**
    - 공식 문서에서도 가장 처음 나오는 내용이다. Navigation Component는 Single Activity 의 구조를 기본으로 한다. Single Activity → Host Fragment → Fragments 의 방식으로 동작을 하게 된다. 그렇기 때문에 실제 앱의 요구사항을 반영하지 못하는 부분도 존재한다. 그렇기 때문에 이 부분을 처리해주기 위해서는 별도의 구현이 필요하다는 내용이 있었다.

Navigation Component가 기존의 Navigation 방식을 개선하기 위해서, 더 좋은 경험을 제공하기 위해서 나온 것은 분명해보인다. 하지만 모든 상황에서 유연하게 쓰일 수 있는 지에 대해서는 아직 의문이 있는 것 같다. 아직 경험이 많이 부족해서 나의 개인적인 견해는 나타내지 못했지만, 자료들을 찾아보면서 대략적인 흐름을 이해할 수 있었다. 그래서 지금 내 수준에서 내리는 결론은..!! Navigation Component를 쓰더라도 이걸 왜 쓰게 됐는지.. 쓰지 않은 경우에는 지금 이 동작이 어떻게 구현이 되어야하는지 생각하면서 쓰자! 이다.

# Fragment Constructor

---

Fragment 의 생성자는 기본 생성자만 사용할 수 있고, 생성자 오버로딩이 금지된다고 한다. 그 이유는 간단하다.

앞서 살펴본 바와 같이 **Fragment**는 생명주기에 따라 소멸과 생성을 반복한다. 

결론적으로 말하자면 소멸과 생성을 반복할 때 안드로이드 시스템이 기본 생성자를 사용해서 fragment를 생성하기 때문이다.

```java
// 1. Class Loader를 이용하여 Fragment class를 가져온다. 
Class<? extends Fragment> clazz = 
FragmentFactory.loadFragmentClass( context.getClassLoader(), fname); 

// 2. class의 기본 생성자를 사용하여 fragment 인스턴스 생성 
Fragment f = clazz.getConstructor().newInstance(); 

// 3. Bundle 객체인 args를 이용하여 fragment에 전달할 값 세팅 
if (args != null) { 
	args.setClassLoader(f.getClass().getClassLoader()); 
	f.setArguments(args); 
} 

// 4. 생성된 fragment인 f를 return 
return f;

// 출처: https://kotlinworld.com/73 [Kotlin World]
```

위의 내용을 보면 알 수 있듯이, fragment 객체를 생성할 때 기본 생성자를 사용하고 생성자를 오버로딩하더라도 오버로딩한 생성자의 사용을 보장할 수 없다. 그리고 기본 생성자가 없다면 당연히 에러가 발생할 것이다.

그렇기 때문에 위처럼 fragment는 반드시 기본 생성자를 사용하고, fragment 객체가 생성되고 난 후 setArguments() 를 통해서 bundle 객체를 입력하는 방식을 사용한다. 

위의 내용을 확인하면 fragment의 인스턴스를 생성하는 과정에 FragmentFactory가 개입하는 것을 알 수 있다.

그리고 [공식문서에서 다음과 같은 내용을 찾을 수 있었다](https://developer.android.com/guide/fragments/fragmentmanager?hl=ko).

> To provide dependencies to your fragment, or to use any custom constructor, you must instead create a custom FragmentFactory subclass and then override `FragmentFactory.instantiate()`.

Fragment에 의존성을 제공하거나, 커스텀 생성자를 사용하기 위해서는 커스텀 FragmentFactory를 구현하고, `FragmentFactory.instantiate()`  를 오버라이딩 하면 된다고 나와있었다.

```kotlin
class DessertsFragment(val dessertsRepository: DessertsRepository) : Fragment() {
    ...
}

class MyFragmentFactory(val repository: DessertsRepository) : FragmentFactory() {
    override fun instantiate(classLoader: ClassLoader, className: String): Fragment =
            when (loadFragmentClass(classLoader, className)) {
                DessertsFragment::class.java -> DessertsFragment(repository)
                else -> super.instantiate(classLoader, className)
            }
}
```

FragmentFactory는 fragment를 생성하기 위해서 기본 생성자만 사용하기 때문에, 커스텀 생성자를 사용하기 위해서는 FragmentFactory를 상속하는 커스텀 FragmentFactory를 구현하고 메서드를 오버라이딩 해주면 된다. 

```kotlin
class MealActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        supportFragmentManager.fragmentFactory = MyFragmentFactory(DessertsRepository.getInstance())
        super.onCreate(savedInstanceState)
    }
}
```

그리고 fragment 가 생성되기 전에 FragmentManager의 FragmentFactory를 커스텀 팩토리로 변경해주면 fragment 객체를 생성할 때 커스텀 생성자를 사용할 수 있게 된다.