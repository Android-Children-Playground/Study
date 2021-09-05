# Week1-2. Lifecycle 차이점 각 Lifecycle에 맞는 동작은 뭐가 있을까?

## Lifecycle 차이점 각 Lifecycle에 맞는 동작은 뭐가 있을까?

### Activity LifeCycle

- `onCreate()` `onStart()` `onResume()` `onPause()` `onStop()` `onDestroy()`

    <img width="337" alt="스크린샷_2021-09-05_오후_4 01 06" src="https://user-images.githubusercontent.com/68374234/132122018-1c85c26e-e4b8-499b-af9b-70ecd0bf1f71.png">

    출처 : [Activity 수명주기 - Android Developers 공식문서](https://developer.android.com/guide/components/activities/activity-lifecycle?hl=ko)

    <img width="721" alt="스크린샷_2021-09-05_오후_4 10 58" src="https://user-images.githubusercontent.com/68374234/132122021-8c819267-692f-468a-854d-6f86a0bea81b.png">출처 : [Android 활동 수명 주기를 구성하는 상태 및 이벤트](https://developer.android.com/topic/libraries/architecture/lifecycle?hl=ko)

- `setContentView()` 호출 등 일부 작업은 Activity LifeCycle 메소드에 속해있음.
- 그러나 종속적인 구성요소의 작업을 구현하는 코드는 해당 구성요소 안에 넣어야함.
    - 이를 위해서 종속적인 구성요소가 수명주기를 인식하도록 해야함
    - [종속적인 구성요소가 수명주기를 인식하게 하는 방법](https://developer.android.com/topic/libraries/architecture/lifecycle?hl=ko)
- `**onCreate()**`
    - Activity를 생성할 때 실행되는 것
    - Activity 가 생성되면 `CREATED` 상태 시작
    - Activity의 전체 수명 주기 동안 한번만 발생해야 하는 기본 애플리케이션 시작 로직을 실행
        - ex) 데이터를 목록에 바인딩, Activity를 ViewModel과 연결, 일부 클래스 범위 변수를 인스턴스화
    - savedInstanceState 매개 변수를 수신함. 이전 저장 상태가 포함된 Bundle 객체임. 처음 생성된 Activity인 경우 Bundle 객체의 값은 null
    - Activity의 수명주기와 연결된 수명주기 인식 구성요소가 있다면 이 구성요소는 `ON_CREATE` 이벤트를 수신
        - @OnLifecycleEvent 라는 주석이 있는 메소드가 호출됨. 수명주기 인식 구성요소는 `CREATED` 상태에 필요한 모든 설정 코드를 실행할 수 있게 된다.
    - Activity는 `**CREATED` 상태에 머무르지 않음.**
        - `onCreate()` 메소드가 실행을 완료하면 `STARTED` 상태가 되고, 시스템이 연달아 `onStart()` 와 `onResume()` 메소드를 호출함.

        > 이 때문에 onStart()에서 메소드를 두지 말라고 한 것 같다는 생각이든다...

- `**onStart()**`
    - `**onCreate()`  메소드 실행 완료 → `STARTED` 상태가 됨 → 시스템이 `onStart()` 호출**
    - `onStart()` 가 호출되면
        - Activity가 사용자에게 보여짐
        - 앱은 Activity를 포그라운드에 보내 상호작용할 수 있도록 준비 ex) 메소드에 앱이 UI를 관리하는 코드를 초기화
    - Activity가 `STARTED` 상태로 전환하면 → Activity의 수명주기와 연결된 모든 수명주기 인식 구성요소는 `ON_START` 이벤트를 수신
    - `onStart()` 는 **매우 빠르게** 완료됨, `**CREATED` 상태와 마찬가지로 Activity는 `STARTED` 상태에 머무르지 않음.**
- `**onResume()**`
    - `**onStart()` 메소드 실행 완료 → `RESUMED` 상태에 들어감 → 시스템이 `onResume()` 호출**
    - `RESUMED` 상태에 들어갔을 때 앱이 사용자와 상호작용함
    - **앱에서 포커스가 떠날 때까지 앱은 `RESUMED` 상태에 머무름**
        - 포커스가 떠나는 이벤트 ex) 전화옴, 사용자가 다른 Activity로 이동함, 기기 화면이 꺼짐
    - Activity가 `RESUMED` 상태로 전환되면 Activity와 연결된 모든 수명주기 인식 구성요소는 `ON_RESUME` 이벤트를 수신
    - `RESUMED` 상태에서 수명주기 구성요소가 포그라운드에서 사용자에게 보여지는 동안 실행해야 하는 모든 기능을 활성화할 수 있음.
    - `**RESUMED` 상태에서 방해되는 이벤트 발생 → Activity는 `PAUSED` 상태에 들어감 → 시스템이 `onPause()` 를 호출**
    - `**PAUSED` 에서 `RESUMED` 상태로 돌아옴 → 시스템이 `onResume()` 호출**
        - `onResume()` 에서 구현하고 `onPause()` 중에 해제한 구성요소를 초기화해야함.
        - Activity가 `RESUMED` 상태로 전환될 때마다 필요한 다른 초기화 작업을 수행해야함.
