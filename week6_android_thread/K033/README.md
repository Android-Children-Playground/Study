# 주제

- MainThread(UIThread)
- Looper, Message Queue
- Handler, view.Post, runOnUIThread

# 사전 지식

### Process

일반적으로, 하나의 실행 중인 프로그램을 나타냅니다.

프로그램을 실행하면 OS로부터 실행에 필요한 메모리(Code, Data, Stack, Heap)를 할당받아 프로세스가 됩니다.

![Untitled](https://user-images.githubusercontent.com/50517813/137630943-ff01c513-95ef-4848-9e2f-9db4d3f98be2.png)

운영체제는 프로세스를 관리하기 위해서 프로세스마다 고유의 PCB(프로세스 제어 블록)를 생성합니다.

PCB는 프로세스의 상태 정보를 저장하고 있는 자료구조로, 프로세스 상태 관리와 더불어 문맥 교환(Context switch)을 위해 사용됩니다.

![Untitled](https://user-images.githubusercontent.com/50517813/137630956-fadf257c-4f2f-49dc-b5f5-9eb5776a6c28.png)

Android에서 실행되는 하나의 앱을 개별적인 프로세스로 취급해 실행합니다.

참고 : [https://yoongrammer.tistory.com/52](https://yoongrammer.tistory.com/52)

### Thread

Thread란 동시 작업을 위한 하나의 실행 단위입니다.

한 개의 프로그램(프로세스)에서 많은 Thread를 동시에 처리할 수 있습니다. (Multi Thread)

그리고 프로세스의 메모리 또한 공유합니다. (stack만 따로 할당 받음)

프로세스로부터 각 메모리를 할당 받은 스레드는 독립적으로 실행됩니다.

![Untitled](https://user-images.githubusercontent.com/50517813/137630974-f9de0cdd-ca16-4549-9620-d153badb89dd.png)

참고 : [https://devlog-wjdrbs96.tistory.com/145](https://devlog-wjdrbs96.tistory.com/145)

### Kotlin Thread 생성

1. Thread 클래스를 상속하는 새로운 클래스 생성 후 run() 메서드 오버라이드

```kotlin
class AppThread1 : Thread() {
    override fun run()
    {
        /* 수행할 작업 작성 */
    }
}

```

위와 같이 생성된 스레드는 start() 메서드를 호출하여 실행합니다.

```kotlin
class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        
        var t = AppThread1()
        t.start()
    }
}
```

1. Runnable 인터페이스 사용

Runnable은 상속관계에 있는 클래스를 지원하기 위한 인터페이스입니다.

```kotlin
class AppThread1 : Runnable {
    override fun run()
    {
				/* 수행할 작업 작성 */
    }
}
```

Thread 클래스를 사용하는 경우와 동일하지만, 스레드를 실행시키는 방법은 객체 자체를 Thread 클래스의 생성자로 전달하여 실행시켜야 합니다.

```kotlin
var t = Thread(AppThread1())
t.start()
```

간단한 실행 메서드라면 람다식으로 구현할 수 있습니다.

```kotlin
class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        Thread {
            /* 수행할 작업 작성 */
        }.start()
    }
}
```

추가로 코틀린에서는 thread 처리 메서드를 구현해서 사용할 수 있습니다.

```kotlin
thread(start=true) {
		/* 수행할 작업 작성 */
}
```

참고 : [https://lab.cliel.com/entry/Kotlin-프로세스Process와-스레드Thread](https://lab.cliel.com/entry/Kotlin-%ED%94%84%EB%A1%9C%EC%84%B8%EC%8A%A4Process%EC%99%80-%EC%8A%A4%EB%A0%88%EB%93%9CThread)

# Android Thread

애플리케이션이 시작되면 시스템이 애플리케이션에 대한 Main Thread를 생성합니다.

![Untitled](https://user-images.githubusercontent.com/50517813/137631042-7ebd8655-a08e-4638-8d97-423b2f844d39.png)

### Main Thread

Drawable event를 포함하여 적절한 사용자 인터페이스 위젯에 이벤트를 발송하는 역할을 맡습니다.  대부분의 경우 Android UI 도구 키트의 구성 요소 (android.widget, android.view 패키지)와 개발자의 애플리케이션이 상호작용합니다. UIThread 라고 불립니다. (특수한 상황에서는 다를 수 있음)

### Single Thread Model

사용자 상호작용에 응답해서 리소스를 많이 소모하는 작업을 수행하는 경우, 애플리케이션을 제대로 구현하지 않으면 낮은 성능을 보일 수 있습니다. 특히 네트워크 액세스나 데이터베이스 쿼리 등의 긴 작업을 UI Thread에서 수행할 경우 앱이 정지될 수 있습니다.

UI를 구성해야하는 UI Thread에서 다른 작업을 처리하기 때문에 마치 앱이 멈춘 것처럼 보일 수 있습니다. 현재 UI 스레드가 일정 시간(현재 5초) 동안 대기상태를 유지한다면 "애플리케이션이 응답하지 않습니다"(ANR)라는 대화상자가 표시됩니다. 따라서 오랜 대기 시간을 요구하는 작업의 경우 Worker Thread에서 처리해야 합니다.

현재 Android OS에서는 UI 자원에 Main Thread와 Sub Thread가 동 접근하여 동기화 이슈를 발생시키는 것을 방지하기 위해 UI Thread에서만 UI 자원을 사용할 수 있도록 만들었습니다. 따라서 UI 작업을 UI 스레드 외의 스레드에서 작업해서는 안 됩니다. 

![Untitled (1)](https://user-images.githubusercontent.com/50517813/137631064-2fd56ca0-366a-426d-81e9-bded600b30b0.png)

![Untitled](https://user-images.githubusercontent.com/50517813/137631082-1873f5e1-ef01-4479-bbb2-19c82a901560.png)

따라서 아래 두 가지 규칙을 지켜야 합니다.

1. UI 스레드를 대기시키면 안 됩니다.
2. UI 스레드 외부에서 UI 작업을 수행하면 안 됩니다.

# Thread Access

우리는 안드로이드에서 단일 스레드 모델에서 작업에 따라 다른 스레드를 사용해야함을 배웠다.

UI를 업데이트하기 위한 데이터를 불러오기 위해 별도의 스레드(background 또는 worker thread)에서 작업을 수행하지만, UI를 업데이트하기 위해서는 UI 스레드를 사용해야 한다.

위 문제를 해결하기 위해 Android에서는 다른 스레드에서 UI 스레드에 액세스 하는 여러 가지 방식을 제공한다.

참고 : [https://developer.android.com/guide/components/processes-and-threads?hl=ko](https://developer.android.com/guide/components/processes-and-threads?hl=ko)

## 1. Activity.runOnUiThread(Runnable)

현재 실행하고자 하는 작업을 UI Thread에서 처리하도록 하는 함수입니다. 

```java
/**
 * Runs the specified action on the UI thread. If the current thread is the UI
 * thread, then the action is executed immediately. If the current thread is
 * not the UI thread, the action is posted to the event queue of the UI thread.
 *
 * @param action the action to run on the UI thread
 */

public final void runOnUiThread(Runnable action) {
	if (Thread.currentThread() != mUiThread) {
		mHandler.post(action); 
	} else { 
		action.run(); 
	} 
}
```

실제 Activity.runOnUiThread()의 구현 코드입니다.

만약 현재 Thread가 UI Thread 인 경우 작업을 즉시 처리합니다. UI Thread가 아니라면 작업을 UI Thread의 event queue에 들어갑니다.

### 사용 예시

```kotlin
class MainActivity : AppCompatActivity() {

    private lateinit var binding: ActivityMainBinding

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        thread {
            var i = 0

            while (i < 10) {
                i += 1

                runOnUiThread {
                    binding.counter.text = i.toString()
                }
                Thread.sleep(1000)
            }
        }
    }
}
```

위 예시에서 새로운 thread를 생성해서 counter의 숫자를 증가시키고 counter textView에 text로 적용하는 작업을 합니다.

이때 UI를 업데이트하는 작업은 runOnUiThread로 감싸서 사용하는 것을 확인할 수 있습니다.

참고 : [https://magicalcode.tistory.com/48](https://magicalcode.tistory.com/48)

## 2. View.post(Runnable)

Activity.runOnUiThread와 동일하게, 실행하고자하는 작업을 UI Thread에서 실행하도록 해주는 함수입니다.

```kotlin
/**
 * <p>Causes the Runnable to be added to the message queue.
 * The runnable will be run on the user interface thread.</p>
 *
 * @param action The Runnable that will be executed.
 *
 * @return Returns true if the Runnable was successfully placed in to the
 *         message queue.  Returns false on failure, usually because the
 *         looper processing the message queue is exiting.
 *
 * @see #postDelayed
 * @see #removeCallbacks
 */
public boolean post(Runnable action) {
    final AttachInfo attachInfo = mAttachInfo;
    if (attachInfo != null) {
        return attachInfo.mHandler.post(action);
    }

    // Postpone the runnable until we know on which thread it needs to run.
    // Assume that the runnable will be successfully placed after attach.
    getRunQueue().post(action);
    return tru
}
```

Runnable action을 attachInfo.mHandler에 post하여 UI Thread의 Event Queue에 전달합니다.

```kotlin
/**
 * A set of information given to a view when it is attached to its parent
 * window.
 */
final static class AttachInfo {
		/* ... */
		    
    /**
     * A Handler supplied by a view's {@link android.view.ViewRootImpl}. This
     * handler can be used to pump events in the UI events queue.
     */
    @UnsupportedAppUsage
    final Handler mHandler;
}
```

AttachInfo는 View의 parent window에 연결할 때 View에 제공되는 정보 집합으로 멤버 변수로 Handler를 하나 가지고 있는데, UI 이벤트 대기열에 이벤트를 펌프하는 데 사용할 수 있습니다.

### 사용 예시

```kotlin
class MainActivity : AppCompatActivity() {

    private lateinit var binding: ActivityMainBinding

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        thread {
            var i = 0

            while (i < 10) {
                i += 1

								// 무명 객체로 Runnable 구현
                binding.counter.post(object : Runnable {
                    override fun run() {
                        binding.counter.text = i.toString()
                    }
                })
								
								// 함수형 인터페이스로 람다식 사용 가능
								binding.counter.post { binding.counter.text = i.toString() }
                Thread.sleep(1000)
            }
        }
    }
}
```

참고 : [https://codetravel.tistory.com/16](https://codetravel.tistory.com/16)
