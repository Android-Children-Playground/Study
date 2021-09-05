# 안드로이드: Activity & Fragment

# Activity란?

- Activity 클래스는 안드로이드 앱의 중요 구성요소 중 하나이며, 사용자와 어플리케이션이 상호작용을 위한 진입점 역할을 담당한다.
- 안드로이드 어플리케이션은 반드시 하나이상의 Activity를 가지고 있어야한다.
- 두개의 Activity를 동시에 Display할 수 없다.
- 다른 어플리케이션의 Activity를 호출하는 것도 가능하다.
- 하나의 Activity내에 여러개의 Fragment를 추가하여 화면을 부할 시킬 수 있다.
- Activity는 화면에 표시되는 UI 구성을 위한 가장 기본이 되는요소이다. 안드로이드 앱은 화면에 UI를 표시하기 위해 최소 하나의 Activity를 가져야하며, 앱실행 시 지정된 Activity를 실행하여 사용자에게 UI를 표시한다.
- 안드로이드 앱이 실행되면 화면이 등장하고 UI가 화면 위에 나타나며, 버튼을 터치하거나 스크롤을 하는 등 앱을 사용하게 된다. 이와 같이 앱의 전반적인 활동을 담당하는 것이 액티비티 이며, 액티비티와 xml을 연결하여 UI를 표시하고 사용자가 여러가지 액션을 취할 수 있게 해준다.

# Activity의 UI 표시

- 화면의 UI와 같은 경우 Activity가 자체적인 그리기 기능을 통해 UI에 표시하는 것이 아닌. View 또는 View Group의 다양한 조합을 화면에 배치함으로써, UI를 표시하게 된다. 이때 onCreate() 합순 내에 setContentView(int)로 화면에 표시할 XML Layout 파일을 지정할수 있다.
- setContentView(int)를 통해 지정된 XML 파일내에 View를 findViewById()를 통해 View를 찾아 사용자와 어떤방식으로 상호작용할지 지정할수 있다.

```kotlin
class MainActivity : AppCompatActivity() {
    
    private lateinit var button1: TextView
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        button1 = findViewById(R.id.text1)
				
				button1.setOnClickListener{
            // 동작...
        }
    }
}
```

# Activity LifeCycle:

![Untitled](https://user-images.githubusercontent.com/51209390/132127358-8f58be2d-29ab-4e37-8ea6-7a9b13c772a6.png)

### onCreate():

- 어플리케이션 최초 실행 시 가장 처음으로 한 번 실행되는 함수
- 해당 Activity의 생명주기 중 딱 한번만 실행되며 viewModel연결과 인스턴스 생성 등 초기화 작업을 이 안에서 실행하면 된다
- onCreate() 메소드를 실행 완료하면 Activity는 시작됨 상태에 진입한다
- onCreate 함수에 항상 매개변수로 있는 savedInstanceState는 이전 Activity의 상태를 가지고 있는 Bundle객체이다
- ex) 세로에서 가로로 화면회전시, onDestroy()가 실행되기 직전 onSaveInstanceState()에서 Bundle 객체 저장 후 onCreate()를 실행하기 때문에 화면회전 후에도 데이터가 그대로 유지된다

### onStart():

- Activity가 시작됨 상태에 진입하면 Activity가 화면에 보이기 직전 빠르게 onStart()가 실행된다
- 이 단계에서 앱은 Activity를 foreground로 보내 사용자와 상호작용할 수 있도록 준비한다
- onStart() 메소드를 실행 완료하면 Activity가 재개됨 상태로 진입한다.

### onResuem():

- onStart()실행 후 바로 onResume()이 호출되며 이 상태에 들어가면 사용자가 앱과 상호작용을 할 수 있다
- 전화가 오거나 홈버튼을 누르거나 다른 Activity로 이동하는 등의 이벤트가 발생해 포커스를 잃기 전까지 이 상태에 머문다

### onRestart():

- Activity가 중단됨 상태에서 다시 호출되었을 때 실행된다
- onRestart() 뒤에는 항상 onStart()가 실행된다

### onPause():

