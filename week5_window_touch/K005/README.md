# Touch System

# 목표

터치 이벤트 종류

제스처

액티비티 부터 터치 대상 까지 터치 이벤트가 전달되는 과정

터치 이벤트 전달의 응용

# 터치 이벤트

안드로이드의 위젯과 상호작용이 필요할 때 클릭 등의 간단한 동작이라면 `setOnClickListener`를 설정하는 것으로 충분하다. 하지만 클릭에서 각각의 동작을 구분하여 처리할 필요가 있거나, 좀 더 복잡한 상호작용을 위해서는 뷰에 전달되는 터치 이벤트마다 각각의 작업을 정의해야 한다.

안드로이드의 터치 이벤트는 `View`의 `OnTouchListener#onTouch` 또는 `onTouchEvent#onTouchEvent` 메소드에 `MotionEvent`의 형태로 전달된다.

각각의 `MotionEvent`는 터치가 시작되었는지, 터치가 된 상태에서 움직였는지, 터치가 종료되었는지 등 action에 대한 정보를 가지고 있다. 해당 액션 정보를 사용하여 분기하면 터치의 세부 동작마다 수행할 작업을 정의할 수 있다.

[MotionEvent](https://developer.android.com/reference/android/view/MotionEvent)의 문서에서는 `MotionEvent`의 액션에 대한 정보를 제공한다. 문서에 따라 현재 `Deprecated` 되지 않은 액션은 다음과 같다. 여기서 `POINTER`와 관련된 액션은 멀티 터치 이벤트 처리에서 사용된다.

- `ACTION_DOWN`
- `ACTION_UP`
- `ACTION_MOVE`
- `ACTION_CANCEL`
- `ACTION_OUTSIDE`
- `ACTION_POINTER_DOWN`
- `ACTION_POINTER_UP`
- `ACTION_HOVER_MOVE`
- `ACTION_SCROLL`
- `ACTION_HOVER_ENTER`
- `ACTION_HOVER_EXIT`
- `ACTION_BUTTON_PRESS`
- `ACTION_BUTTON_RELEASE`

# 제스처

위젯과 상호작용하기 위해 일반적인 터치 이벤트 외에도 더블 탭, 튕기기(fling) 등의 제스처에 따른 동작이 필요할 수도 있다. `onTouchEvent`를 오버라이드하거나 커스텀 제스처를 정의할 수도 있지만 이를 직접 구현하는 것은 복잡한 과정이며 실수가 발생하기 쉽다.

제스처를 사용하는 가장 간편한 방법은 안드로이드가 제공하는 `GestrueDetector`를 사용하는 것이다.

`GestureDetector`는 제스처를 감지하는 몇 가지 리스너를 제공한다.

```java
public interface OnGestureListener {
    boolean onDown(MotionEvent e);

    void onShowPress(MotionEvent e);

    boolean onSingleTapUp(MotionEvent e);

    boolean onScroll(MotionEvent e1, MotionEvent e2, float distanceX, float distanceY);

    void onLongPress(MotionEvent e);

    boolean onFling(MotionEvent e1, MotionEvent e2, float velocityX, float velocityY);
}

public interface OnDoubleTapListener {
    boolean onSingleTapConfirmed(MotionEvent e);

    boolean onDoubleTap(MotionEvent e);

    boolean onDoubleTapEvent(MotionEvent e);
}

public interface OnContextClickListener {
    boolean onContextClick(MotionEvent e);
}
```

해당 제스처를 감지하기 위해선 위의 인터페이스를 구현하는 리스너를 생성하고, 해당 리스너를 사용하여 `GestureDetectorCompat` 객체를 생성한다.

이후 제스처를 감지해야 하는 뷰의 `onTouchEvent` 메소드에서 `GestureDetectorCompat`에게 `MotionEvent`를 전달하는 것으로 제스처 리스너를 사용할 수 있다.

```kotlin
class CustomView(
    context: Context, attr: AttributeSet? = null
) : View(context, attr), GestureDetector.OnGestureListener, GestureDetector.OnDoubleTapListener {

    private val gestureDetector = GestureDetectorCompat(context, this).apply {
        setOnDoubleTapListener(this@CustomView)
    }

    override fun onDown(e: MotionEvent?): Boolean {
        TODO("Not yet implemented")
    }

    override fun onShowPress(e: MotionEvent?) {
        TODO("Not yet implemented")
    }

    override fun onSingleTapUp(e: MotionEvent?): Boolean {
        TODO("Not yet implemented")
    }

    override fun onScroll(
        e1: MotionEvent?, e2: MotionEvent?, distanceX: Float, distanceY: Float
    ): Boolean {
        TODO("Not yet implemented")
    }

    override fun onLongPress(e: MotionEvent?) {
        TODO("Not yet implemented")
    }

    override fun onFling(
        e1: MotionEvent?, e2: MotionEvent?, velocityX: Float, velocityY: Float
    ): Boolean {
        TODO("Not yet implemented")
    }

    override fun onSingleTapConfirmed(e: MotionEvent?): Boolean {
        TODO("Not yet implemented")
    }

    override fun onDoubleTap(e: MotionEvent?): Boolean {
        TODO("Not yet implemented")
    }

    override fun onDoubleTapEvent(e: MotionEvent?): Boolean {
        TODO("Not yet implemented")
    }

    override fun onTouchEvent(event: MotionEvent?): Boolean {
        gestureDetector.onTouchEvent(event)

        return super.onTouchEvent(event)
    }
}
```

# 안드로이드 터치 시스템

## Before

**Q: 레이아웃과 뷰가 중첩되어 있을 터치 이벤트를 처리하는 target**

- 터치한 포인트에서 최상단에 위치한 뷰가 터치 이벤트를 처리한다.

**Q: Boolean 값을 반환하는 이벤트의 의미**

`Boolean` 값을 반환하는 이벤트에서 `true`를 반환한다는 것은 여러 가지 의미를 가진다.

- 현재 이벤트에 대해 관심이 있으니 후속 이벤트를 보내달라.
  
    : 뷰에서 `ACTION_DOWN`을 수신했을 때 `true`를 반환하면, 해당 뷰에서 후속 이벤트(`ACTION_MOVE`, `ACTION_UP`, ...)가 호출된다.
    
- 현재 이벤트를 처리하였으니 이벤트에 대한 추가 작업을 하지 마라.
  
    : `onLongClick` 등의 이벤트에서 `true`를 반환하면 `onClick` 이벤트는 호출하지 않는다.
    

**Q: 터치 이벤트를 등록하는 방법**

터치 이벤트는 여러 가지 방법을 사용하여 수신할 수 있다.

- `Activity#onTouchEvent`
- `View#onTouchEvent`
- `onTouchListener#onTouch`

## 터치 이벤트의 전달 과정

안드로이드 터치 시스템의 동작 과정을 알아보기 위해 먼저 공식 문서에서 터치 시스템에 대해 소개하는 내용을 살펴보자.

> *이벤트를 처리하기 위해 기본적으로 제공하는 이벤트 리스너와 핸들러를 사용할 수도 있지만, 레이아웃 안에서 좀 더 복잡한 이벤트를 관리하는 경우 다음 메서드를 사용할 수도 있다.*
- [Activity.dispatchTouchEvent(MotionEvent)](https://developer.android.com/reference/android/app/Activity#dispatchTouchEvent(android.view.MotionEvent))
  
    터치 이벤트가 `window`에 전달되기 전에 액티비티에서 먼저 처리할 수 있다.
    
- [ViewGroup.onInterceptTouchEvent(MotionEvent)](https://developer.android.com/reference/android/view/ViewGroup#onInterceptTouchEvent(android.view.MotionEvent))
  
    자식 뷰로 전달되는 터치 이벤트를 부모 뷰에서 가로채어 처리할 수 있다. 이 메소드에서 `true`를 반환하면 터치 이벤트는 자식 뷰로 전달되지 않고 부모 뷰에서 처리된다.
    
    `ViewGroup`의 메소드이기 때문에 `View`에서는 사용할 수 없다.
    
- [ViewParent.requestDisallowInterceptTouchEvent(boolean)](https://developer.android.com/reference/android/view/ViewParent#requestDisallowInterceptTouchEvent(boolean))
  
    자식 뷰에서 부모 뷰가 `onInterceptTouchEvent`를 통해 터치 이벤트를 가로채는 것을 막기 위해 사용된다. 이 메소드는 하나의 터치 이벤트에 대해 동작하기 때문에 지속적인 `disallow`가 필요하다면 `onTouchEvent` 내에서 터치 이벤트가 발생할 때 마다 호출해야 한다.
    

그럼 일반적인 터치 이벤트가 어떻게 전달되는지 확인해보자.

## 터치 이벤트의 전달 과정

시뮬레이션을 위해 다음과 같은 레이아웃을 사용한다. 각각의 레이어는 `FrameLayout`을 상속하며, 터치 이벤트 관련 메소드에서 로그를 출력한다.

```xml
<?xml version="1.0" encoding="utf-8"?>
<com.acp.week5.Layer1 xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:id="@+id/layer1"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:tag="Layer1"
    tools:context=".MainActivity">

    <com.acp.week5.Layer2
        android:layout_width="match_parent"
        android:layout_height="450dp"
        android:background="#A5D6A7"
        android:tag="Layer2">

        <com.acp.week5.Layer3
            android:layout_width="match_parent"
            android:layout_height="200dp"
            android:background="#4CAF50"
            android:tag="Layer3">

        </com.acp.week5.Layer3>

    </com.acp.week5.Layer2>

</com.acp.week5.Layer1>
```

![Untitled](./res/picture (1).png)

가장 녹색 영역이 짙은 Layer3을 터치했을 때의 로그는 다음과 같이 출력된다.

![Untitled](./res/picture (2).png)

로그를 확인하면 터치 이벤트가 발생했을 때 먼저 `Activity`에서 `dispatchTouchEvent`가 호출되고, `onInterceptTouchEvent`에서 터치 이벤트 인터셉트 여부를 확인하고, 레이아웃 계층을 따라 터치 이벤트가 전파되는 것을 볼 수 있다.

계층을 따라 전파된 터치 이벤트는 타겟에 도달했을 때 `TouchEvent`를 호출하며, 이벤트를 사용할 때는 타겟 뷰부터 액티비티까지 역순으로 `TouchEvent`가 호출된다. 또한 이벤트 소모에 대한 처리를 하지 않았기 때문에 후속 이벤트에 대해선 액티비티가 처리하고 있는 것을 볼 수 있다.

그렇다면 이번엔 Layer3의 `onTouchEvent`에서 `true`를 반환했을 때의 로그를 확인해보자.

![Untitled](./res/picture (3).png)

터치 이벤트가 전달 되는 과정은 동일하지만 Layer3의 `onTouchEvent`에서 `true`를 반환했기 때문에 후속 이벤트가 Layer3에 전달되는 것을 볼 수 있다.

만약 여기서 Layer1이 터치 이벤트를 인터셉트 한다면 어떻게 될까?

![Untitled](./res/picture (4).png)

터치 이벤트가 전달되는 과정에서 Layer1에 의해 인터셉트 되었기 때문에 자식 뷰로 전달되지 않고 Layer1의 `onTouchEvent`가 즉시 호출된다. 하지만 Layer1의 `onTouchEvent`에서 이벤트 소모에 대한 처리를 하지 않아서 후속 이벤트는 액티비티에 전달되었다.

지금까지 한 일을 잠깐 요약해보자.

자식 뷰의 `onTouchEvent`에서 `true`를 반환하면 터치 이벤트가 상위 뷰로 전달되지 않으며, 후속 이벤트를 자식 뷰에서 받을 수 있다.

부모 뷰의 `onInterceptTouchEvent`에서 `true`를 반환하면 자식 뷰로 터치 이벤트를 전달하지 않고 부모 뷰에서 해당 이벤트를 처리한다.

그렇다면 `View#dispatchTouchEvent`에서 반환 값의 의미는 무엇일까?

해당 메소드의 코멘트에서는 다음과 같이 설명한다.

> Pass the touch screen motion event down to the target view, or this view if it is the target.
> 

`disaptchTouchEvent` 메소드는 터치 이벤트를 자식 뷰로 전달하거나, 자신이 뷰가 타겟이라면 자신에게 터치 이벤트를 전달한다고 말한다. 즉, `disaptchTouchEvent`에서 `true`를 반환한다는 것은 시스템에게 자신이 터치 이벤트를 받는 대상이라고 알리는 것이 된다.

하지만 여기서 한 가지 알아야 할 점은 `disaptchTouchEvent`의 반환 값이 사용되는 시점이다.

`disaptchTouchEvent` 에서는 `onInterceptTouchEvent`를 호출하고, 자식 뷰에 대해 `dispatchTouchEvent`를 호출한 다음, `onTouchEvent`를 호출하고 종료된다. 즉, `dispatchTouchEvent`의 반환 값은 터치 이벤트가 모든 자식 뷰들에게 한 번씩 전달되고 나서 후속 이벤트가 발생하는 시점에서 의미를 가진다.

이전의 작업들을 지우고 Layer1에서 `disaptchTouchEvent`가 true를 반환하는 작업만 추가하여 로그를 확인해보자.

![Untitled](./res/picture (5).png)

로그에서 볼 수 있듯 처음의 터치 이벤트가 자식 뷰들에게 전파된 후 후속 이벤트에 대해서만 Layer1이 처리하고 있다. 따라서 부모 뷰에서 자식 뷰의 터치 이벤트를 가로챈 후 후속 이벤트에 대해서도 부모 뷰가 처리할 수 있도록 구현하려면 부모 뷰에서 `onInterceptTouchEvent`가 true를 반환하도록 수정할 뿐만 아니라, `onTouchEvent` 또는 `dispatchTouchEvent`에서 `true`를 반환하여 후속 이벤트를 받는 대상이 자신이라는 것을 알려야 한다.

하지만 `onInterceptTouchEvent`와 `dispatchTouchEvent`를 단순히 `true`로 설정한다면 자식 뷰들이 터치 이벤트를 수신할 수 없기 때문에 사용에 주의가 필요하다. 일반적인 경우라면 `onTouchEvent`에서 처리하는 것이 권장되는 듯 하다.

여기 까지 터치 이벤트가 전달되는 과정을 이해했으면 아래 그림이 어떤 의미를 가지는지 알 수 있다.

![Untitled](./res/picture (6).png)

출처: [https://stackoverflow.com/questions/7449799/how-are-android-touch-events-delivered/46862320#46862320](https://stackoverflow.com/questions/7449799/how-are-android-touch-events-delivered/46862320#46862320)

![Untitled](./res/picture (7).png)

출처: [https://stackoverflow.com/questions/7449799/how-are-android-touch-events-delivered/46862320#46862320](https://stackoverflow.com/questions/7449799/how-are-android-touch-events-delivered/46862320#46862320)

다만 위의 그림에서는 한 가지 오류가 있는데, 가장 아래 `View`에서 `onTouchListener`를 처리하는 부분이다.

그림에 따르면 뷰에서 `onTouchListener`가 설정되어 있는 경우 `onTouchEvent` 대신 `onTouch`를 수행하고 부모 뷰로 전달되는데, 실제로는 `onTouch`에서 `false`를 반환했을 때 해당 뷰의 `onTouchEvent`가 실행되고, `onTouchEvent`에서도 `false`를 반환했을 때 부모 뷰로 이벤트가 전달된다.

여기에 따르면 다음과 같이 그림을 수정할 수 있다.

![Untitled](./res/picture (8).png)

# 터치 이벤트의 응용

## EditText의 Touch 영역 밖에서 키보드 숨기기

일반적인 UX로 사용자가 `EditText` 작업 중 다른 영역을 터치하면 소프트 키보드가 숨겨지기를 기대한다. 하지만 이는 `EditText`가 자체적으로 제공하는 기능이 아니기 때문에 개발자가 해당 기능을 구현해야 한다.

이를 위해서는 모든 터치 이벤트가 액티비티부터 `dispatch` 된다는 것을 활용할 수 있다.

액티비티에서 터치 이벤트를 수신했을 때 현재 포커스가 `EditText`에 있고, 터치된 영역이 `EditText`가 차지하는 영역 밖이라면 소프트 키보드를 숨기는 방식으로 구현할 수 있다.

```kotlin
override fun dispatchTouchEvent(ev: MotionEvent): Boolean {
    currentFocus?.let {
        if ((ev.action == MotionEvent.ACTION_UP)
            && it is EditText
            && !it::class.java.name.startsWith("android.webkit.")
        ) {
            val locationOnScreen = IntArray(2)
            it.getLocationOnScreen(locationOnScreen)

            val x = ev.rawX + it.getLeft() - locationOnScreen[0]
            val y = ev.rawY + it.getTop() - locationOnScreen[1]

            if (x < it.getLeft() || x > it.getRight() || y < it.getTop() || y > it.getBottom()) {
                val imm = getSystemService(Context.INPUT_METHOD_SERVICE) as InputMethodManager
                imm.hideSoftInputFromWindow(it.windowToken, 0)
                it.clearFocus()
            }
        }
    }
    return super.dispatchTouchEvent(ev)
}
```

## 이중 스크롤

vertical 리사이클러 뷰 내부에 horizontal 리사이클러 뷰가 중첩된 구조를 생각해보자(그 반대도 관계 없다).

리사이클러 뷰는 구현 상 현재 스크롤중인 리사이클러 뷰가 터치 이벤트를 수신하게 된다.

좀 더 구체적으로 말해보자면 위의 레이아웃에서 수직 방향으로 스크롤 중인 경우 외부 리사이클러 뷰의 `onInterceptTouchEvent`에서 `true`를 반환하기 때문에 스크롤 중의 터치는 외부 리사이클러 뷰에게 전달된다.

내부 리사이클러 뷰의 경우에는 스크롤 중 `requestDisallowInterceptTouchEvent`를 호출하기 때문에 수평 방향 스크롤 중의 터치가 발생해도 외부 리사이클러 뷰에서 가로챌 수 없다.

이러한 동작은 스크롤이 빠르게 이루어질때 이질감을 줄 수 있다.

[참고영상 1](https://raw.githubusercontent.com/rubensousa/RecyclerViewNestedExample/master/videos/nested_problem1.webm)

[참고영상 2](https://raw.githubusercontent.com/rubensousa/RecyclerViewNestedExample/master/videos/nested_problem2.webm)

출처: [https://rubensousa.com/2019/08/16/nested_recyclerview_part1/](https://rubensousa.com/2019/08/16/nested_recyclerview_part1/)

각각의 리사이클러 뷰에서 터치 이벤트를 가로채지 않고 매끄럽게 동작시키기 위해서는 리사이클러 뷰의 `onInterceptTouchEvent`를 재정의하여 터치 이벤트를 가로채는 로직을 변경해야 한다.

```kotlin
override fun onInterceptTouchEvent(e: MotionEvent): Boolean {
    val lm = layoutManager ?: return super.onInterceptTouchEvent(e)
    var allowScroll = true

    when (e.actionMasked) {
        MotionEvent.ACTION_DOWN -> {
            lastX = e.x
            lastY = e.y
            // If we were scrolling, stop now by faking a touch release
            if (scrolling) {
                val newEvent = MotionEvent.obtain(e)
                newEvent.action = MotionEvent.ACTION_UP
                return super.onInterceptTouchEvent(newEvent)
            }
        }
        MotionEvent.ACTION_MOVE -> {
            // We're moving, so check if we're trying
            // to scroll vertically or horizontally so we don't intercept the wrong event.
            val dx = abs(e.x - lastX)
            val dy = abs(e.y - lastY)
            allowScroll = if (dy > dx) lm.canScrollVertically() else lm.canScrollHorizontally()
        }
    }
    return if (!allowScroll) {
        false
    } else super.onInterceptTouchEvent(e)
}
```

메소드의 구현을 보면 터치 이벤트가 발생했을 때 터치된 포인트를 저장하고, 현재 스크롤중이라면 fake 모션 이벤트를 만들어서 전달한다. *(테스트 중 fake 이벤트를 전달할 필요가 없을 듯 하여 단순히 false를 반환하는 것으로 변경해 봤는데 동작에 문제가 없었다.*

이후 ACTION_MODE에서 이동된 포인트와 기존의 터치 포인트의 차이를 계산해서 스크롤의 방향을 결정하고, 해당 방향으로 스크롤 할 수 있는지 확인한다.

마지막으로 `allowScroll`을 확인하여 `false`인 경우 `onInterceptTouchEvent`가 `false`를 반환하는데, 이는 수직 방향 스크롤 중 수평 방향으로 스크롤 하거나, 그 반대의 경우 내부 또는 외부의 리사이클러 뷰가 터치 이벤트를 처리해야 하므로 스크롤 중인 리사이클러 뷰에서는 터치 이벤트를 가로채지 않고 넘긴다는 것을 의미한다.

스크롤 뷰와 리사이클러 뷰에서 터치 이벤트가 전달되는 과정은 [스크롤 인터셉트](https://www.youtube.com/watch?v=j5uIR6eLt7E&ab_channel=naverd2)라는 영상에서 자세히 설명한 후 솔루션을 제공하고 있다. 하지만 솔루션의 경우 내부 리사이클러 뷰의 스크롤 중엔 수직 스크롤이 동작하지 않아 생략하였다.

# 참고

## 터치 이벤트

[https://developer.android.com/guide/topics/ui/ui-events?hl=ko](https://developer.android.com/guide/topics/ui/ui-events?hl=ko)

[https://developer.android.com/training/gestures/viewgroup?hl=ko](https://developer.android.com/training/gestures/viewgroup?hl=ko)

[https://jamssoft.tistory.com/161](https://jamssoft.tistory.com/161)

[https://readystory.tistory.com/185](https://readystory.tistory.com/185)

[https://zzandoli.tistory.com/13](https://zzandoli.tistory.com/13)

[https://jhy156456.tistory.com/entry/터치-이벤트-간략-정리onTouchEvent-dispatchTouchEvent-onInterceptTouchEvent](https://jhy156456.tistory.com/entry/%ED%84%B0%EC%B9%98-%EC%9D%B4%EB%B2%A4%ED%8A%B8-%EA%B0%84%EB%9E%B5-%EC%A0%95%EB%A6%AConTouchEvent-dispatchTouchEvent-onInterceptTouchEvent)

[http://daplus.net/java-안드로이드-onintercepttouchevent와-dispatchtouchevent의-차이점은-무엇입니까/](http://daplus.net/java-%EC%95%88%EB%93%9C%EB%A1%9C%EC%9D%B4%EB%93%9C-onintercepttouchevent%EC%99%80-dispatchtouchevent%EC%9D%98-%EC%B0%A8%EC%9D%B4%EC%A0%90%EC%9D%80-%EB%AC%B4%EC%97%87%EC%9E%85%EB%8B%88%EA%B9%8C/)

[https://stackoverflow.com/questions/7449799/how-are-android-touch-events-delivered/46862320#46862320](https://stackoverflow.com/questions/7449799/how-are-android-touch-events-delivered/46862320#46862320)

## 소프트 키보드

[https://stackoverflow.com/questions/8697499/hide-keyboard-when-user-taps-anywhere-else-on-the-screen-in-android/8697635](https://stackoverflow.com/questions/8697499/hide-keyboard-when-user-taps-anywhere-else-on-the-screen-in-android/8697635)

## 이중 스크롤

[https://www.youtube.com/watch?v=j5uIR6eLt7E&ab_channel=naverd2](https://www.youtube.com/watch?v=j5uIR6eLt7E&ab_channel=naverd2)

[https://rubensousa.com/2019/08/16/nested_recyclerview_part1/](https://rubensousa.com/2019/08/16/nested_recyclerview_part1/)

[https://www.charlezz.com/?p=1402](https://www.charlezz.com/?p=1402)
