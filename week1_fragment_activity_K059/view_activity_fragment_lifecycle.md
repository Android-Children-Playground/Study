# 주제 

1. Activity, Fragment 차이 -> 못했습니다,, 시간이 부족하네요 

2. Lifecycle 차이점  

   Activity Lifecycle
   Fragment Lifecycle
   View Lifecycle

   각각의 라이프사이클이 어떻게 동작하는지. 단순히 생성되고 종료될 때의 라이프사이클 뿐만 아니라 액티비티, 프래그먼트 전환 등 여러 상황에서의 라이프사이클 흐름.

## Lifecycle

`Lifecycle`은 액티비티나 프래그먼트와 같은 구성요소의 수명 주기 상태 관련 정보를 포함하며 다른 객체가 이 상태를 관찰할 수 있게 하는 클래스입니다.

`Lifecycle`은 두 가지 enum을 사용하여 연결된 구성요소의 수명 주기 상태를 추적합니다.

1. 이벤트 프레임워크 및 `Lifecycle` 클래스에서 전달되는 수명 주기 이벤트입니다. 이러한 이벤트는 액티비티와 프래그먼트의 콜백 이벤트에 매핑됩니다.
2. 상태 `Lifecycle` 객체가 추적한 구성요소의 현재 상태입니다.

<img width="781" alt="Screen Shot 2021-09-05 at 5 04 09 PM" src="https://user-images.githubusercontent.com/55446114/132119975-f73fe451-7550-4bb7-a7ce-d7ca150e972f.png">

```java
public abstract class Lifecycle {
		
		...

		@MainThread
    public abstract void addObserver(@NonNull LifecycleObserver observer);

		@MainThread
    public abstract void removeObserver(@NonNull LifecycleObserver observer);
		
		@MainThread
    @NonNull
    public abstract State getCurrentState();
		
		public enum Event {
        
        ON_CREATE,
        ON_START,
        ON_RESUME,
        ON_PAUSE,
        ON_STOP,
        ON_DESTROY,
        /**
         * An constant that can be used to match all events.
         */
        ON_ANY;
        
         ....
		}
		
		...

		public enum State {
        
        DESTROYED,
        INITIALIZED,
        CREATED,
        STARTED,
        RESUMED;

        public boolean isAtLeast(@NonNull State state) {
            return compareTo(state) >= 0;
        }
    }
```

## Lifecycle Observer

```java
class MyObserver : LifecycleObserver {

    @OnLifecycleEvent(Lifecycle.Event.ON_RESUME)
    fun connectListener() {
        ...
    }

    @OnLifecycleEvent(Lifecycle.Event.ON_PAUSE)
    fun disconnectListener() {
        ...
    }
}

myLifecycleOwner.getLifecycle().addObserver(MyObserver())
```

`LifecycleObserver` 메서드에 `annotation`을 추가하여 구성요소의 수명 주기 상태를 모니터링할 수 있습니다. 그런 후, 다음 예와 같이 Lifecycle 클래스의 `addObserver()` 메서드를 호출하고 관찰자의 인스턴스를 전달하여 관찰자를 추가할 수 있습니다.

실제로 `LifecycleObserver` 코드를 확인하면 비어있다.

```java
/**
 * Marks a class as a LifecycleObserver. It does not have any methods, instead, relies on
 * {@link OnLifecycleEvent} annotated methods.
 * <p>
 * @see Lifecycle Lifecycle - for samples and usage patterns.
 */
@SuppressWarnings("WeakerAccess")
public interface LifecycleObserver {

}
```

OnLifecycleEvent 라는 annotated methods 에 의존한다고 나와있다.

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface OnLifecycleEvent {
    Lifecycle.Event value();
}
```

제대로 이해하려면 annotation에 대한 이해가 필요해 보인다.

## LifecycleOwner

`LifecycleOwner`는 클래스에 `Lifecycle`이 있음을 나타내는 단일 메서드 인터페이스입니다.

```java
/**
 * A class that has an Android lifecycle. These events can be used by custom components to
 * handle lifecycle changes without implementing any code inside the Activity or the Fragment.
 *
 * @see Lifecycle
 * @see ViewTreeLifecycleOwner
 */