- onResume()상태에서 이벤트가 발생해 포커스를 잃게되면 Activity는 일시정지 상태가 되며 onPause()를 호출한다
- Activity가 완전히 가려지지않고, 일부분이 보이거나 투명 Activity가 실행되었을 시에는 onPause()까지만 실행이 된다
- onPause()상태를 유지하다가 Activity가 다시 포커스를 얻게되면 onResume() 콜백을 실행한다
- 다른 Activity 실행 전에 실행되기 때문에 onPause()에서 너무 많은 작업을 하게되면 다음 Activity가 실행이 지연되게 된다

### onStop():

- Activity가 중단됨 상태에 진입하면 onPause()에 이어 onStop()까지 호출하게 된다
- 홈버튼을 누르거나 새로운 Activity가 실행되어 해당 Activity가 완전히 보이지 않게되는 경우에 해당한다
- 이 상태에서 Activity를 다시 시작하면 onRestart()를, Activity 실행을 종료하면 onDestroy()를 실행한다
- 예를들어, 홈버튼을 누른 상태에서 다시 앱을 실행하면 onRestart()에 이어 onStart()까지 실행되고, 뒤로가기를 눌러 앱을 종료하면 onPause() , onStop()에 이어 onDestroy()까지 호출한다

### onDestory():

- Activity에서 finish를 실행 해 Activity가 종료되어 스택에서 사라질 때 호출된다

# Fragment란?

- Fragment는 하나의 Activity가 여러개의 화면을 가질수 있도록 고안된 개념이다. 당양한 크기의 화면을 가진 모바일 환경이 늘어남에 따라 하나의 디스플레이 안에서 다양한 요소들을 보여주 기위한 해결책으로 사용되고 있다.((ex) Tablet PC)
- Fragment는 앱 UI의 재사용 가능한 부분을 나타내며, 자체 레이아웃을 정의 및 관리하고 자체 수명 주기를 보유하고 자체 입력 이베트를 처리할수있다.
- Fragment는 독립적으로 존재할 수 없고 Activity나 다른 Fragment에 호스팅 되어있어야 한다.
- 여러 개의 Fragment를 하나의 Activity 에 조합하여 창이 여러 개인 UI를 구축할수 있으며, 하나의 Fragment를 다른 Activity 에서 재사용 하는 것도 가능하다.
- Fragment의 수명주기는 호스트 Activity 의 수명 주기에 직접적으로 영향을 받는다. ex) 호스트 Activity가 일시 정지될경우, 그안의 모든 프래그먼트도 일시정지 처리된다.
- Activity 내에서 실행 중에 추가, 제거, 교체 하는 것이 가능하다.

# Fragment 사용 이유:

### Performance:

- Activity 초기화가 오래 걸리지는 않지만 Fragment와 비교했을때 상대적으로 무겁다.
- Activity 내에서 Fragment를 관리할수 있기 때문에 Activity비해 관리하기 편하다.
- Activity Back Stack에 Activity를 쌓아두기 보다 Fragment Back Stack에서 Fragment를 관리 하는게 메모리 관리에서 효율도 챙기고 화면 전환시에 Activity 보다 더 순조롭다.

### 데이터 공유:

- Activity 간 데이터 공유하는 가장 일반적인 방법은 Intent를 사용하는 방법이다.
- Activity는 다른 Process에서 실행하는 것을 염두하고 설계 되었기 때문에 memory 영역을 공유하지 않는다. 그렇기에 Linux Kernal Level에서 프로세스간 통신(IPC)를 해야 하므로 직접 메모리에 접근하는 것보다 많은 제약사항이 존제한다.
- Fragment간 데이터 공유는  Activity 내에서 자유롭게 이루어진다. (Activity에 비해 작업이 가볍다.)

### 재사용성의 증가:

- View or Business Logic을 Fragment 단위로 불리하는 것으로 아키텍쳐 원칙에서 가장 중요한 관심사 불리를 통해 의존성을 분리하고 독립성을 키울수 있다.

# Fragment LifeCycle:

