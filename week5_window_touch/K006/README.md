# Window는 무엇일까?

### **Window**

공식 문서에 따르면 Window는 최상위 원도우 `모양 및 동작 정책`에 대한 추상 클래스라고 표현하고 있다. 그리고 window 는 `view hierarchy` 를 가지고 있는 사각형 영역이라고 정의하는 곳도 있다. 이를 조합한다면 우리가 보는 UI 화면의 최상위 단에 대한 정의이자, 이는 view hierarchy 정보를 가지고 있다고 볼 수 있겠다.

<img src="https://user-images.githubusercontent.com/71161576/135750443-dbe45276-27fc-4ffb-9e32-d5ad13f25711.png" width="200">

출처 : [https://stackoverflow.com/questions/9451755/what-is-an-android-window](https://stackoverflow.com/questions/9451755/what-is-an-android-window)

이렇게 하나의 화면에도 다수의 window가 존재하게 된다. 그리고 이들은 `WindowManager`에 의해서 관리 된다.

<img src="https://user-images.githubusercontent.com/71161576/135750515-dc168d1f-a6ff-412a-a37b-530e1cda3c97.png" width="600">

출처 : [https://stackoverflow.com/questions/9451755/what-is-an-android-window](https://stackoverflow.com/questions/9451755/what-is-an-android-window)

위의 이미지에서 알 수 있듯이, StatusBar, NavigationBar, Dialog 등이 별도의 window로 표현되고 있는 것을 알 수 있다.

그리고 추가적으로 공식 문서에 따르면 모든 액티비티는 하나의 window를 가지고 있다고 나와있다.

즉, 우리가 액티비티를 통해서 레이아웃을 inflate하고 사용자 UI 를 구성하고, 동작하는 모든 것들이 window를 기반하여 동작한다고 볼 수 있을 것 같다.

그리고 Window와 더불어 `Surface` 와 `View`의 개념이 함께 사용이 된다.

<img src="https://user-images.githubusercontent.com/71161576/135750548-48c24b81-1231-445e-85d2-afbe5e94ef73.png" width="600">

출처 : [https://stackoverflow.com/questions/9451755/what-is-an-android-window](https://stackoverflow.com/questions/9451755/what-is-an-android-window)

### **Surface**

surface는 화면을 구성할 pixel들을 가지고 있는 객체이다. 모든 window 는 각자의 surface를 가지고 있다. 보통 surface는 하나 이상의 buffer를 가진다.(보통은 두개) 두개의 buffer를 가지고 있는 경우 SurfaceFliger가 현재화면을 조합해서 만드는 동안 다음 화면에 그려질 화면 모양을 예비 buffer에 보관함으로써 다음 화면 출력을 대비하게 된다. - 출처 : [https://stackoverflow.com/questions/9451755/what-is-an-android-window](https://stackoverflow.com/questions/9451755/what-is-an-android-window)

### **View**

view는 window 안에 있는 상호작용 가능한 UI 구성요소다. window는 하나의 `view hierarchy` 를 가진다. 언제든 window가 다시 그려져야 한다면 (예를 들어 어떤 view가 invalidate된 경우) surface는 locked 되고 canvas를 return한다. hierarchy를 따라 canvas는 각각의 view를 거치게 된다. view들은 각자 순서에 각자 필요한 부분을 canvas에 그리게 된다. (정확히 말하면 canvas를 이용해서 surface에 연결된 bitmap에 그린다.) 이 과정을 마치면 surface를 unlocked 그리고 posted 된다. 이렇게 완성된 buffer는 foreground와 swap되고 SurfaceFlinger에 의해 조합되서 출력된다.

출처 : [https://stackoverflow.com/questions/9451755/what-is-an-android-window](https://stackoverflow.com/questions/9451755/what-is-an-android-window)

 

# window에서 view까지 터치 과정

사용자의 입력을 받고, 입력에 따른 동작을 수행하는 것은 UI 의 가장 기본적인 기능이라고 할 수 있다. 

흔히 `setOnClickListener()` 같은 이벤트 리스너를 등록해서 사용하는 방식으로 우리는 사용자의 입력에 대한 동작을 정의할 수 있었다. 그렇다면 어떻게 화면에서 사용자의 입력이 전달되고, 이것이 우리가 등록한 동작으로 연결되는 것일까?

우선 사용자의 입력에 따라서 안드로이드는 적절한 이벤트를 발생시킨다. 그리고 이 이벤트가 결국 view까지 전달이 되어야 우리가 리스너를 통해서 등록한 콜백이 동작할 것이다.

액티비티를 예로 들어보겠다. 액티비티는 end point로, 사용자의 입력이 가장 먼저 들어오는 곳이 된다. 그러고 나면 우리가 액티비티에 inflate 해주었던 layout, 그리고 그 내부에 있는 view group 또는 view 를 찾아 들어갈 것이라고 유추해 볼 수 있다. ( Top - Down )

이처럼 이벤트를 전달 하는 부분에 해당되는 메서드가 `dispatchTouchEvent()` 이다. 

이를 예제를 통해서 한번 확인해보자.

이벤트가 어떻게 전달이 되는지 확인해보기 위해서 

Activity → CustomLayout(ViewGroup) → Custom(View) 의 형태로 레이아웃을 만들어 보았다.

```kotlin
class CustomLayout(context: Context) : ConstraintLayout(context){

    private val TAG = "CustomLayout"

    override fun dispatchTouchEvent(ev: MotionEvent?): Boolean {
        Log.i(TAG, "Layout(ViewGroup) : dispatchTouchEvent")
        return super.dispatchTouchEvent(ev)
    }
}
```

```kotlin
class CustomView(context: Context) : AppCompatTextView(context) {

    private val TAG = "CustomView"

    init {
        this.text= "This is a CustomView!"
    }

    override fun dispatchTouchEvent(event: MotionEvent?): Boolean {
        Log.i(TAG, "View : dispatchTouchEvent")
        return super.dispatchTouchEvent(event)
    }
}
```

```kotlin
class MainActivity : AppCompatActivity() {
    private val TAG = "MainActivity"
    private val binding: ActivityMainBinding by lazy {
        ActivityMainBinding.inflate(layoutInflater)
    }

    override fun onCreate( savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(binding.root)

        val customLayout = CustomLayout(this)
        addContentView(customLayout, ConstraintLayout.LayoutParams(MATCH_PARENT, MATCH_PARENT))

        val customView = CustomView(this)
        customLayout.addView(customView)
    }

    override fun dispatchTouchEvent(ev: MotionEvent?): Boolean {
        Log.i(TAG, "Activity(Window) : dispatchTouchEvent")
        return super.dispatchTouchEvent(ev)
    }
}
```

보면 모두 dispatchTouchEvent() 메서드가 존재하는 것을 확인할 수 있고, 이는 각각 Activity, ViewGroup, View 에 정의 되어있는 메서드이다. 

커스텀 뷰를 클릭했을 때의 실행결과는 다음과 같다.
<img src="https://user-images.githubusercontent.com/71161576/135750613-b3dbd3ba-cfe4-4974-880e-76debca557a9.png" width="600">

먼저 안드로이드가 터치를 감지하여 터치 이벤트를 발생 시켰을 것이고, end point인 액티비티에서부터 순서대로 dispatchTouchEvent를 호출하면서 이벤트를 하위 요소들에 전달하고 있는 것을 확인할 수 있다.

( 마지막 줄의 액티비티 dispatchTouchEvent는 손을 떼는 동작에 발생하는 것을 확인했는데 이는 하위로 전파 되지 않았다.)

실행 순서를 고려해보았을 때, 계층 형식으로 동작하고 있다는 것을 알 수 있었다. 해당 영역을 터치한 경우 당연히 액티비티 영역에 해당하기 때문에 액티비티의 메서드는 수행이된다. 그리고 하위로 내려가면서 해당 영역에 구성요소가 존재한다면 그 구성요소에도 이벤트를 전달하는 방식으로 수행하는 모습이다.

<img src="https://user-images.githubusercontent.com/71161576/135750638-61d1f7ed-ffd3-4a20-a1e1-529b05204a41.png" width="600">

그렇기 때문에 커스텀 뷰가 존재하지 않는 곳을 터치한 경우에는 레이아웃의 dispatchTouchEvent() 까지만 호출된 것을 확인할 수 있다.

그리고 추가적으로 가장 상위라고 할 수 있는 액티비티의 dispatchTouchEvent()는 어디서 오는 것인지 조금 더 살펴보았다.

가장 상위의 android.app.Activity 클래스에 dispatchTouchEvent() 메서드가 존재하는데 이 또한 오버라이딩 된 메소드였다. 이를 한번 따라가봤더니 이는 `Window.Callback` 인터페이스 내부에 존재하는 핸들러였다. 

```java
public abstract class Window {
		...
		public interface Callback {
				...
				boolean dispatchTouchEvent(MotionEvent var1);
				...
		}
}
```

위에서 살펴봤던 것처럼 액티비티는 하나의 window를 가지고 있는데, 이는 view hierarchy 정보도 가지고 있다. 이를 바탕으로 window는 사용자의 입력을 받아서 발생한 이벤트를 하위 구성요소로 넘겨주는 기능도 수행한다는 것을 알 수 있다.

이렇게 가장 하위에 존재하는 View 까지 탐색하여 이벤트를 전달되어 핸들러가 동작하게 된다. 그렇다면 이 핸들러가 동작하는 순서는 어떻게 되는 것일까?

그래서 각 클래스에 `onTouchEvent()` 핸들러로 결과를 출력해보았다.

<img src="https://user-images.githubusercontent.com/71161576/135750667-b4f296e3-e6f5-4a88-9f5c-5f0a547309f0.png" width="600">

이벤트를 전달하는 것과는 반대의 순서대로 동작이 수행되는 것을 확인할 수 있었다. 

그리고 추가적으로 ViewGroup 에는 `onInterceptTouchEvent()` 라는 메서드가 존재하는데 이는  true를 리턴할 때 현재 ViewGroup 하위의 View로는 이벤트를 전달하지 않는다.

커스텀레이아웃 클래스에 해당 메서드를 추가하고 결과를 확인해보았다.
<img src="https://user-images.githubusercontent.com/71161576/135750680-84335259-73ba-4c3a-9bbd-d2241874fe76.png" width="600">

이 때는 커스텀 레이아웃의 하위에 존재하는 커스텀 뷰에는 이벤트가 전달되지 않은 것을 확인할 수 있다.

### **Dialog**

흔히 사용하던 **Dialog**에도 window 가 사용이 되는지 확인해보았다.

Dialog 클래스를 보면 마찬가지로 `Window.Callback` 을 구현하고 있었다. 그렇기 때문에 액티비티와 동일한 방식으로 이벤트를 처리하고 별도로 동작이 가능한 구성요소이다.

```kotlin
val dialog = AlertDialog.Builder(this).create() // this: Context
		dialog.window
		dialog.dispatchTouchEvent()
		dialog.onTouchEvent()
```

## 다양한 이벤트 처리하기

### 제스처 이벤트

제스처 이벤트는 일반적인 터치가 아니라 스크롤 등의 방향을 알아내어 이에 따라 발생하는 이벤트를 말한다. 그러면 제스처 이벤트는 어떻게 처리하는 것 일까!

`GestureDetector` 로 제스쳐 이벤트를 처리해줄 수 있고, 이 객체에 터치 이벤트를 전달하면 상황에 맞는 메서드를 호출한다.

GestureDetector 객체를 생성하기 위해서는 `OnGestureListener` 를 구현해서 생성자 매개변수로 넘겨줘야한다.

```kotlin
val detector = GestureDetector(this, object: GestureDetector.OnGestureListener {
            override fun onDown(p0: MotionEvent?): Boolean {
                Log.i("", "onDown")
                return true
            }

            override fun onShowPress(p0: MotionEvent?) {
                Log.i("", "onShowPress")
            }

            override fun onSingleTapUp(p0: MotionEvent?): Boolean {
                Log.i("", "onSingleTapUp")
                return true
            }

            override fun onScroll(p0: MotionEvent?, p1: MotionEvent?, p2: Float, p3: Float): Boolean {
                Log.i("", "onScroll")
                return true
            }

            override fun onLongPress(p0: MotionEvent?) {
                Log.i("", "onLongPress")
            }

            override fun onFling(p0: MotionEvent?, p1: MotionEvent?, p2: Float, p3: Float): Boolean {
                Log.i("", "onFling")
                return true
            }
        })
```

```kotlin
customLayout.setOnTouchListener { view, motionEvent ->
		detector.onTouchEvent(motionEvent)
		true
}
```

터치 이벤트가 발생했을 때, 생성한 detector 객체를 사용하면 제스처 관련 이벤트를 추가적으로 감지할 수 있다. 

각 핸들러 메서드가 동작하는 이벤트는 다음과 같다.

- **`onDown()`** : 화면을 눌렀을 경우
- **`onShowPress()`** :화면이 눌렸다 떼어지는 경우
- **`onSingleTapUp()`** : 화면이 한손가락으로 눌렀다 떼어지는 경우
- **`onScroll()`** : 화면이 눌린채 일정한 속도와 방향으로 움직였다 떼는 경우
- **`onFling()`** : 화면이 눌릴 채 가속도를 붙여 손가락을 움직였다 떼느 경우
- **`onLongPress()`** : 화면을 손가락으로 오래 누르는 경우

onFilng() 같은 경우는 사용해보니까, 튕기기..? 이런 용도로 쓸 수도 있을 것 같다.

공식문서에는 제스처(움직임 감지) 와 관련하여 다음과 같은 기능도 사용할 수 있다고 나와있다. 

- 포인터의 시작 위치와 종료 위치(Ex. 화면에서 A 지점에서 B 지점으로 개체 이동)
- 포인터 이동 방향. x 좌표와 y 좌표에 의해 결정.
- 히스토리 기능. `MotionEvent` 메서드 `getHistorySize()`를 호출하여 동작의 크기를 찾을 수 있음. 그런 다음 모션 이벤트의 `getHistorical*<Value>*` 메서드를 사용하여 각 히스토리 이벤트의 `위치`, `크기`, `시간`, `압력`을 구할 수 있다. 히스토리는 터치 그리기와 같은 용도로 사용자 손가락의 자취를 렌더링할 때 유용하다.
- 포인터가 터치스크린에서 움직일 때의 `속도`  → `VelocityTracker` 클래스

출처 : [https://developer.android.com/training/gestures/movement?hl=ko](https://developer.android.com/training/gestures/movement?hl=ko)

부가적인 기능을 더 활용하면 더 효과적으로 제스처를 감지할 수 있을 것이다.

### 멀티 터치 이벤트

멀티터치 동작은 포인터(손가락) 여러 개가 동시에 화면을 터치하는 것을 말하는 것이다. 멀티 터치 이벤트에서 사용 되는 주요 이벤트들은 다음과 같다.

- `ACTION_DOWN`: 화면을 터치하는 첫 번째 포인터에 해당된다. 이 포인터의 포인터 데이터는 `MotionEvent`에서 항상 0번째 인덱스에 해당된다.
- `ACTION_POINTER_DOWN`: 첫 번째 포인터 다음에 들어오는 추가 포인터에 해당된다. 이 포인터의 인덱스 데이터는 `getActionIndex(event: MotionEvent)`가 반환하는 값이다.
- `ACTION_MOVE`: 누르기 동작 중에 변경사항이 발생하면 생성된다. 드래그 동작.
- `ACTION_POINTER_UP`: 기본이 아닌 포인터가 위로 올라갈 때(떼어질 때) 의 이벤트.
- `ACTION_UP`: 마지막 포인터가 화면 밖으로 나갈 때(떼어질 때) 전송된다.

멀티 터치 이벤트에서 다수의 포인터는 인덱스와 ID를 통해 MotionEvent 내에서 구분된다.

그리고 멀티 터치 이벤트의 동작을 처리하기 위해서는 반드시 `getActionMasked()` 메서드(또는 호환성 버전인 `MotionEventCompat.getActionMasked()`이면 더 좋음) 를 사용할 것을 권장한다고 공식문서에 나와있다.

```kotlin
val (xPos: Int, yPos: Int) = MotionEventCompat.getActionMasked(event).let { action ->
        Log.d(DEBUG_TAG, "The action is ${actionToString(action)}")
        MotionEventCompat.getActionIndex(event).let { index ->
            MotionEventCompat.getX(event, index).toInt() to MotionEventCompat.getY(event, index).toInt()
        }
    }

    if (event.pointerCount > 1) {
        Log.d(DEBUG_TAG, "Multitouch event")

    } else {
        Log.d(DEBUG_TAG, "Single touch event")
    }

    ...

    fun actionToString(action: Int): String {
        return when (action) {
            MotionEvent.ACTION_DOWN -> "Down"
            MotionEvent.ACTION_MOVE -> "Move"
            MotionEvent.ACTION_POINTER_DOWN -> "Pointer Down"
            MotionEvent.ACTION_UP -> "Up"
            MotionEvent.ACTION_POINTER_UP -> "Pointer Up"
            MotionEvent.ACTION_OUTSIDE -> "Outside"
            MotionEvent.ACTION_CANCEL -> "Cancel"
            else -> ""
        }
    }
```

### Nested Scroll