@SuppressWarnings({"WeakerAccess", "unused"})
public interface LifecycleOwner {
    /**
     * Returns the Lifecycle of the provider.
     *
     * @return The lifecycle of the provider.
     */
    @NonNull
    Lifecycle getLifecycle();
}
```

이 인터페이스에는 클래스에서 구현해야 하는 `getLifecycle()` 메서드가 하나 있습니다. 대신 전체 애플리케이션 프로세스의 수명 주기를 관리하려는 경우 `ProcessLifecycleOwner`를 참고하세요.

이 인터페이스는 `Fragment` 및 `AppCompatActivity`와 같은 개별 클래스에서 `Lifecycle`의 소유권을 추출하고, 함께 작동하는 구성요소를 작성할 수 있게 합니다. 모든 custom application class는 `LifecycleOwner` 인터페이스를 구현할 수 있습니다.

관찰자가 관찰을 위해 등록할 수 있는 수명 주기를 소유자가 제공할 수 있으므로, `LifecycleObserver`를 구현하는 구성요소는 `LifecycleOwner`를 구현하는 구성요소와 원활하게 작동합니다.





## View Lifecycle

view는 포커스를 얻으면 레이아웃을 그리도록 요청합니다.  
이때 레이아웃의 계층 구조중에 **rootView**를 제공해야합니다.  
따라서 그리기는 루트 노드에서 시작되어 `전위 순회`방식으로 그려집니다.  
부모뷰는 자식뷰가 그려지기전에 그려지고, 형제뷰는 전위방식에 따라 순서대로 그려지게 됩니다.  
 레이아웃을 그리는 과정은 측정(measure)단계와 레이아웃(layout)단계를 통해 그려지게 됩니다.

<img width="250" alt="Screen_Shot_2021-08-11_at_1 08 59_AM" src="https://user-images.githubusercontent.com/55446114/132091944-8dbdfd63-cfcc-440c-8940-215a5f0ba562.png">

https://beomseok95.tistory.com/249. 
https://woovictory.github.io/2019/01/06/Android-View-Functions/. 
https://hyeonu1258.github.io/2018/03/26/%EC%95%88%EB%93%9C%EB%A1%9C%EC%9D%B4%EB%93%9C%20%EB%A9%B4%EC%A0%91/

### measure(int widthMeasureSpec, int heightMeasureSpec)

부모노드에서 자식노드를 경유하며 실행되며, 뷰의 크기를 알아내기 위해 호출됩니다. 이것은 뷰의 크기를 측정하는 것은 아니며 실제 크기 측정은 `onMeasure(int, int)`를 통해 이뤄집니다.. `measure(int, int)`의 내부에서는 `onMeasure(int, int)`를 호출함으로써 뷰의 크기를 알아냅니다.  

### layout(int l, int t, int r, int b)

부모노드에서 자식노드를 경유하며 실행되며, 뷰와 자식뷰들의 크기와 위치를 할당할 때 사용된다. `measure(int, int)`에 의해 각 뷰에 저장된 크기를 사용하여 위치를 지정한다. 내부적으로 `onLayout()`를 호출하고 `onLayout()`에서 실제 뷰의 위치를 할당하는 구조로 되어있다.  

ViewGroup 은 자신이 가지고 있는 child View 들이 어떤 식으로 배치(layout) 되는 지를 결정해야 한다. 이런 결정은 결국 onLayout() 에서 child View 들의 구역(region) 을 정해주는 것으로 가능합니다.  
그러므로 현재 View 의 onLayout() 에서는 child 에게 그려지는 범위까지만 알려주면 됩니다. 그러면 child 는 그 범위를 자신의 범위로 인식하고 그 범위에 따른 상대적인 좌표를 계산하게 됩니다.



### onMeasure(int, int)

너비와 높이를 결정하기 위해 뷰와 그 content를 측정합니다. 이 메서드는 contents의 정확하고 효율적인 측정을 제공하기 위해 하위 클래스에 의해 override되어야 합니다. 

이 메서드를 override할 때 ,  setMeasuredDimension() 을 호출하여 이 뷰의 측정된 너비와 높이를 저장해야 합니다. 그렇지 않으면 measure()에 의해 IllegalStateException이 throw 됩니다. super class의 onMeasure()을 호출하는 것은 valid한 사용입니다.

onMeasure() 에서 할 일은 onMeasure() 를 가지고 있는 class 의 width 와 height 를 설정 해 주는 일을 한다고 보면 된다. View 를 extends 했다면 view 의 width 와 height 를 설정해 주는 작업을 onMeasure() 에 작성하면 되고, layout 을 extends 했다면, layout 의 width와 height 를 작성하는 코드를 만들면 된다.

`Parameters`

- `widthMeasureSpec`	 int: horizontal space requirements as imposed by the parent. The requirements are encoded with View.MeasureSpec.

- `heightMeasureSpec`   int: vertical space requirements as imposed by the parent. The requirements are encoded with View.MeasureSpec.

인수로 전달 된 Spec은 부모 레이아웃이 차일드에게 제공하는 여유 공간의 폭과 높이에 대한 정보이며 공간의 성질을 지정하는 모드(Spec의 상위 2비트)와 공간의 크기값(나머지 하위 30비트)이 저장되어 하나의 정수로 묶여 전달됨. 

Spec의 두 값을 추출하거나 다시 합칠때는 View.MeasureSpec 클래스의 아래 메서드를 사용함.

- int getMode(int measureSpec)

- int getSize(INT measureSpec)

- int makeMeasureSpec(int size, int mode)

<img width="615" alt="Screen Shot 2021-09-04 at 8 05 34 PM" src="https://user-images.githubusercontent.com/55446114/132092300-3afdf9f2-1339-4f63-8453-96031ceee55e.png">

onMeasure에서 부모 레이아웃으로 차일드가 원하는 크기를 다음 메소드로 리턴? 함.

```kotlin
void setMeasuredDimesion(int measureWidth, int measureHeight)
```

두 인수는 차일드가 원하는 폭과 높이.

https://m.blog.naver.com/PostView.naver?isHttpsRedirect=true&blogId=zoomen1004&logNo=220227606331

### View 측정과정 확인해보자 

```xml
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">
    <RelativeLayout
        android:layout_width="100dp"
        android:layout_height="100dp"
        app:layout_constraintLeft_toLeftOf="parent"
        app:layout_constraintTop_toTopOf="parent">
        <com.example.viewtest.CustomView
            android:layout_width="100dp"
            android:layout_height="100dp"
            android:background="@color/black"
            app:layout_constraintTop_toTopOf="parent"
            app:layout_constraintLeft_toLeftOf="parent"
            />
    </RelativeLayout>

