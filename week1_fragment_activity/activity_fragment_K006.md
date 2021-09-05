# **Activity란 무엇인가?**

---

[https://developer.android.com/guide/components/activities/intro-activities?hl=ko](https://developer.android.com/guide/components/activities/intro-activities?hl=ko)

# Fragment란 무엇인가?

---

## Fragment 는 왜 등장하게 된 것인가

과거 단말기의 화면이 작았을 때는 Activity만으로 view들을 표현하기에 충분했다.  하지만 태블릿 기기가 등장하고 Activity만으로는 화면을 다양하게 구성하기 힘들어졌다. (초기에는 여러 개의 Activity를 사용하는 방식을 사용했지만 lifecycle 이나 여러 구조적인 문제들로 다루기 힘들었다고 한다.)

그래서 등장한 것이 `Fragment` 이다.

## Fragment

Fragment는 태블릿 등의 큰 화면에서 activity만 사용하는 경우의 복잡성을 해결하기 위해서 등장한 것이라고 하였다. 즉, 이전에 다수의 activity로 표현했던 것들을 fragment로 표현한다는 뜻이다.

다음은 안드로이드 공식 문서의 fragment에 대한 정의이다.

> Fragment는 `FragmentActivity` 내의 어떤 동작 또는 사용자 인터페이스의 `일부`를 나타낸다.

1. **Fragment는 `FragmentActivity` 내부에 존재한다.**

Fragment는 단독으로 존재할 수 없다. 이것이 activity와 fragment의 가장 큰 차이점 중 하나 일 것이다.  반드시 모든 Fragment는 자신을 호출(?) 한 activity가 반드시 존재하게 되어 있다. 

그리고 이는 FragmentActivity 또는 이를 상속받은 activity 클래스가 된다. FragmentActivity를 포함한 하위 클래스부터 fragment 기능을 사용할 수 있다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/a12092c7-7f46-4372-b76e-4c589deecc46/Untitled.png)

내부에 FragmentManager를 호출하는 메서드도 당연히 존재한다.

![스크린샷 2021-09-05 오전 10.47.06.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/7f513014-f6ba-42ab-9477-8754e217e2bb/스크린샷_2021-09-05_오전_10.47.06.png)

Activity를 생성하면 자동으로 AppCompatActivity를 상속하는 클래스를 생성해주는데, 이 또한 

FragmentActivity를 상속하기 때문에 Fragment를 사용할 수 있다.

1. **사용자 인터페이스의 일부를 나타낼 수 있다.**

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/ccfa9cf0-875c-40f0-8072-aba1e4124734/Untitled.png)

fragment는 이처럼 `모듈식`으로 사용자 인터페이스의 일부분을 나타낼 수 있다. 태블릿 기기처럼 화면이 큰 경우에는 화면을 분할하여 이 부분을 fragment로 표현할 수 있다. 

하지만 태블릿이 아닌 우리가 일반적으로 사용하는 스마트폰 같은 모바일 기기의 경우에는 굳이 한 화면을 분할하여 fragment를 사용할 필요가 없다.  그래서 이 경우에는 fragment를 swap-in / swap-out  하는 방식으로 fragment를 사용한다.

이 부분이 내가 헷갈렸던 부분이다. 스마트폰에서는 fragment가 activity와 마찬가지로 화면 하나를 차지한다면,  다수의 activity를 사용하는 것과 다른 점이 무엇일까 고민했었다.

이 부분은 다음 소 주제로  정리하고자 한다.

# Activity vs Fragment

---

위와 같은 상황에서 **fragment**를 사용할 것인지, **activity**를 사용할 것인지 고민이 될 것이다. 자료들을 찾아본 결과 이제는 **fragment**를 많이 사용하고 있다는 내용 뿐, 언제 **fragment**를 사용하고 언제 **activity**를 사용하는 지에 대해서는 명확하게 나와있지 않다.

그래서 나는 **fragment**와 **activity**의 차이점을 이해하고,  **fragment** 가 나온 이유와 연관 해서 사용하면 될 것이라고 생각한다.

