# Week1-2. Lifecycle 차이점 각 Lifecycle에 맞는 동작은 뭐가 있을까?

## Lifecycle 차이점 각 Lifecycle에 맞는 동작은 뭐가 있을까?

### Activity LifeCycle

- `onCreate()` `onStart()` `onResume()` `onPause()` `onStop()` `onDestroy()`

    <img width="337" alt="스크린샷_2021-09-05_오후_4 01 06" src="https://user-images.githubusercontent.com/68374234/132122018-1c85c26e-e4b8-499b-af9b-70ecd0bf1f71.png"> <br>

    출처 : [Activity 수명주기 - Android Developers 공식문서](https://developer.android.com/guide/components/activities/activity-lifecycle?hl=ko)

    <img width="721" alt="스크린샷_2021-09-05_오후_4 10 58" src="https://user-images.githubusercontent.com/68374234/132122021-8c819267-692f-468a-854d-6f86a0bea81b.png"> <br>
    출처 : [Android 활동 수명 주기를 구성하는 상태 및 이벤트](https://developer.android.com/topic/libraries/architecture/lifecycle?hl=ko)

- `setContentView()` 호출 등 일부 작업은 Activity LifeCycle 메소드에 속해있음.
- 그러나 종속적인 구성요소의 작업을 구현하는 코드는 해당 구성요소 안에 넣어야함.
    - 이를 위해서 종속적인 구성요소가 수명주기를 인식하도록 해야함
    - [종속적인 구성요소가 수명주기를 인식하게 하는 방법](https://developer.android.com/topic/libraries/architecture/lifecycle?hl=ko)

<br>

- `onCreate()`
    - Activity를 생성할 때 실행되는 것
    - Activity 가 생성되면 `CREATED` 상태 시작
    - Activity의 전체 수명 주기 동안 한번만 발생해야 하는 기본 애플리케이션 시작 로직을 실행
        - ex) 데이터를 목록에 바인딩, Activity를 ViewModel과 연결, 일부 클래스 범위 변수를 인스턴스화
    - savedInstanceState 매개 변수를 수신함. 이전 저장 상태가 포함된 Bundle 객체임. 처음 생성된 Activity인 경우 Bundle 객체의 값은 null
    - Activity의 수명주기와 연결된 수명주기 인식 구성요소가 있다면 이 구성요소는 `ON_CREATE` 이벤트를 수신
        - @OnLifecycleEvent 라는 주석이 있는 메소드가 호출됨. 수명주기 인식 구성요소는 `CREATED` 상태에 필요한 모든 설정 코드를 실행할 수 있게 된다.
    - **Activity는 `CREATED` 상태에 머무르지 않음.**
        - `onCreate()` 메소드가 실행을 완료하면 `STARTED` 상태가 되고, 시스템이 연달아 `onStart()` 와 `onResume()` 메소드를 호출함.

        > 이 때문에 onStart()에서 메소드를 두지 말라고 한 것 같다는 생각이든다...
    - **onCreate()`에서 해야하는 동작**
        > 뷰가 그려질 때 해야하는 동작들을 여기서 해야하지 않을까...?
        - setContentView()는 `onCreate()`에 속해있다.
        ```kotlin
        @Override
        public void setContentView(View view) {
            initViewTreeOwners();
            getDelegate().setContentView(view);
        }
        private void initViewTreeOwners() {
            // Set the view tree owners before setting the content view so that the inflation process
            // and attach listeners will see them already present
            ViewTreeLifecycleOwner.set(getWindow().getDecorView(), this);
            ViewTreeViewModelStoreOwner.set(getWindow().getDecorView(), this);
            ViewTreeSavedStateRegistryOwner.set(getWindow().getDecorView(), this);
        }
        @NonNull
        public AppCompatDelegate getDelegate() {
            if (mDelegate == null) {
                mDelegate = AppCompatDelegate.create(this, this);
            }
            return mDelegate;
        }

        /**
         * Should be called instead of {@link Activity#setContentView(android.view.View)}}
         */
        public abstract void setContentView(View v);
        ```

<br>

- `onStart()`
    - **`onCreate()`  메소드 실행 완료 → `STARTED` 상태가 됨 → 시스템이 `onStart()` 호출**
    - `onStart()` 가 호출되면
        - Activity가 사용자에게 보여짐
        - 앱은 Activity를 포그라운드에 보내 상호작용할 수 있도록 준비 ex) 메소드에 앱이 UI를 관리하는 코드를 초기화
    - Activity가 `STARTED` 상태로 전환하면 → Activity의 수명주기와 연결된 모든 수명주기 인식 구성요소는 `ON_START` 이벤트를 수신
    - **`onStart()` 는 매우 빠르게 완료됨, `CREATED` 상태와 마찬가지로 Activity는 `STARTED` 상태에 머무르지 않음.**
    - **`onStart()`에서 해야하는 동작**
        > 매우 빠르게 완료되므로 이 `onStart()`에서는 최대한 코드를 두지 않는 것이 좋을 것 같다.

<br>

- `onResume()`
    - **`onStart()` 메소드 실행 완료 → `RESUMED` 상태에 들어감 → 시스템이 `onResume()` 호출**
    - `RESUMED` 상태에 들어갔을 때 앱이 사용자와 상호작용함
    - **앱에서 포커스가 떠날 때까지 앱은 `RESUMED` 상태에 머무름**
        - 포커스가 떠나는 이벤트 ex) 전화옴, 사용자가 다른 Activity로 이동함, 기기 화면이 꺼짐
    - Activity가 `RESUMED` 상태로 전환되면 Activity와 연결된 모든 수명주기 인식 구성요소는 `ON_RESUME` 이벤트를 수신
    - `RESUMED` 상태에서 수명주기 구성요소가 포그라운드에서 사용자에게 보여지는 동안 실행해야 하는 모든 기능을 활성화할 수 있음.
    - **`RESUMED` 상태에서 방해되는 이벤트 발생 → Activity는 `PAUSED` 상태에 들어감 → 시스템이 `onPause()` 를 호출**
    - **`PAUSED` 에서 `RESUMED` 상태로 돌아옴 → 시스템이 `onResume()` 호출**
    - **`onResume()`에서 해야하는 동작**
        - `onResume()` 에서 구현하고 `onPause()` 중에 해제한 구성요소를 초기화해야함.
        - Activity가 `RESUMED` 상태로 전환될 때마다 필요한 다른 초기화 작업을 수행해야함.

<br>

- `onPause()`
    - **`RESUMED` 상태에서 방해되는 이벤트 발생 → Activity는 `PAUSED` 상태에 들어감 → 시스템이 `onPause()` 를 호출**
    - 시스템은 사용자가 Activity를 떠나는 것을 나타내는 첫 번째 신호로 `onPuase()` 를 호출, 해당 Activity가 항상 소멸되는 것은 아님. → Activity가 포그라운드에 있지 않게 되었다는 것이지만 멀티윈도우 환경에서는 여전히 표시될 수 있음.
    - `PAUSED` 상태에 들어가는 경우 ex) 다이얼로그가 띄워졌을 때, 멀티윈도우 모드에서 포커스를 잃었을 때 등
    - Activity가 `PAUSED` 상태로 전환되면 Activity와 연결된 모든 수명주기 인식 구성요소는 `ON_PAUSE` 이벤트를 수신
    - **`onPuse()` 메소드 실행이 완료됨 → Activity가 다시 시작되거나 사용자에게 완전히 보여질 때까지 Activity의 상태는 `PAUSED` 에 머무름.**

        **→ Activity 다시 시작 됨 →  `RESUMED` 상태로 돌아옴 → `onResume()` 호출**

        **→ Activity가 완전히 보이지 않게 됨 → `onStop()` 호출**
    - **`onPause()`에서 해야하는 동작**
        - Activity가 `PAUSED` 상태일 때 계속 실행되어서는 안되지만 잠시 후 다시 시작할 작업을 일시중지하거나 조정함.**

<br>

- `onStop()`
    - **Activity가 사용자에게 더이상 표시되지 않음 → `STOPPED` 상태에 들어감 → 시스템이 `onStop()` 호출**
    - ex) 새로 시작된 Activity가 화면 전체를 차지할 경우
    - 시스템은 Activity의 실행이 완료되어 종료될 시점에도 `onStop()`을 호출
    - Activity가 `STOPPED` 상태로 전환되면 Activity와 연결된 모든 수명주기 인식 구성요소는 `ON_STOP` 이벤트를 수신
    - `onPuase()` 대신 `onStop()`을 사용하면 -> 사용자가 멀티윈도우모드에서 Activity를 보고있더라도 UI관련 작업이 계속 진행됨
    - **`onStop()`에서 해야하는 동작**
        - CPU를 비교적 많이 소모하는 종료작업
        - 정보를 데이터베이스에 저장
        - 앱이 사용자에게 보이지 않는동안 앱은 필요하지 않는 리소스를 해제하거나 조정해야함

<br>

- `onDestroy()`
    - 활동이 소멸되기 전에 호출됨
    - **사용자가 Activity를 완전히 닫거나 Activity에서 finish()가 호출되어 종료되는 경우 -> 시스템이 `onDestroy()`를 호출**
    - **기기회전 또는 멀티윈도우모드로 인해 시스템이 일시적으로 Activity를 소멸시키는 경우 -> 시스템이 `onDestroy()`를 호출**
    - Activity가 `DESTROYED` 상태로 전환 -> Activity와 연결된 모든 수명주기 인식 구성요소는 `ON_DESTROY` 이벤트를 수신
    - **Activity가 기기회전 또는 멀티윈도우모드로 인해 다시 생성될 경우** -> ViewModel은 그대로 보존되어 다음 Activity 인스턴스에 전달됨 -> 추가작업이 필요하지 않음.
    - **Activity가 다시 생성되지 않을 경우** -> ViewModel은 onCleared() 메소드를 호출해서 Activity가 소멸되기 전에 모든 데이터를 정리해야함.
    - 위의 2가지 경우는 isFinishing() 메소드로 구분 가능
        - isFinishing() : finish()를 호출해서 Activity가 finishing 과정에 있는 것인지, 완료됐다고 요청했기 때문에 finishing 꽈정에 있는 것인지 체크한다. Activity가 단순히 일시정지 상태인지 완전히 완료 상태인지를 결정하기 위해 종종 `onPuase()`에서 사용됨.
    - Activity가 종료되는 경우 -> `onDestroy()` 는 Activity 가 수신하는 마지막 수명주기 콜백
    - 기기회전 또는 멀티윈도우모드로 인해  `onDestroy()`가 호출되는 경우 -> 시스템이 즉시 새 Activity 인스턴스를 생성 -> 새로운 구성에서 새로운 인스턴스에 대해 `onCreate()`를 호출

- 어떤 빌드업 이벤트에서 초기화 작업을 실행하든 빌드업 이벤트에 상응하는 수명 주기 이벤트를 사용하여 리소스를 해제하세요. `ON_START` 이벤트 이후에 무언가를 초기화하는 경우, `ON_STOP` 이벤트 이후에 이를 해제하거나 종료하세요. `ON_RESUME` 이벤트 이후에 초기화하는 경우, `ON_PAUSE` 이벤트 이후에 해제하세요.

<br>

### Fragment LifeCycle
<img width="397" alt="스크린샷_2021-09-05_오후_3 32 32" src="https://user-images.githubusercontent.com/68374234/132120640-b2a1bf10-e198-4d42-9e93-9c58a7b8d31d.png">

- `onAttach()` 
    - Fragment가 Activity에 붙을 때 호출 됨, 
    - `CREATED`상태일 때 이미 호출되었을 것, 
    - 자원을 가져다 쓸 때 onAttach()를 override하여 사용

<br>

- Fragment`CREATED` `onCreate()`
    - Activity와 마찬가지로 초기화해야하는 리소스들을 여기서 초기화함
    - Fragment를 생성하면서 넘겨준 값들이 있다면, 여기서 변수에 넣어주면 됨
    - But, 여기서 UI는 초기화할 수 없음 <- 아직 View가 그려진 것은 아님
    - **Activity에서는 `onCreate()`에서 View나 UI관련 작업을 할 수 있는 것과 달리 Fragment에서는 할 수 없음 -> onCreateView()에서 UI관련 작업을 할 수 있음**

<br>

- Fragment`CREATED` View`INITIALIZED` `onCreateView()` `onViewCreated()` 
    - fragment에서 UI를 그릴때 호출되는 콜백 
    - 레이아웃을 inflate하는 곳
    - View 객체를 얻을 수 있음 -> 버튼이나 텍스트뷰 등 UI작업을 할 수 있음
    - Fragment가 자신의 인터페이스를 처음 그리기 위해 호출함
    - View를 반환해야함, `onCreateView()`는 Fragment의 레이아웃 루트이기 때문에 UI를 제공하지 않는 경우 null을 반환하면 됨.
    - Fragment에 속한 각종 View나 ViewGroup에 대한 UI 바인딩 작업을 할 수 있음 (Layout을 inflate 하여 View작업을 함)
    - `onCreateView()`의 매개변수로 전달되는 contatiner == Activity의 ViewGroup
        - container에 Fragment가 위치하게 됨
    - `onCreateView()`의 매개변수로 전달되는 savedInstanceState : Bundle 객체로 Fragment가 재개되는 경우 이전 상태에 대한 데이터를 제공함
    - `onCreateView()`를 통해 반환된 View 객체는 `onViewCreated()`의 파라미터로 전달됨
    - **이 시점 부터 Fragment의 View의 Lifecycle이 `INITIALIZED` 상태로 업데이트** 되었기 때문에 View의 초기값을 설정해주거나 LiveData 옵저빙, RecyclerView 또는 ViewPager2에 사용될 Adapter 세팅 등을 여기서 해주는 것이 좋다.
        > 그래서 이 시점이라는 것이 `onCreateView()`에서 하라는건지 `onViewCreated()`하라는건지 모르겠다...

<br>

- Fragment, View`CREATED` `onViewStateRestored()` 
    - 저장해둔 모든 state 값이 Fragment의 View 계층 구조에 복원 됐을 때 호출됨
    - **이때 Fragment의 View의 Lifecycle이 `INITIALIZED` 상태에서 `CREATED` 상태로 변경됐음을 알림**

<br>

- Fragment, View`STARTED` `onStart()`
    - Fragment가 사용자에게 보여질 수 있을 때 호출됨
    - Fragment가 attach 되어있는 Activity의 onStart() 시점과 유사함
    - 이 시점부터 Fragment의 childFragmentManager를 통해 FragmentTransaction을 안전하게 수행할 수 있음
    - **Fragment의 View의 Lifecycle도 `STARTED` 상태로 변경됨**

<br>

- Fragment, View`RESUMED` `onResume()`
    - Fragment가 보이는 상태에서 모든 Animator와 Transaction 효과가 종료되고, 프래그먼트가 사용자와 상호작용할 수 있을 때 `onResume()`이 호출됨
    - `onStart()`와 마찬가지로 주로 Activity onResume() 시점과 유사함
    - `RESUMED` 상태가 됐다는 것은 사용자가 프래그먼트와 상호작용하기에 적절한 상태가 된 것 == `onResume()`이 호출되지 않은 시점에서는 입력을 시도하거나 포커스를 설정하는 등의 작업을 임의로 하면 안된다는 것을 의미

<br>

- Fragment, View`STARTED` `onPause()`
    - 사용자가 Fragment를 떠나기 시작했지만 Fragment는 여전히 visible일 때 `onPause()` 호출됨
    - **Fragment와 View의 Lifecycle이 `PAUSED`가 아닌 `STARTED`가 됨**

<br>

- Fragment, View`CREATED` `onStop()`
    - **Fragment가 더이상 화면에 보여지지 않게 되면 Fragment와 View의 Lifecycledms `CREATED` 상태가 되고 `onStop()`이 호출됨**
    - 이 상태는 부모 Activity나 Fragment 중단됐을 때 뿐만아니라, 부모 Activity나 Fragment의 상태가 저장될 때도 호출됨.
    - API 28버전부터 `onStop()`이 `onSavedInstanceState()`보다 먼저 호출됨 -> `onStop()`이 FragmentTransaction을 안전하게 수행할 수 있는 마지막 지점이 되었음
    <img width="601" alt="스크린샷 2021-09-06 오전 1 46 48" src="https://user-images.githubusercontent.com/68374234/132134744-a3453337-db3b-4f55-ad5c-a9c8c497d971.png">

<br>

- Fragment, View`CREATED` `onSaveInstanceState()`

<br>

- View`CREATED` Fragment `DESTROYED` `onDestroyView()`
    - 모든 exit animation과 transaction이 완료되고, Fragment가 화면으로부터 벗어났을 경우 Fragment의 View의 LifeCycle은 `DESTROYED`가 되고 `onDestroy()`가 호출됨
    - 이 시점 부터는 getViewLifecycleOwnerLiveData()의 리턴값으로 null이 반환됨
    - 해당 시점에서는 Garbage Collector에 의해 수거될 수 있도록 Fragment View에 대한 모든 참조가 제거되어야 함

<br>

- Fragment`DESTROYED` `onDestroy()
    - Fragment가 제거되거나 FragmentManager가 destroy 됐을 경우, Fragment의 Lifecycel은 `DESTROYED`상태가 되고 `onDestroy()`가 호출됨
    - 해당 지점은 Fragment의 Lifecycle의 끝을 알림
    - `onAttach()`가 `onCreate()` 이전에 호출됐던 것처럼 `onDestroy()`이후에 `onDettach()` 호출됨

<br>

참고 : Android Developers 공식문서, https://asong-study-record.tistory.com/69, https://readystory.tistory.com/199
