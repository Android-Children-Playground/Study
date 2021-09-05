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

# Activity와 Fragment의 특징 요약

[제목 없음](https://www.notion.so/63a3e37f48c0495c9663ace60d62c973)