</RelativeLayout>
```

`CustomView` 의 부모뷰의 width와 height 는 각각 100dp이다.

CustomView를 100dp, 100dp 로 맞추고 onMeasure(), onLayout(),onDraw()에서 진짜 100dp로 측정되는지 확인해보자

```kotlin
import android.content.Context
import android.graphics.Canvas
import android.util.AttributeSet
import android.util.Log
import android.view.View


class CustomView : View {
    constructor(context: Context?) : super(context) {
    }
    constructor(context: Context?, attrs: AttributeSet?) : super(context, attrs) {
    }
    constructor(context: Context?, attrs: AttributeSet?, defStyleAttr: Int) : super(
        context,
        attrs,
        defStyleAttr
    ) {
    }

    constructor(
        context: Context?,
        attrs: AttributeSet?,
        defStyleAttr: Int,
        defStyleRes: Int
    ) : super(context, attrs, defStyleAttr, defStyleRes) {
    }

    override fun onMeasure(widthMeasureSpec: Int, heightMeasureSpec: Int) {
        super.onMeasure(widthMeasureSpec, heightMeasureSpec)
        val metrics = resources.displayMetrics
        val density = metrics.density
        Log.v("CustomView-density", "" + density)

        //모드를 출력해 보자.
        val widthMode = MeasureSpec.getMode(widthMeasureSpec)
        val heightMode = MeasureSpec.getMode(heightMeasureSpec)
        printMode("width mode : ", widthMode)
        printMode("height mode : ", heightMode)

        //측정된 폭과 높이를 출력해 보자
        val width = MeasureSpec.getSize(widthMeasureSpec)/density
        val height = MeasureSpec.getSize(heightMeasureSpec)/density
        Log.v("CustomView-onMeasure", "width : $width height : $height")
		    //dp 단위로 출력 
		}

    override fun onLayout(changed: Boolean, left: Int, top: Int, right: Int, bottom: Int) {
        Log.v(
            "CustomView-onLayout",
            "rect : (x, y, w, h) : " + left + " " + top + " " + (right - left) + " " + (bottom - top)
        )//pixel 단위로 출력 
    }

    override fun onDraw(canvas: Canvas?) {
        val left = left
        val top = top
        val width = width
        val height = height
        val mwidth = measuredWidth
        val mheight = measuredHeight
        Log.v(
            "CustomView-onDraw",
            "rect : (x, y, w, h, mw, mh) : $left $top $width $height $mwidth $mheight"
        )//pixel 단위로 출력 
    }