소제목이 마음에 들지는 않지만 마땅히 떠오르는 표현이 없어서 그냥 적어보았다. 흔히 대결 구도는 같은 목적으로 사용될 수 있는 것들에 대해 두 가지를 비교하고 더 나은 것을 판별하기 위해서 사용한다. 하지만 밑에서도 나오겠지만 **activity** 와 **fragment** 의 목적을 다르게 보기 때문에, 비교 가능 대상은 아니지만 일단 넘어가자 : )

### 구조적인 차이점

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/4e5b7755-af89-427b-8f87-c4f16c656d84/Untitled.png)

**Activity** 와 **fragment**의 가장 큰 차이점이라 하면, **fragment** 인스턴스는  **activity** 인스턴스에  종속 된다는 것이다. ( **Fragment** 클래스 자체가 **Activity** 클래스에 종속된다는 것은 아니다 )

위에서도 언급했지만, **fragment**는 단독으로 존재하지 않고, **activity**에 종속된다.  다수의 **fragment**가 특정 **Activity**의 **FragmentManager**에 의해서 관리되는 구조이다. 

이 특징이 두 가지를 구분하는 가장 큰 차이점이라고 생각이 된다. 혹자는 이러한 구조적 특성 때문에 **fragment**가 많이 활용 되지만 완벽하게 **activity**를 대체하는 것은 불가능하다고 말한다. 

### **데이터 전달**

두 가지의 구조적인 차이점 때문에 데이터를 전달하고 공유하는 방식 또한 차이가 있다.

 

**Activity**간 데이터를 전달하는 가장 일반적인 방법은 **Intent**를 사용하는 방법이다. **Activity**의 정의에서도 알 수 있듯이 **activity**는 다른 프로세스에서 실행하는 것을 염두하고 설계 되었기 때문에 메모리 영역을 공유하지 않는다. 그렇기 때문에 리눅스 커널 레벨에서 프로세스 간 통신(IPC)을 하게 되는데, 이 부분에서 메모리를 직접 공유하는 것보다 퍼포먼스가 많이 떨어지게 된다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/b5900bc6-a1b6-4176-8a93-faa5c2a9ba1b/Untitled.png)