![Untitled 1](https://user-images.githubusercontent.com/51209390/132127355-8f824cf2-2d74-4ceb-a6da-539929fc211b.png)

### onAttach():

- Fragment가 Activity에 붙을 때 호출
- 아직 Fragment가 완벽하게 생성된 상태는 아니며, 인자로 context가 주어진다.

### onCreate():

- Activity와 마찬가지로 초기화해야하는 리소스들을 여기서 초기화한다.
- Fragment를 생성하면서 넘겨준 값들이 있다면, 여기서 변수에 넣어주면된다.
- UI는 여기서 초기화 할수 없다.
- 본격적으로 Fragment 가 Activity에 호출을 받아 생성되는 시점이다.
- Activity의 onCreate()에선 View나 UI 관련 작업을 할 수있지만, Fragment의 onCreate()에서는 할수 없다.

### onCreateView():

- 레이아웃을 inflate 하는 지점이다.
- View 객체를 얻을 수 있으므로, 버튼이나 텍스트뷰 등을 초기화 할 수 있다.
- Fragment가 자신의 인터페이스를 처음 그리기 위해 호출한다.
- View를 반환해야 한다. 이 메서드는 Fragment의 레이아웃 루트이기 때문에 UI를 제공하지 않을 경우에는 null을 반환하면 된다.
- Fragment에 속한 각종 view나 viewGroup에 대한 UI 바인딩 작업을 할 수 있다.
- Fragment에서 UI를 그릴 때 호출되는 콜백이다.
- onCreateView의 매개변수로 전달되는 container가 Activity의 ViewGroup이며, 여기에 Fragment가 위치하게된다.
- 또 다른 매개변수인 savedInstanceState는  Bundle 객체로 Fragment가 재개되는 경우 이전 상태에 대한 데이터를 제공한다.

### onActivityCreated()

- Fragment에서 onCreateView를 마치고 Activity에서 onCreate가 호출되고 나서 호출되는 함수.
- Activity와 Fragment의 뷰가 모두 생성된 상태로, View를 변경하는 작업이 가능한 단계이다.
- Activity에서 Fragment를 모두 생성하고 난 다음에 호출된다.
- Activity와 Fragment가 드디어 연결되는 시점이다.
- Activity와 Fragment의 뷰가 모두 생성되고, 연결된 상태이다.

### onResuem():

- fragment가 비로소 화면에 보여지는 단계이다.
- 사용자에게 포커스를 잡은 상태로 사용자와의 상호작용이 가능하다.
- Activity와 마찬가지로 이벤트가 발생하여 Fragment가 가려지기 전까지 이 상태가 유지된다.

### onPause():

- Fragment는 사용자와의 상호작용을 중지한다.
- 부모 Activity가 아닌 다른 Activity가 위로 올라오거나, 다른 Fragment가 add되는 경우 일시정지 상태로 들어간다.
- UI관련 처리를 정지하고, 중요한 데이터를 저장한다

### onStop():

- Fragment는 더이상 보여지지 않게되며, Fragment 기능이 중지된다.
- Fragment가 완전히 가려지는 경우, onPause()에 이어 onStop()까지 실행된다.
- 시스템에서 onStateInstance()를 호출하여 UI의 상태를 저장하므로 Activity를 다시 띄우면 이전 상태가 그대로 보여진다.

### onDestoryView():

- Fragment와 관련된 view가 제거될 때 실행된다.
- Activity에서 Fragment 생성 시 addToBackStack()를 요청했을 경우 onDestroy()를 호출하지 않고 인스턴스가 저장되어 있다가 Fragment를 다시 부를 때 onCreateView()를 실행하여 다시 화면에 보여지게

### onDestory():

- view가 제거된 후 Fragment가 완전히 소멸되기 전에 호출된다.

### onDetach():

- Fragment가 완전히 소멸되고, Activity와의 연결도 끊어질 때 실행된다.

### Activity LifeCycle과 Fragment LifeCycle의 관계

![Untitled 2](https://user-images.githubusercontent.com/51209390/132127357-96f4091c-1b52-4cbe-a21a-90c07502e825.png)
