# Android UI Thread

안드로이드 프로그램은 다른 프로그램과 마찬가지로 `Multi Thread`를 지원한다. 하지만 잘 알고 있듯이 UI 수정 만큼은 `Main Thread` 에서 수정하도록 강제하고 있다.

잘 알고 있듯이 서로 다른 스레드에서 같은 자원을 접근하는 경우에 `동시성(동시 자원 접근)문제`가 발생할 수 있다.
<img src="https://user-images.githubusercontent.com/71161576/137662088-8d02b1b0-012d-47c1-a48d-d6e3a60a8c37.png" width="600" />

출처 : [https://kajalrawal.medium.com/concurrentmodificationexception-in-java-756afe64dfb7](https://kajalrawal.medium.com/concurrentmodificationexception-in-java-756afe64dfb7)

안드로이드는 결국 UI 프로그래밍이고 UI 관련 작업에서 문제가 발생하는 것은 치명적이기 때문에, UI 작업은 메인 스레드에서 진행하도록 강제한다. 

이처럼 메인 스레드에서만 UI 작업을 진행하도록 강제하기 때문에 안드로이드에서는 메인 스레드를 UI 스레드라고 부르기도 한다.

안드로이드의 이러한 UI 싱글 스레드 모델의 규칙은 첫째, **UI 관련 작업은 오직 UI 스레드에서만 수행 할 수 있도록 할 것** , 둘째, **메인 스레드(UI 스레드)를 block하지 말 것**, 이 두 가지다.

두번째의 경우를 살펴본다면 다음과 같은 화면을 떠올리면 된다.

<img src="https://user-images.githubusercontent.com/71161576/137662256-6907ffd9-1ff3-40ce-b001-17249a16c180.png" width="300" />

안드로이드 어플을 사용하다가 간혹 이러한 화면을 마주한 적이 있을 것이다. 이는 바로 메인 스레드가 일정 시간보다 오래 block 되는 경우(현재 약 5초) 발생하는 **ANR(Application Not Responding)**이다.

앱쓰다가 만나면 진짜 정 떨어지는 건 다들 경험해봤을 것이다. 그렇기 때문에 절대 발생시키지 않아야하는 문제이다.

ANR이 발생하는 것을 막기 위해서는 오래 걸릴 것으로 예상되는 작업이나, 언제 종료될 지 모르는 작업은 메인 스레드에서 수행하면 안된다.

그래서 데이터베이스 접근 및 서버 통신 같은 작업은 메인 스레드가 아닌 `다른 스레드`에서 진행하고, 이 결과를 다시 메인 스레드에서 화면에 반영해야한다.

위에서 안드로이드가 왜 UI 작업을 메인 스레드에서 처리하도록 강제하는지, 그리고 이러한 이유에서 작업에 따라서 스레드를 분기해서 처리해야하는 이유를 알아보았다.

실제로 안드로이드 프로그래밍에서 위와 같은 상황들 때문에 스레드 간의 분기를 통해 프로그램이 진행이 된다. 그리고 다시 작업의 결과를 특정 스레드에서 처리하도록(UI 반영) 해야하는데, 이때는 당연히 스레드 간의 통신이 필요하게 된다. 이때 Looper 와 Handler 가 이를 가능하게 한다.

# Looper, MessageQueue, Handler

## Looper

Looper 에 대해서는 대충 어떻게 동작하는 지 정도 알고 있었다. 계속 loop 를 돌면서 message queue에 존재하는 message들을 실행하는 역할을 한다 정도? 

<img src="https://user-images.githubusercontent.com/71161576/137662353-3e0ccf3a-829e-4bb7-a423-5e74aac21a13.png" width="600" />

우선 Looper는 하나의 스레드에 존재하고, 하나의 스레드를 담당할 수 있다. 하지만 우리가 새로운 스레드를 생성하면 Looper는 자동으로 생성되지 않는다. 

Looper 클래스는 `android.os`패키지에 존재하는 클래스이다. 그렇기 때문에 자바의 Thread를 생성해도 뭐 자동으로 생성해주지는 않는다. 하지만 메인 스레드는 내부적으로 Looper를 가지도록 구현되어있다.

공식문서에 나와있는대로 스레드 내부에서 `prepare()` 로 looper를 생성할 수 있고, `loop()` 를 통해서 message queue가 빌 때까지 message 내용을 처리한다.

<img src="https://user-images.githubusercontent.com/71161576/137662433-fef673df-2972-4371-8e10-310003f26cb6.png" width="600" />

<img src="https://user-images.githubusercontent.com/71161576/137662446-c6ad3eb9-8f8e-446c-99ee-d441b138d772.png" width="600" />

그리고 위에서 언급한대로 looper는 하나의 스레드에서 하나의 `message queue`와 연결이 된다. 이렇게 메인 스레드의 looper 와 message queue를 사용하여 UI 처리에서의 동시성 문제를 해결한다.

looper는 생성되어 TLS(Thread Local Storage)에 저장된다. 

## MessageQueue

<img src="https://user-images.githubusercontent.com/71161576/137663075-e55abbae-265e-4f05-9f1f-0be478d3a58a.png" width="600" />

`Message queue는` 다른 스레드나 혹은 자기 자신으로부터 전달받은 Message를 기본적으로 선입선출 형식으로 보관하는 Queue 이다. 

looper는 message queue에서 Message나 Runnable 객체를 차례로 꺼내 Handler가 이를 처리하도록 전달한다. 

## Handler

<img src="https://user-images.githubusercontent.com/71161576/137662582-825954eb-7d0d-4ae1-af12-a5d70d6cd96c.png" width="600" />

Handler는 스레드의 Message Queue와 연계하여 `Message`나 `Runnable` 객체를 받거나 처리하여 스레드 간의 통신을 할 수 있도록 한다. Handler 객체는 하나의 스레드와, 해당 스레드의 Message Queue에 바인드 된다.  

공식문서에서 나와있듯이 Handler는 MessageQueue에 저장된 내용을 실행하거나, 다른 스레드에서 전달한 동작을 해당 스레드의 messageQueue에 등록할 수 있다.

MessageQueue에 저장된 내용을 처리하기 위해서는 `handleMessage()` 를 사용할 수 있다.

<img src="https://user-images.githubusercontent.com/71161576/137662699-d016a7ad-ad9e-469b-b371-e4b9edbc61c7.png" width="600" />

그리고 실행할 동작을 MessageQueue 에 등록하기 위해서는 `post()` , `sendMessage()`를 사용할 수 있다.

<img src="https://user-images.githubusercontent.com/71161576/137662745-7edd0de2-e1cd-4146-ba98-8e1fcce55eba.png" width="600" />

<img src="https://user-images.githubusercontent.com/71161576/137662777-f673785c-7d89-40b0-9e0d-f5dfb85d0562.png" width="600" />

`Looper`, `MessageQueue`, `Handler` 의 동작을 도식화 하면 다음과 같이 표현할 수 있다.

<img src="https://user-images.githubusercontent.com/71161576/137662809-a0173305-9f8f-4a20-94d6-1718ce6574e3.png" width="600" />

출처 : [https://academy.realm.io/kr/posts/android-thread-looper-handler/](https://academy.realm.io/kr/posts/android-thread-looper-handler/)

그렇다면 실제 코드 상에서 위의 동작을 한번 확인해보자.

간단한 예시를 만들어 보았다.

```kotlin
class MainActivity : AppCompatActivity() {
    private val TAG = "MainActivity"
    private val mainHandler: Handler by lazy {
        Handler(mainLooper)
    }
		...

    override fun onCreate(savedInstanceState: Bundle?) {
        ...
        val workerThread = MyThread(mainHandler)
        workerThread.start()
    }
}
```

```kotlin
class MyThread(private val mainHandler: Handler) : Thread() {

    override fun run() {
        super.run()
        Log.i("Thread name", "${Thread.currentThread().name}")
        Log.i("TAG", "do something in worker thread")

        mainHandler.post {
            Log.i("Thread name", "${Thread.currentThread().name}")
            Log.i("TAG", "do something in main thread")
        }
    }
}
```

우선 메인 스레드에는 자동으로 looper 와 message queue가 존재한다. 그리고 이를 기반으로 메인 스레드에서 동작하는 main handler를 생성했다. 

이 핸들러는 메인 스레드의 message queue의 내용을 looper가 돌면서 전달해주고, 이를 수행하게 된다. 

MyThread 는 메인 스레드가 아닌 별도의 스레드인데, 여기에서는 UI와 관련 없는 작업을 수행하고 결과를 UI에 반영하기 위해서 main handler를 통해 작업을 message queue에 등록할 수 있다.

위의 예제는 post()를 통해서 message queue에 메인 스레드에서 처리할 작업을 등록한 것이다.

**그리고 앞서 알아본 것처럼 handler의 sendMessage로 수행할 작업을 message queue에 등록할 수도 있다.**

message로 등록하는 정보는 Message 객체를 사용한다.

<img src="https://user-images.githubusercontent.com/71161576/137662892-3a7be66e-a03d-476d-a290-9f6c787e23d8.png" width="600" />

`Message`는 스레드 간 통신할 내용을 담는 객체이자 message queue에 들어갈 처리할 작업의 단위이다. 일반적으로 Message가 필요할 때 Message() 로 새 Message 객체를 생성하면 성능 이슈가 생길 수 있으므로 안드로이드가 시스템에 만들어 둔 Message Pool의 객체를 재사용한다. obtain() 메서드로 메시지 풀에 미리 생성된 Message 객체를 얻을 수 있다. 

이렇게 sendMessage() 로 메시지를 등록하면 다시 handle에서는 `handleMessage()` 를 통해서 해당 메시지에 대한 특정 처리를 하도록 구현할 수 있다.

```kotlin
class MyThread(private val mainHandler: Handler) : Thread() {

    override fun run() {
        super.run()
        Log.i("Thread name", "${Thread.currentThread().name}")
        Log.i("TAG", "do something in worker thread")

        val message = Message.obtain()
        message.arg1 = 1
        mainHandler.sendMessage(message)
    }
}
```

```kotlin
class MyHandler(looper: Looper) : Handler(looper) {

    override fun handleMessage(msg: Message) {
        super.handleMessage(msg)

        if (msg.arg1 == 1) {
            Log.i("MyHandler", "${Thread.currentThread().name}")
            Log.i("MyHandler", "handleMessage arg == 1")
        }
        else {
            throw IllegalArgumentException()
        }
    }
}
```

이 경우에는 메인 스레드의 handler가 message 의 정보를 기반으로 특정 처리를 할 수 있다.

<img src="https://user-images.githubusercontent.com/71161576/137662917-ef3733e0-cef7-4098-9b52-742a2f317372.png" width="600" />

추가적으로 Handler 의 기본 생성자가 deprecated 되었다. 그 대신에 Looper를 명시해주어야하는데, 해당 Looper 의 스레드를 따르는 Handler를 생성할 수 있게 되었다.

```kotlin
class MyThread : Thread() {

    override fun run() {
        super.run()

        Looper.prepare()
        // create a handler on main thread
        val mainHandler = Handler(Looper.getMainLooper())

        // create a handler on worker thread
        val handler = Handler() // deprecated

        val work = Runnable {
            Log.i("Thread name", "${currentThread().name}")
            Log.i("TAG", "run: do something anything")
        }
        mainHandler.post(work)
        handler.post(work)
        Looper.loop()
    }
}
```

<img src="https://user-images.githubusercontent.com/71161576/137662959-27db49d5-40d9-461d-9e5f-3c9aef160b6a.png" width="600" />

Handler의 기본생성자는 deprecated 되었는데, 기존의 기본 생성자로 생성한 Handler 객체는 생성자 호출 당시의 스레드를 따라가는 Handler 였다. 

하지만 변경된 생성자를 사용하면 Looper를 지정할 수 있고, Handler는 해당 Looper의 스레드에서 동작하게 된다.

앞서 살펴봤듯이 스레드는 기본적으로 Looper를 가지고 있지 않기 때문에, 기존의 기본 생성자로 Handler 를 생성하면 Looper의 존재 여부를 확인할 수 없었고 이로 인한 문제점들이 존재하였다.

그래서 지금은 Handler의 생성자로 바인딩 될 Looper 객체를 넘겨주도록 변경이 된 것 같다.

## runOnUiThread

지금까지 안드로이드에서 다른 스레드간의 통신을 위해서 `Looper`, `MessageQueue`, `Handler` 가 동작하는 방식에 대해서 알아보았다.

그리고 처리할 작업을 다른 스레드에 위임하기 위해서 Handler의 post() 와 sendMessage() 를 사용하는 것도 알아보았다.

post() 방식과 sendMessage()의 사용에 대해서 간단히 다시 비교해보자면, Message 객체를 전송하는 방식의 경우에는 Message 객체를 생성해서 보내고, 추가적으로 handleMessage() 를 Handler 가 구현해야 한다는 번거로움이 있었다. 하지만 post() 를 사용하면 Runnable 객체 내부에서 바로 run() 메서드를 오버라이딩 해서 보내면 되기 때문에 다소 편리하게 정보를 전달할 수 있다. 

여기서 한번 생각해볼 부분이 있다. 우리는 보통 메인 스레드에서의 작업 부하를 줄이기 위해서 다른 스레드에 작업을 위임하고 다시 UI 갱신 작업을 메인 스레드에 위임한다. 

그렇기 때문에 항상 메인스레드에서 Handler 객체를 생성하고, 다른 워커 스레드에서 작업을 한 뒤 결과인 Runnable 객체를 메인 스레드의 Handler 객체의 post 메서드를 통해서 반영한다. 

근데 이렇게 할 필요가 있을까? 어차피 메인 스레드에서 처리하는거면 이미 메인 스레드에 존재하는 Handler를 사용하면 될 것이다.

그리고 Runnable 객체는 다른 스레드에서 MessageQueue에 추가할 수도 있지만, 현재 스레드에서 추가하는 경우도 있을 수 있다.

이때는 post로 messageQueue에 등록하는게 아니라 바로 run() 을 실행하면 될 것이다. 

이러한 아쉬운 부분을 채워주는 것이 `runOnUiThread()` 메서드이다.

<img src="https://user-images.githubusercontent.com/71161576/137662988-f4e6e0c9-7538-4e42-b0d9-a09bc50a036d.png" width="600" />

runOnUiThread는 Activity의 메서드로, 현재 스레드가 메인 스레드인 경우에는 Runnable 객체의 run() 메서드를 즉시 실행하고, 현재 스레드가 메인 스레드가 아닌 경우에는 메인 스레드의 MessageQueue에 Runnable 객체를 post한다.