    private fun printMode(tag: String, mode: Int) {
        when (mode) {
            MeasureSpec.AT_MOST -> Log.v("CustomView-MeasureSpec", "$tag AT_MOST")
            MeasureSpec.EXACTLY -> Log.v("CustomView-MeasureSpec", "$tag EXACTLY")
            MeasureSpec.UNSPECIFIED -> Log.v("CustomView-MeasureSpec", "$tag UNSPECIFIED")
        }
    }
}
```

<img width="967" alt="Screen Shot 2021-09-04 at 10 10 04 PM" src="https://user-images.githubusercontent.com/55446114/132095507-7910c884-944e-4f87-81fd-5f62775cd4c4.png">

8번의 onMeasure() 호출이 일어났다. 8번의 측정과정을 거쳐 pixel : 263(dp로 100)  으로 우리가 원하는 뷰의 크기대로 측정이 된다

이번에는 부모 뷰의 크기를 자식뷰 보다 작게 하고 측정해보자

```xml
...
    <RelativeLayout
        android:layout_width="10dp"
        android:layout_height="10dp"
        app:layout_constraintLeft_toLeftOf="parent"
        app:layout_constraintTop_toTopOf="parent">
        <com.example.viewtest.CustomView
            android:layout_width="100dp"
            android:layout_height="100dp"
            android:background="@color/black"
            app:layout_constraintTop_toTopOf="parent"
            app:layout_constraintLeft_toLeftOf="parent"
            />
    </RelativeLayout>
...
```

부모 뷰의 크기를 각각 10dp 10dp로 설정했다.

<img width="967" alt="Screen Shot 2021-09-04 at 10 11 32 PM" src="https://user-images.githubusercontent.com/55446114/132095558-ac04cb14-1918-4056-9261-691a57481354.png">

자식 뷰의 크기가 width : 100dp, height: 100dp가 아닌 10dp로 측정되었다. 부모 뷰의 크기보다 더 커질 수 없기 때문이다. 

`super.onMeasure( )` 의 코드를 확인해보자

```java
protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
    setMeasuredDimension(getDefaultSize(getSuggestedMinimumWidth(), widthMeasureSpec),
            getDefaultSize(getSuggestedMinimumHeight(), heightMeasureSpec));
}
```

`setMeasuredDimension()` 을 호출한다. 이는 측정된 크기를 저장하는 메서드이다.

`getSuggestedMinimumWidth()`의 코드를 확인해보자

```java
protected int getSuggestedMinimumWidth() {
    return (mBackground == null) ? mMinWidth : max(mMinWidth, mBackground.getMinimumWidth());
}
```

`getMinimumWidth()`는 다음과 같다.

```java
/**
 * Returns the minimum width suggested by this Drawable. If a View uses this
 * Drawable as a background, it is suggested that the View use at least this
 * value for its width. (There will be some scenarios where this will not be
 * possible.) This value should INCLUDE any padding.
 *
 * @return The minimum width suggested by this Drawable. If this Drawable
 *         doesn't have a suggested minimum width, 0 is returned.
 */