Fragment간 데이터 전달은 이들은 관리하는 activity를 통해서 자유롭게 이루어진다. 관련된 fragment 들은 하나의 activity에 종속되고, 이 메모리를 공유할 수 있게 된다. [코드 베이스로 frgmanet간의 데이터를 전달하는 방법](https://developer.android.com/training/basics/fragments/communicating)도 있지만, 이는 다소 까다로운 과정을 수반한다.

현재는 ViewModel / Livedata 등의 라이브러리로 데이터 전송을 더 수월하게 진행할 수 있다.

### 퍼포먼스 관점

퍼포먼스 관점에서도 두 가지의 차이점이 존재한다. 앞서 데이터 전송 과정에서도 activity 보다는 fragment 간의 데이터 전송이 퍼모먼스 관점에서 더 좋다고 했었다. 

데이터 전송이 아니라 사용 관점에서도 퍼모먼스 차이가 존재하는 것 같다. 자료를 찾아보니 **fragment**가 **activity**보다는 무겁지 않고, **activity** 백 스택에 **activity** 를 쌓아두는 것보다 **fragment** 백스택에서 **fragment** 를 관리하는 것이 메모리 관리면에서도 효율적이며, 화면전환도  더 순조롭게 할 수 있다고 한다.

### Lifecycle

**Fragment**도 독자적인 **lifecycle**을 가지고 있지만, 이 또한 **activity lifecycle**에 종속된다. 

이 내용에 대해서는 뒤에 다른 소주제로 정리할 것이다.

이렇게 보니 대부분 **fragment**의 장점에 대해서만 언급하였다. 실제로 **activity**를 전환 하는 방식에서 **fragment**를 사용하는 방식으로 많이 변했다고 한다. 

그렇다면 **fragment**가 **activity**를 대체 할 수 있을까?? 이것은 아니라고 생각이 된다.  **fragment** 도 결국 **activity**에 종속되어야 하고, **fragment**가 등장한 배경을 고려하면 **fragment**는 **activity**의 대체제의 성격은 아닌 것 같다. 

**Activity** / **Fragment** 선택하는 기준에 대해서 [가장 설득력이 있었던 자료](https://stackoverflow.com/questions/18500438/activities-vs-fragment-implementation)가 있어서 참고하였다. 

**activity**는 그 역할에 맞게 하나의 주제(작업)에 대한 사용자의 엔트리 포인트로 작용하게 하고, 그 작업의 세부 기능이나 정보에 대해서는 **fragment**를 활용하면 좋을 것 같다는 생각을 했다.

# Lifecycle 종류 및 차이점

---

**Lifecycle** 은 **Android** 작동 방식의 핵심으로, 이를 준수하지 않으면 메모리 누출 또는 애플리케이션의 비정상 종료가 발생할 수 있다.

그렇기 때문에 어떤 상황에서 어떤 **lifecycle** 상태를 가지는 것인지 이해하는 것은 중요하다.

## Activity Lifecycle

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/81fe830f-a634-4ec4-bd73-12781d3cf05f/Untitled.png)

**Activity** 클래스는 **lifecycle** 상태에 대한 6가지 **lifecycle callback** 메서드를 가지게 된다.

Callback 메서드의 종류는 다음과 같다.

`onCreate()`  `onStart()`  `onResume()`  `onPause()`  `onStop()`  `onDestroy()`

### onCreate()

**onCreate()** 콜백은 시스템이 **activity**를 생성할 때 실행되는 것으로, 필수적으로 구현해야 한다. **Activity**가 생성되고 나면 **created** 상태가 되고, **onCreate()** 콜백이 호출 된다.

이때 **LifecycleObserver**는 `**ON_CREATE**` 이벤트를 수신하게 된다.

이는 **activity**가 생성되는 시점에 호출이 되기 때문에, **activity**의 전체 **lifecycle** 동안 **한 번만** 발생해야 하는 기본 애플리케이션 시작 로직을 실행한다.

보통 `setContentView()` 로 레이아웃을 전달하여 화면을 정의하거나, `데이터 바인딩`, activity와  `ViewModel` 연결 그리고 일부 클래스를  `인스턴스화` 하는 작업을 한다.

그리고 **onCreate()** 메서드는 `savedInstanceState` 매개변수를 받는데, 이는 **activity**의 이전 저장 상태가 포함된 **Bundle** 객체이다. 이번에 처음 생성된 활동인 경우 **Bundle** 객체의 값은 **null이다**.

일반적으로 앱을 실행하면 **onCreate** - **onStart** - **onResume** 콜백까지 빠르게 호출된다.

### onStart()

**Activity**가 생성되고 나서 시작되면 시스템은 **onStart()** 콜백을 호출한다. **onStart()**가 호출되면 **activity**가 사용자에게 표시되고, 앱은 **activity**를 **foreground**에 보내 상호작용할 수 있도록 준비한다.

이 콜백에서는 주로 `UI를 관리`하는 코드를 초기화한다. ( 애니메이션, 데이터 갱신 시작)

마찬가지로 이때 **LifecycleObserver** 는 **`ON_START`** 이벤트를 수신하게 되고, 이어서 시스템이 **onResume()** 콜백을 이어서 호출하게 된다.

### onResume()

**Activity** 가 **Resumed** 상태가 되면 **foreground** 에 표시되고 **onResume()** 콜백이 호출이 된다. **Resumed** 상태에서 **activity**는 **foreground** 에 존재하게 되고, 사용자와 상호작용이 가능하다.

그리고 오직 하나의 **activity**만 **Resumed** 상태일 수 있다.

여기서 나는 **Started** 상태와 **Resumed** 상태의 차이점이 헷갈렸다. 우선 두 상태 모두 **foreground**에 존재한다고 볼 수 있다. 즉, 우리의 화면에 두 **activity**가 모두 보이는 상태인 것이다. 이는 **다중 창 모드(multi-window)** 와 **오버뷰(over view)** 화면이 이에 해당 될 것이다. 

여기서 **Resumed** 상태는 하나의 **activity**만 가능한 점을 고려해볼 때 현재 사용자가 상호작용 가능한 **focus** 상태의 **activity**를 말하는 것이고, 이 **activity**의 상태가 **Resumed** 상태인 것이다.

마찬가지로 이때 **LifecycleObserver**는 `**ON_RESUME**` 이벤트를 수신하게 된다.

### onPause()

**Activity**를 떠나는 것을 나타내는 첫 번째 신호로 이 콜백을 호출한다. **Activity**가 **foreground**에 있지 않게 되었다는 것을 의미한다. 하지만 사용자가 다중 창 모드에 있을 경우에는 여전히 화면에 표시 될 수도 있다.  ( 여기서 **foreground** 라는 개념이 헷갈린다.. 안드로이드 공식 문서에 따르면 **Pause** 상태로 간다는 것은 foreground가 아니라는 것을 의미하는데, 사실상 눈에는 보이게 되는 것이다.)

**onResume** 상태는 단 하나의 **activity**만 가질 수 있기 때문에, 화면에 보이지만 **background** 상태에(?) 있는 **activity**는 **pause** 상태에 있는 것이다.**( lost focus)**

- **onPause()** 콜백에서 수행할 수 있는 기능

    **Activity**가 일시중지(Pause) 중이고 사용자가 필요로 하지 않을 때 배터리 수명에 영향을 미칠 수 있는 모든 리소스를 해제할 수 있다. 하지만 위에서도 언급했듯이 일시중지된 **activity**는 다중 창 모드에서 여전히 완전히 보이는 상태일 수 있다. 그러므로 UI 관련 리소스와 작업을 완전히 해제하거나 조정할 때는 **onPause()** 대신 **onStop()**을 사용하는 것이 좋다.

### onStop()

**Activity**가 사용자에게 보이지 않는 **Stopped** 상태에서 호출되는 콜백이다. 현재 **activity** 가 다른 **activity**에 의해서 완전히 가려 보이지 않는 경우가 **Stopped** 상태에 해당한다. 홈 화면으로 이동하여 **activity**가 보이지 않는 경우도 여기에 해당한다. 그리고 **activity**의 실행이 완료되어 종료될 시점에도 **onStop()**을 호출할 수 있다.

**onStop()** 콜백에서는 앱이 사용자에게 보이지 않는 동안 앱에 필요하지 않은 리소스를 해제하거나 조정해야 한다. 그리고 이 때는 부하가 큰 작업을 하기에 적당하다. 예를 들어 데이터 저장과 관련된 데이터 베이스 트랜잭션 등의 작업을 처리하기 좋다.

**Stopped** 상태에서는 시스템이 두 가지의 메서드를 호출할 수 있다. 만약 **activity** 가 실행을 종료하게 된다면 **onDestroy()** 콜백을 호출하고 **activity**의 자원을 완전히 반환하여 종료한다. 그게 아니고 다시 **activity**가 재실행이 된다면 **onRestart()** - **onStart()** 를 호출하고 **Started** 상태부터 다시 **lifecycle**을 진행한다.

### onDestroy()

**onDestroy()** 콜백은 **activity**가 소멸되기 **전**에 호출된다. **Activity**의 **lifecycle**에서 가장 마지막으로 호출되는 콜백이다. 시스템은 다음 중 하나에 해당할 때 이 콜백을 호출한다.

1. (사용자가 **activity**를 완전히 닫거나 **activity**에서 **finish()**가 호출되어) **activity**가 종료되는 경우
2. 구성 변경(예: 기기 회전 또는 다중 창 모드)으로 인해 시스템이 일시적으로 **activity**를 소멸시키는 경우

**Activity**가 소멸될때 **ViewModel** 객체를 이용하면 **Activity**의 뷰 데이터를 보관할 수 있다. 이렇게 **ViewModel**에 저장된 뷰 데이터는 화면 회전 등으로 인해서 **activity** 가 소멸될 경우 뷰 데이터를 그대로 사용할 경우 유용하다. ( 이 부분은 **ViewModel**에 대해서 학습해보면 더 자세히 알 수 있을 것 같다.)

그리고 화면 회전 등의 구성 변경으로 **Activity**가 소멸된 경우에는 바로 새로운 **Activity**가 **onCreate()**된다. 이전의 onStop() 콜백에서 아직 해제되지 않은 자원이 있다면 onDestroy() 콜백에서 모두 해제해야한다.

## Fragment Lifecycle

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/593e9812-39d6-485b-b2d3-8bf0aae46e1f/Untitled.png)

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

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/54d09843-2067-4046-ada9-aeecc9b25d73/Untitled.png)

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