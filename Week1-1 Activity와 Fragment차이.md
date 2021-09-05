# Week1-1. Activity와 Fragment의 차이

## Activity, Fragment의 차이

### Activity

- Activity는 main()메서드를 사용하여 앱을 실행하는 프로그래밍 패러다임과는 다른 안드로이드 시스템의 특수성에 의하여 생긴 개념이다.
- 모바일 앱 환경은 사용자와 앱의 상호작용이 항상 동일한 위치에서 시작되는 것이 아니라는 점에서 데스크톱 앱 환경과 다르다.
    - ex )홈 화면에서 이메일 앱을 열면 이메일 목록이 표시될 수 있다. 이에 반대로 소셜 미디어 앱을 사용하고 있는 상태에서 이메일 앱을 실행하면 이메일을 작성하기 위한 이메일 앱 화면으로 바로 이동할 수 있다.
- 한 앱이 다른 앱을 호출할 때 그 앱의 전체를 호출하는 것이 아니라 그 앱의 특정 Activity를 호출하는 것이다.
- Activity는 앱과 사용자의 상호작용을 위한 진입점 역할과 동시에 하나의 UI화면을 그리는 Contatiner역할을 수행한다.
- 일반적으로 하나의 Activity는 앱에서 하나의 화면을 구현함.
    - 대부분의 앱에는 여러 화면이 포함되어 있음 >> 대부분의 앱은 여러 Activity로 구성됨.
    - 일반적으로 앱에서 하나의 Activity가 기본 Activity로 지정됨. 이 기본 Activity가 앱을 실행할 때 표시되는 첫번째 화면 (AndroidManifest에서 설정)
    - 각 Activity는 다양한 Activity를 실행하기 위해 또 다른 Activity를 시작할 수 있다.
- 각 Activity는 다른 Activity에 대해 느슨하게 결합됨. 최소한의 종속성만 있음.
- 앱의 Activity를 사용하려면 manifest에 Activity 관련 정보를 등록하고 Activity 수명 주기를 적절히 관리해야한다.

### Fragment

- 앱 UI의 재사용 가능한 부분을 나타낸다.
- 자체 레이아웃을 정의 및 관리하고 자체 수명주기를 보유하며 자체 입력이벤트를 처리할 수 있다.
- 독립적으로 존재할 수 없고 Activity나 다른 Fragment에서 호스팅 되어야한다.
- Fragmentdml 뷰 계층 구조는 호스트 뷰 계층 구조의 일부가 되거나 연결됩니다.
- **모듈성**
    - Fragment는 생명주기, 레이아웃, 입력 이벤트를 가지는 Activity의 모듈식 섹션 / 레이아웃, 동작처리, 생명주기를 가지는 독립적인 모듈
    - UI를 개별 영역으로 분할할 수 있도록 하여 Activity의 UI에 모듈성과 재사용성을 도입한다.
    - Activity는 앱의 사용자 인터페이스 주위에 전역 요소를 배치하기에 적합
    - Fragment는 단일 화면이나 화면 일부의 UI를 정의하고 관리하는데 더 적합
    - UI를 Fragment로 나누면 런타임시 Activity의 모양을 더 쉽게 수정할 수 있다.
    - Activity가 STARTED 이상에 있는 동안 프래그먼트를 추가, 교체, 삭제 할 수 있다.
        - 이런 변경사항기록을 Activitiy에서 관리하는 백 스택에 보관할 수 있으므로 변경사항을 취소할 수 있다.
    - 동일한 Activity내에서, 여러 Activity에서 또는 다른 Fragment의 하위요소로도 동일한 Fragment 클래스의 여러 인스턴스를 사용할 수 있다.

### Fragment의 필요성?

- Activity의 한계 때문
- Fragment를 사용하면 사용자는 액티비티를 변경하지 않고도 View를 변경할 수 있다.
- Fragment는 화면 안에 들어가는 레이아웃이 중복되지 않도록 한번만 정의하고 재사용이 가능하도록 만든 것.
- Framgent를 통해 하나의 화면을 여러부분으로 나누어 보여줄 수 있음(태블릿), 분할된 화면들을 독립적으로 구성하고 상태를 관리하기 위해 만들어졌음. (Fragment가 처음 등장한 것도 태블릿 때문이었음. Activity만으로는 화면을 다양하게 구성하기 힘들어졌고, 초기에 Activity를 중첩하여 넣었지만 생명주기나 여러 구조적인 문제를 겪음.)

### Activity와 Fragment의 차이점

- Fragment는 Activity와 View의 개념을 합쳐놓은 것이라고 할 수 있다.
- **Fragment는 Activity와 View의 개념을 다 가지고 있지만** 기본적으로 설정이 되어있는 계층적인 관계는 변함이 없다.
- Activity는 가장 밑부분에 존재하는 틀, View들은 Activity가 있어야만 존재할 수 있다. Fragment도 마찬가지.
- 그런데 Fragment는 Activity의 성질도 갖고 있으므로 Fragment위에도 View가 존재할 수 있다. Fragment는 Activity와 View의 중간성질.
- Fragment는 View가 아니기 때문에 findViewById() 메소드를 즉각 호출하는 것이 불가능하다. getSupportFragmentManager()를 호출해서 FragmentManager 객체를 불러와서 findViewById()를 사용해야 한다.
- Activity가 onCreate()할 때 : Fragment는 onAttach() -> onCreate() -> onCreateView() -> onViewCreated() -> onViewStateRestored()
- Activity가 onDestroy()할 때 : Fragment는 onSaveInstanceState() -> onDestroyView() -> onDestroy() -> onDetach()
    <div>
    <img width="337" alt="스크린샷_2021-09-05_오후_4 01 06" src="https://user-images.githubusercontent.com/68374234/132122018-1c85c26e-e4b8-499b-af9b-70ecd0bf1f71.png"> 
    <img width="397" alt="스크린샷_2021-09-05_오후_3 32 32" src="https://user-images.githubusercontent.com/68374234/132120640-b2a1bf10-e198-4d42-9e93-9c58a7b8d31d.png">
    </div>

참고 : [Acitivity](https://developer.android.com/guide/components/activities/intro-activities?hl=ko), [Fragment](https://developer.android.com/guide/fragments?hl=ko), [https://milkye.tistory.com/60](https://milkye.tistory.com/60), [https://kumgo1d.tistory.com/76](https://kumgo1d.tistory.com/76) 