public int getMinimumWidth() {
    final int intrinsicWidth = getIntrinsicWidth();
    return intrinsicWidth > 0 ? intrinsicWidth : 0;
}
```

 Drawable이 Solid컬러가 아닌 비트맵과 같은 크기가 있는 Drawable이면 컨텐츠의 크기를 반환한다.그렇지 않으면 -1을 반환.  따라서 getSuggestedMinimumWidth() 코드를 읽어보면 배경 Drawable이 없으면 뷰의 최소크기를 반환하고 배경Drawable이 있으면 뷰의 최소 크기와 배경Drawable의 최소크기 중 큰 값을 반환하는 것을 알 수 있다.

이제 `getDefaultSize()`를 살펴보자 

```java
public static int getDefaultSize(int size, int measureSpec) {
  int result = size;
  int specMode = MeasureSpec.getMode(measureSpec);
  int specSize = MeasureSpec.getSize(measureSpec);

  switch (specMode) {
    case MeasureSpec.UNSPECIFIED:
      result = size;
      break;
    case MeasureSpec.AT_MOST:
    case MeasureSpec.EXACTLY:
      result = specSize;
      break;
  }
  return result;
}
```

위 코드를 보면 두 가지 size가 나오는데 하나는 내가 원하는 size이고 다른 하나는 측정된 size.

- result : 내가 원하는 크기
- specSize : 측정된 크기

3가지 종류의 측정모드가 있다.

- `MeasureSpec.UNSPECIFIED`
- `MeasureSpec.AT_MOST`
- `MeasureSpec.EXACTLY`

각각이 풍기는 뉘앙스는 아래와 같다 

- MeasureSpec.UNSPECIFIED
  - 아직 정확하게 측정된 값은 없어. 아무래도 다시 측정과정을 진행해야겠어. 그러니 네가 원하는 값을 맘대로 적든 말든 상관 없어~
- MeasureSpec.AT_MOST
  - 네가 설정할 수 있는 최대 측정값을 알려줄게. 이 안에서 값을 설정해~. 아 그리고 다시 측정과정이 발생할 수 있어. 최대치도 변할 수 있고~
- MeasureSpec.EXACTLY
  - 정확한 측정값이야. 특별한 사유가 없는 한 이 크기로 해

따라서 `MeasureSpec.UNSPECIFIED` 일 경우에는 뷰의 최소 크기를 측정 제안치로 계속 설정합니다. 부모뷰그룹이 `MeasureSpec.UNSPECIFIED `일 때는 자식 뷰가 제안한 측정치를 가지고 다시 측정 과정을 거칩니다. 그러다 `MeasureSpec.AT_MOST` 이거나 `MeasureSpec.EXACTLY` 일 경우에는 측정된 값을 기본값으로 설정합니다.

`onMeasure()`에서 `super.onMeasure()`를 호출하지 않고 `setMeasuredDimension()`을 내맘대로 설정하면 어떻게 될까?

```java
@Override
protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
    setMeasuredDimension(4000, 4000);
}
```

측정모드를 번갈아 가며 여러번 측정되지만 onLayout과 onDraw를 보면 결국에는 무턱대고 설정한 4000x4000이 된다. 그렇다면 뷰는 화면에 어떻게 보일까?

<img width="200" alt="Screen Shot 2021-08-19 at 5 10 23 PM" src="https://user-images.githubusercontent.com/55446114/132095352-4ae897d6-59e4-45fd-bbf8-2981950bff55.png">

부모 뷰의 크기로 설정한 100dp, 100dp 를 넘기지 못한다. 즉, setMeasuredDimension()메서드를 사용해서 구현자 마음대로 뷰의 크기를 설정할 수 있다. 하지만 정상적으로 화면에 표시하기 위해서는 최대값을 고려해서 뷰를 표시해야 한다. 

* RelativeLayout을 사용하지 않고 ConstraintLayout을 사용하면 어떨까?

```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">
    <androidx.constraintlayout.widget.ConstraintLayout
        android:layout_width="100dp"
        android:layout_height="100dp"
        app:layout_constraintLeft_toLeftOf="parent"
        app:layout_constraintTop_toTopOf="parent"
        >
        <com.example.viewtest.CustomView
            android:layout_width="100dp"
            android:layout_height="100dp"
            android:background="@color/black"
            app:layout_constraintTop_toTopOf="parent"
            app:layout_constraintLeft_toLeftOf="parent"
            />
    </androidx.constraintlayout.widget.ConstraintLayout>