- `**onPause()**`
    - `**RESUMED` 상태에서 방해되는 이벤트 발생 → Activity는 `PAUSED` 상태에 들어감 → 시스템이 `onPause()` 를 호출**
    - 시스템은 사용자가 Activity를 떠나는 것을 나타내는 첫 번째 신호로 `onPuase()` 를 호출, 해당 Activity가 항상 소멸되는 것은 아님. → Activity가 포그라운드에 있지 않게 되었다는 것이지만 멀티윈도우 환경에서는 여전히 표시될 수 있음.
    - `onPause()` 를 사용하여 Activity가 `PAUSED` 상태일 때 계속 실행되어서는 안되지만 잠시 후 다시 시작할 작업을 일시중지하거나 조정함.
    - `PAUSED` 상태에 들어가는 경우 ex) 다이얼로그가 띄워졌을 때, 멀티윈도우 모드에서 포커스를 잃었을 때 등
    - Activity가 `PAUSED` 상태로 전환되면 Activity와 연결된 모든 수명주기 인식 구성요소는 `ON_PAUSE` 이벤트를 수신
    - `**onPuse()` 메소드 실행이 완료됨 → Activity가 다시 시작되거나 사용자에게 완전히 보여질 때까지 Activity의 상태는 `PAUSED` 에 머무름.**

        **→ Activity 다시 시작 됨 →  `RESUMED` 상태로 돌아옴 → `onResume()` 호출**

        **→ Activity가 완전히 보이지 않게 됨 → `onStop()` 호출**

- `**onStop()**`
    - **Activity가 사용자에게 더이상 표시되지 않음 → `STOPPED` 상태에 들어감 → 시스템이 `onStop()` 호출**
    - ex) 새로 시작된 Activity가 화면 전체를 차지할 경우
    - 시스템은 Activity의 실행이 완료되어 종료될 시점에도 `onStop()`을 호출
    - Activity가 `STOPPED` 상태로 전환되면 Activity와 연결된 모든 수명주기 인식 구성요소는 `ON_STOP` 이벤트를 수신
    - 
- `**onDestroy()**`

- 어떤 빌드업 이벤트에서 초기화 작업을 실행하든 빌드업 이벤트에 상응하는 수명 주기 이벤트를 사용하여 리소스를 해제하세요. `ON_START` 이벤트 이후에 무언가를 초기화하는 경우, `ON_STOP` 이벤트 이후에 이를 해제하거나 종료하세요. `ON_RESUME` 이벤트 이후에 초기화하는 경우, `ON_PAUSE` 이벤트 이후에 해제하세요.

### Fragment LifeCycle

-