</androidx.constraintlayout.widget.ConstraintLayout>
```

<img width="841" alt="Screen Shot 2021-09-04 at 10 07 46 PM" src="https://user-images.githubusercontent.com/55446114/132095433-f80f9cf9-0411-4c9b-8ce6-1f6fcde76fb3.png">

onMeasure() 호출이 2번으로 줄어들었다. ConstraintLayotu 을 사용해야 하는 이유가 여기서 드러난다. 





## Activity , Fragment Lifecycle

`Activity` 는 기본적으로 4가지 상태가 있습니다.

1. activity가 화면의 foreground(최상위 스택의 가장 높은 위치)에 있는 경우 활동 중이거나 실행 중입니다. 이것은 일반적으로 사용자가 현재 상호 작용하고 있는 activity입니다.
2. activity가 focus를 잃었지만 여전히 사용자에게 표시되는 경우에는 visible 합니다. 어떤 경우에 이런일이 발생하냐면, 전체 크기가 아닌 새로운 activity 또는 투명 activity가 activity 상단에 focus 되거나,  다른 activity가 다중 창 모드에서 더 높은 위치에 있거나  activity 자체가 현재 창 모드에서 focus를 맞출 수 없는 경우 가능합니다.
3. activity가 다른 activity에 의해 완전히 가려지면 stopped되거나 hidden 되어 집니다.  여전히 모든 상태 및 구성원 정보를 유지하지만 더 이상 사용자에게 표시되지 않으므로 창은 숨겨지고 다른 곳에서 메모리가 필요할 때 시스템에 의해 종종 종료됩니다.
4. 시스템은 activity가 finish되거나 단순히 프로세스를 종료하여 파괴함으로써 메모리에서 활동을 삭제할 수 있습니다.  이 경우, 사용자에게 다시 표시되어지려면  완전히 다시 시작하고 이전 상태로 복원해야 합니다.

다음 다이어그램은 활동의 중요한 상태 경로를 보여줍니다. 사각형은 Activity가 상태 간에 이동할 때 작업을 수행하기 위해 구현할 수 있는 콜백 메서드를 나타냅니다. 색칠된 타원은 활동이 있을 수 있는 주요 상태입니다

<img width="300" alt="Screen Shot 2021-09-04 at 10 19 06 PM" src="https://user-images.githubusercontent.com/55446114/132095795-1b954de8-4975-4851-9223-18d03befeb83.png">

3가지 반복되는 Loop 가 있다는 것에 주목해야 합니다.

1. activity의 전체 수명은 onCreate(Bundle)에 대한 첫 번째 호출에서 onDestroy() 최종 호출 사이에 발생합니다.  
   activity는 onCreate()에서 "전역" 상태의 모든 설정을 수행하고 onDestroy()에서 나머지 모든 리소스를 해제합니다.  예를 들어, 네트워크에서 데이터를 다운로드하기 위해 백그라운드에서 실행 중인 스레드가 있는 경우 onCreate()에서 해당 스레드를 생성한 다음 onDestroy()에서 스레드를 중지할 수 있습니다.
2. Activity의 가시적 수명은 onStart ~ onStop. 
   이 시간 동안 사용자는 화면에서 활동을 볼 수 있지만 foreground에 있지  않아도  사용자와 상호 작용할 수 있습니다. 이 두 가지 method 사이에서 사용자에게 activity를 표시하는 데 필요한 리소스를 유지할 수 있습니다.  
   예를 들어, UI에 영향을 미치는 변경 사항을 모니터링하기 위해 onStart()에 BroadcastReceiver를 등록하고, 사용자가 더 이상 표시하고 있는 것을 볼 수 없을 때 onStop()에서 등록을 취소할 수 있습니다. Activity가 사용자에게 표시되고 숨겨짐에 따라 onStart() 및 onStop() 메서드를 여러 번 호출할 수 있습니다.
3. Activity의 foreground 수명은 onResume() ~ onPause()  
   이 시간 동안 activity는 가시적이고 활동적이며 사용자와 상호 작용합니다. Activity는 rusumed된 상태와 paused 상태 사이를 자주 왔다갔다 할 수 있습니다.  
    예를 들어 기기가 절전 모드로 전환될 때, 활동 결과가 전달될 때, 새 intent가 전달될 때 등등.. 이러한 method의 코드는 상당히 가벼워야 합니다. 

<img width="500" alt="Screen Shot 2021-09-04 at 10 27 02 PM" src="https://user-images.githubusercontent.com/55446114/132096044-aa7fc180-e29d-4148-a606-401ae02790d6.png">

| method    | killable       | description                                                  | next                     |
| --------- | -------------- | ------------------------------------------------------------ | ------------------------ |
| onCreate  | no             | 액티비티가 처음 생성될때 호출.  여기에서 모든 일반 static  설정을 해야 함.  create view bind data to lists 이전에 frozon state? 된 적이 있는 경우 Bundle 도 함께 제공됩니다. | onStart                  |
| onRestart | no             | 액티비티가 중지되고 다시 시작할 때 호출됨.                   | onStart                  |
| onStart   | no             | 액티비티가 사용자에게 보여질 때 호출됨  액티비티가 foreground로 가면 onResume이 호출되고 ,  숨겨지면 onStop 이 호출됨 | onResume<br />onStop     |
| onResume  | no             | 액티비티가 사용자와 상호작용하기 시작할 때 호출됩니다. 이 시점에서 액티비티는 사용자 입력을 받을 수 있고 액티비티 스택의 맨 위에 있습니다. 항상 onPause() 가 뒤따름. | onPause                  |
| onPause   | HONEYCOMB 이전 | 액티비티가 foreground 상태에서 벗어날때 <br /> focus를 잃을 때  <br />stopped/hidden 또는 destroyed 상태로 전환되기 전에 호출됨.<br />  액티비티는 여전히 사용자에게 표시되므로 시각적으로 활성 상태를 유지하고 UI를 계속 업데이트 하는 것이 좋다. <br />이 메서드가 반환될 때 까지 다음 액티비티가 resumed 되지 않기 때문에 이 메서드의 구현은 빨라야 합니다.  <br />액티비티가 다시 foreground로 돌아오면, onResume() ,  사용자에게 보여지지 않으면 onStop() 이 뒤따름 | onResume onStop          |
| onStop    | Yes            | 액티비티가 더이상 사용자에게 보여지지 않을 때 호출됨 <br />새로운 액티비티가 스택 맨 위에서 시작되고 있거나  <br />기존에 존재하던 액티비티가 이 액티비티 앞으로 가져와지거나  <br />이 액티비티가 파괴되기 때문에 발생 <br />일반적으로 애니메이션을 중지하고 UI를 새로 고치는 등의 작업에 사용 <br />이 액티비티가 사용자와 다시 상호작용하기 위해 다시 foreground 로 오는 경우 onResatrt() 가 뒤따르고,  이 액티비티가 사라지면 onDestroy()가 뒤따름 | onRestart<br />onDestroy |
| onDestroy | Yes            | 액티비티가 소멸되기 전에 받는 마지막 호출<br /> 이것은 액티비티가 완료중이거나(Activity#finish) <br />시스템이 공간을 절약하기 위해 이 액티비티 인스턴스를 일시적으로 파괴하기 때문에 발생 <br />isFinishing() 메서드를 사용하여 이 두 시나리오를 구분할 수 있습니다. | Nothing                  |

❗  `Killable` 컬럼에 주의!!

종료 가능한 것으로 표시된 메서드의 경우, 종료 가능한 것으로 표시된 메서드의 경우 해당 메서드가 활동을 호스팅하는 프로세스를 반환한 후 다른 코드 행이 실행되지 않고 시스템에 의해 언제든지 종료될 수 있습니다. 이 때문에 onPause() 메서드를 사용하여 영구 데이터(예: 사용자 편집)를 스토리지에 기록해야 합니다. `onSaveInstanceState(android.os.Bundle)` 메서드는 액티비티를 이러한 background 상태에 배치하기 전에 호출되므로 액티비티의 동적 인스턴스 상태를 지정된 `Bundle`에 저장하여 나중에 `onCreate(Bundle)`에서 수신할 수 있습니다. `onSaveInstanceState(Bundle)` 대신 `onPause()`에 영구 데이터를 저장하는 것이 중요합니다. 전자는 수명 주기 콜백의 일부가 아니므로 설명서에 설명된 대로 모든 상황에서 호출되지는 않기 때문입니다.

`Build.VERSION_CODES.HONEYCOMB` 이전과 이후 버전이 약간 다릅니다. Honeycomb부터 애플리케이션은 onStop()이 반환될 때까지 종료 가능한 상태가 아닙니다. 이는 onSaveInstanceState(android.os.Bundle)가 호출될 때 영향을 미치고(onPause() 이후에 안전하게 호출될 수 있음) 애플리케이션이 지속적 상태를 저장하기 위해 onStop()까지 안전하게 기다릴 수 있습니다.

`Build.VERSION_CODES.P` 로 시작하는 플랫폼을 대상으로 하는 애플리케이션의 경우 onSaveInstanceState(android.os.Bundle)는 항상 onStop() 이후에 호출되므로 애플리케이션은 onStop()에서 프래그먼트 트랜잭션을 안전하게 수행할 수 있으며 나중에 영구 상태를 저장할 수 있습니다.

메모리 부족이 심한 경우 시스템은 언제든지 응용 프로그램 프로세스를 종료할 수 있습니다.

https://stackoverflow.com/questions/8515936/android-activity-life-cycle-what-are-all-these-methods-for

<img width="500" alt="Screen Shot 2021-09-04 at 10 25 14 PM" src="https://user-images.githubusercontent.com/55446114/132095994-f217884b-5418-4328-9d9f-893ee02027d5.png">

- **When open the app**

  ```
  onCreate() --> onStart() -->  onResume()
  ```

- **When back button pressed and exit the app**

  ```
  onPaused() -- > onStop() --> onDestory()
  ```

- **When home button pressed**

  ```
  onPaused() --> onStop()
  ```

- **After pressed home button when again open app from recent task list or clicked on icon**

  ```
  onRestart() --> onStart() --> onResume()
  ```

- **When open app another app from notification bar or open settings**

  ```
  onPaused() --> onStop()
  ```

- **Back button pressed from another app or settings then used can see our app**

  ```
  onRestart() --> onStart() --> onResume()
  ```

- **Any phone is ringing and user in the app**

  ```
  onPause() --> onResume()
  ```

- **After call end**

  ```
  onResume()
  ```

- **When phone screen off**

  ```
  onPaused() --> onStop()
  ```

- **When screen is turned back on**

  ```
  onRestart() --> onStart() --> onResume()
  ```

### Activity 상태에 따른 Fragment Lifecycle

<img width="300" alt="Screen Shot 2021-09-04 at 10 28 57 PM" src="https://user-images.githubusercontent.com/55446114/132096106-c07cc5df-8739-4292-a9d6-1963abf3b718.png">

onCreate() 가 호출되면 activity 상태가 craeted가 되고, Fragment의 onAttatch() ~ onActivityCreated() 가 순서대로 호출됩니다. 

onStart() 가 호출되면 activity 상태가 started 가 되고, Fragment의 onStart() 가 호출됩니다. 

이런식으로 프래그먼트가 있는 액티비티는 해당 프래그먼트의 생명주기에 영향을 미칩니다. 

### Fragment 전환시 

```xml
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">

    <androidx.fragment.app.FragmentContainerView
        android:id="@+id/fcv"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        app:layout_constraintTop_toTopOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintBottom_toTopOf="@id/btn_game"
        app:layout_constraintVertical_chainStyle="packed"
        />
    <Button
        android:id="@+id/btn_game"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="game"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintEnd_toStartOf="@id/btn_setting"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintVertical_chainStyle="packed"
        />
    <Button
        android:id="@+id/btn_setting"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="setting"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toEndOf="@id/btn_game"
        app:layout_constraintBottom_toBottomOf="parent"
        />

</androidx.constraintlayout.widget.ConstraintLayout>
```

```kotlin
import androidx.appcompat.app.AppCompatActivity
import android.os.Bundle
import android.util.Log
import android.widget.Button
import androidx.fragment.app.*
import com.example.fragmenttest.enumtest.Card
import com.example.fragmenttest.enumtest.FruitType
import com.example.fragmenttest.enumtest.NumberType

class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        supportFragmentManager.registerFragmentLifecycleCallbacks(FragmentLifecycleLogCallback() , false)
        val gameFragment = GameFragment()
        val settingFragment = SettingFragment()
        if (savedInstanceState == null) {
            supportFragmentManager.beginTransaction()
                .replace(R.id.fcv, gameFragment)
                .setTransition(FragmentTransaction.TRANSIT_FRAGMENT_OPEN)
                //.addToBackStack(null)
                .commit()
        }

        findViewById<Button>(R.id.btn_game).setOnClickListener {
            supportFragmentManager.beginTransaction()
                .replace(R.id.fcv, gameFragment)
                .setTransition(FragmentTransaction.TRANSIT_FRAGMENT_OPEN)
                //.addToBackStack(null)
                .commit()
        }
        findViewById<Button>(R.id.btn_setting).setOnClickListener {
            supportFragmentManager.beginTransaction()
                .replace(R.id.fcv, settingFragment)
                .setTransition(FragmentTransaction.TRANSIT_FRAGMENT_OPEN)
                //.addToBackStack(null)
                .commit()
        }
    }

    override fun onRestart() {
        super.onRestart()
        Log.d("life", "onRestart : MainActivity")
    }
}
```

- 프래그먼트 replace() 할 때 backstack에 저장하지 않은 경우 

<img width="789" alt="Screen Shot 2021-09-04 at 10 44 43 PM" src="https://user-images.githubusercontent.com/55446114/132096557-3806fb5b-5283-4df5-9f25-668a53aae229.png">

전환되는 GameFragment가 destroy 됩니다. (Activity에는 아무런 영향이 없다.)

addToBackStack 주석을 제거하면, 

<img width="789" alt="Screen Shot 2021-09-04 at 10 47 00 PM" src="https://user-images.githubusercontent.com/55446114/132096626-a2eda949-cca6-4e69-991e-86430aebc359.png">

destroy 되지 않습니다. 

