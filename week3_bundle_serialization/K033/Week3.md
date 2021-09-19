# Bundle

## Bundle은 뭘까?

여러가지 타입의 값을 Key-Value 형태로 저장하는 Map 클래스다.

{ [String, data], [String, data] ... }

Android 에서 데이터 전송시 필수적으로 사용되며, Int, Double, String 같이 기본 타입부터, Array, Serializable, Parcelable 구현한 객체까지 전송한다.

Activity나 Fragment간, 즉 같은 프로세스 내에서 데이터 통신을 위해 사용하기도하지만, 프로세스간 통신(IPC)을 위해 Bundle 클래스를 사용한다.

## 왜 사용할까?

### 1. 어플(프로세스 내)

Android에서 Activity 간의 데이터를 주고 받을 때, 혹은 화면 가로/세로 전환 등으로 인해 현재 Activity에서 사용한 데이터를 일시적으로 보관할 필요가 있다. 그런 경우에 Bundle을 사용한다.

Android 에서는 Activity나 Fragment간의 데이터를 주고 받을 때 Bundle 객체를 사용하여 데이터를 전송할 수 있다.

```kotlin
class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        val bundle: Bundle = Bundle()
        bundle.putString("name", "bundle")
        bundle.putInt("age", 1234)
        val array: ArrayList<Int> = arrayListOf(1,2,3,4)
        bundle.putIntegerArrayList("array", array)

        findViewById<Button>(R.id.button).setOnClickListener {
            val intent: Intent = Intent(applicationContext, SubActivity::class.java)
            intent.putExtra("bundle", bundle)
            startActivity(intent)
        }

    }
}

class SubActivity: AppCompatActivity() {
    val TAG = "SubActivity"
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.sub_main)

        val intent = intent
        val bundle = intent.getBundleExtra("bundle")
        bundle?.run {
            Log.d(TAG, "name: ${bundle.get("name")}")
            Log.d(TAG, "age: ${bundle.get("age")}")
            Log.d(TAG, "array: ${bundle.get("array")}")
            Log.d(TAG, "array: ${bundle.getIntegerArrayList("array")}")
        }

    }
}
```

![Untitled](https://user-images.githubusercontent.com/50517813/133932515-cb466f04-088d-44bf-bf74-a4120100267e.png)

- SavedInstanceState

생명주기에 대한 메소드를 Override하여 사용할 때, savedInstanceState라는 이름의 Bundle 객체를 매개변수로 받는 것을 확인할 수 있다.

savedInstanceState는 액티비티를 중단하거나 화면 전환 등을 할 때, onStop() 메소드가 호출된 후에  `onSaveInstanceState()`메서드가 호출된다. `onSaveInstanceState()`를 오버라이드하여 유지하고자 하는 값을 저장할 수 있는데 Bundle 형태로 저장한다.

![Untitled](https://user-images.githubusercontent.com/50517813/133932531-93e27f54-c95e-4f8c-a68d-d67151846118.png)

화면 전환시 생명주기 변화

또한 액티비티를 중단할 때 `onSavedInstanceState()` 메서드를 호출하여 저장하고 싶은 데이터를 Bundle에 저장하고, 다시 생성할 때 `onCreate()` 또는 `onRestoreInstanceState` 에서 데이터를 다시 가져와서 사용 수 있다.

Bundle에 `onSaveInstacneState()`를 활용하여 데이터를 저장할 때 직렬화 과정을 거친다.

즉 Bundle을 사용해서 데이터를 저장하고 필요에 따라 불러와서 사용하거나 전송할 수 있다.

많은 데이터를 전송하면 시스템에서 `TransactionTooLargeException` 예외가 발생할 수 있다

예시) savedInstanceState를 활용하지 않은 앱의 경우

```kotlin
class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        val binding = ActivityMainBinding.inflate(layoutInflater)
        setContentView(binding.root)
        
        var count = 0
        binding.button.setOnClickListener {
            // Count 증가
            ++count
            binding.counter.text = count.toString()
        }
    }
}
```

![Untitled](https://user-images.githubusercontent.com/50517813/133932540-9415867b-527f-498b-9b96-975da024ca0f.png)

![Untitled](https://user-images.githubusercontent.com/50517813/133932548-23889559-87a7-4b09-9a7f-af5662bfacfe.png)

![Untitled](https://user-images.githubusercontent.com/50517813/133932566-25765e09-a5bb-49d1-bfff-649ff4520d29.png)

화면이 가로에서 세로로 전환되면 Activity가 지워지고 다시 생성된다.

기존의 Activity가 삭제되었으므로 Activity에 존재하는 데이터는 사라지게 된다.

예시) savedInstanceState를 사용했을 때

```kotlin
class MainActivity : AppCompatActivity() {
    var count = 0

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        val binding = ActivityMainBinding.inflate(layoutInflater)
        setContentView(binding.root)

        savedInstanceState?.run {
            count = savedInstanceState.getInt("count")
            binding.counter.text = count.toString()
        }

        binding.button.setOnClickListener {
            // Count 증가
            ++count
            binding.counter.text = count.toString()
        }
    }

    override fun onSaveInstanceState(outState: Bundle) {
        super.onSaveInstanceState(outState)
        outState.putInt("count", count)
    }
}
```

![Untitled](https://user-images.githubusercontent.com/50517813/133932548-23889559-87a7-4b09-9a7f-af5662bfacfe.png)

![Untitled (1)](https://user-images.githubusercontent.com/50517813/133932588-048b256c-ff40-49f0-b528-a3b19f6cf591.png)

화면 전환 후에도 기존 count 값이 유지된다.

예시) onRestoreInstanceState 활용

```kotlin
class MainActivity : AppCompatActivity() {
    var count = 0
    private lateinit var binding: ActivityMainBinding

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        binding = ActivityMainBinding.inflate(layoutInflater)
        setContentView(binding.root)

        binding.button.setOnClickListener {
            // Count 증가
            ++count
            binding.counter.text = count.toString()
        }
    }

    override fun onRestoreInstanceState(savedInstanceState: Bundle) {
        super.onRestoreInstanceState(savedInstanceState)
        count = savedInstanceState.getInt("count")
        binding.counter.text = count.toString()
    }

    override fun onSaveInstanceState(outState: Bundle) {
        super.onSaveInstanceState(outState)
        outState.putInt("count", count)
    }
}
```

onRestoreInstanceState()는 onStart() 다음에 호출되는데, savedInstanceState가 null이 아닐때만 호출되기 때문에 null 체크를 할 필요 없다.

### 2. 프로세스(어플리케이션) 간 통신

### IPC(Internal - Process - Communication)

프로세스 간 통신, 프로세스 사이에서 데이터를 서로 주고 받는 행위, 경로, 방법

일반적으로 안드로이드는 IPC에 관한 두 가지 상황이 있다.

- 응용 프로그램 간 통신
- 다중 프로세스 응용프로그램(Activity, Service, Receiver, Provider 등의 구성요소가 서로 다른 프로세스에서 실행되는 응용프로그램)의 프로세스 통신

안드로이드의 IPC 는 AIDL, Messenger, Broadcast 세 가지 방법으로 구현할 수 있다.

### AIDL(안드로이드 인터페이스 정의 언어)

Android에서는 일반적으로 한 프로세스가 다른 프로세스의 메모리에 액세스할 수 없다.

따라서 객체들을 운영체제가 이해할 수 있는 Primitive한 타입으로 분해하고 마샬링해야한다.

위 작업을 AIDL로 대신 처리할 수 있다.

AIDL은 다중 IPC 구조가 필요한 경우에 사용되며 동시 작업이 필요없는 경우에는 Messenger를 사용하는 것이 좋다.

### ViewModel의 사용

ViewModel을 사용하여 상태를 저장할 수 있다. Bundle은 대량의 데이터를 저장할 수 없으며 직렬화가능한 데이터만 저장할 수 있지만 ViewModel은 상관없다. 게다가 Activity의 생명주기보다 더 길게 유지되기 때문에 어느 생명주기에서든 ViewModel에 데이터를 저장하고 호출할 수 있다.

하지만 ViewModel은 앱이 정상적으로 종료되지 않았을 때는 새로 생성되며 기존의 데이터를 잃게된다. ← ViewModel이 메모리 내에 위치하기 때문에 프로세스가 종료되면 인스턴스가 사라짐.

savedInstanceState는 앱이 프로세스에 의해 중단된 경우에도 데이터를 저장하고 있다.

# 객체 직렬화

### 직렬화

메모리 내에 존재하는 정보(객체)를 보다 쉽게 전송 및 전달하기 위해 byte 코드 형태로 나열하는 것

안드로이드에서 액티비티간 데이터를 전달하는데 사용하는 Bundle 객체는 직렬화가 가능한 데이터만 전송할 수 있다.

Primitive한 데이터들은 직렬화가 자동으로 가능하지만, 클래스의 객체같은 경우에는 직렬화를 가능하게끔 만들어줘야한다. 우린 클래스를 직렬화 가능하게 만들기 위해, Serializable 또는 Parcelable을 사용한다.

### 역직렬화

byte 형태의 Data르 원래대로 Object나 Data로 변환하는 기술

## Serializable

객체를 쉽게 직렬화하기 위한 자바의 인터페이스

### 장점

단순히 implement 하는 것만으로도 직렬화가 가능하다는 것을 알려주기 때문에 사용하기 매우 쉽다.

해당 클래스가 직렬화 가능 대상인 것을 알려주기만 할 뿐 어떤 메소드도 가지고 있지 않는 단순한 마커 인터페이스이다.

### 단점

사용방법이 간단한 대신 시스템적인 비용(성능 저하, 배터리 소모 등)이 비싸다.

Reflection을 사용하여 직렬화 처리를 하는데, Reflection은 프로세서 동작 중에 사용되며 처리 과정 중에 많은 추가 객체를 생성한다. 생성된 객체들은 가비지 컬렉터에의해 수집되는데, 가비지 컬렉터의 과한 동작을 유발한다.

## Parcelable

Parcel에 쓰고 복원할 수 있는 클래스의 인스턴스를 위한 Android SDK의 인터페이스

사용자가 직렬화 처리 방법을 명시적으로 작성한다.

### 장점

Serializable 사용에서 시스템 비용을 높이는 이유였던 Reflection을 굳이 사용하지 않아도 된다.

### 단점

필수로 구현해야하는 메소드를 포함하기 때문에 클래스가 복잡해지고 유지 보수가 어려워질 수 있다.

### 필수 구현 코드

- `decribeContent()`
    - Parcel의 내용을 기술한다. FileDescriptor 같은 특별한 객체가 들어가면 이 부분을 통해서 알려줘야 한다.
- `writeToParcel(Parcel out, int flag)`
    - Parcel 안에 데이터를 넣는 작업을 한다.

### CREATOR

Parcelable을 implement할 때 위 두 메소드와 함께 CREATOR라는 static field를 반드시 가지고 있어야 한다. `Parcelable.Creator<T>` 로 선언하는 순간 초기화되어야한다.

이 객체는 나중에 Parcel로 unmmarshalling 할 때 사용되어진다.

이 필드를 사용하기 위해 두 가지 메소드를 추가로 정의해야한다.

- `createFromParcel(Parcel source)`
    - parcel된 데이터를 다시 원래대로 만들어 주는 작업을 한다.
- `newArray(int size)`
    - Parcel.createTypeArray()를 호출했을 때 불리며 new T[size] 로 반환해준다.

## 성능 차이

Serializable과 Parcelable 간의 여러 차이점이 있지만 간략하게 속도에 관한 성능 차이에 대해 작성한다. 현재 성능 차이에 대한 여러 가지 의견이 존재한다.

1. Parcelable이 Serializable 보다 빠르다.
2. 
![Untitled](https://user-images.githubusercontent.com/50517813/133932622-6545270f-8aa0-4488-8f6c-74250544d0b8.png)

위 결과를 보면 Parcelable이 10배 이상 빠르다는 것을 알 수 있다. 하지만 다른 의견도 존재한다.

1. '기본 사용법에 의한 Serializable'은 Parcelable 보다 느리다.

위 말을 조금 달리 말해보자면, Serializable을 다른 방식으로 사용한다면, Parcelable보다 빠를수도 있다는 말이다.

현재 우리가  '기본적으로 사용하는 Serializable'은 Java의 자동 직렬화 프로세스를 사용하고 있다. 이 부분을 클라이언트가 개선한다면, Parcelable보다 더 빠르게 작동할 수 있다.

아래 메소드를 구현하면 Serializable에서 자동으로 처리되는 직렬화 프로세스에 대체할 수 있다.

마치 Parcelable을 사용하는 것 처럼 writeObject와 readObject 메소드에 직렬화하려는 클래스에 맞는 로직을 포함한다면, Serializable에 의해 발생하는 가비지를 생성하지 않게 할 수 있다.

```jsx
private void writeObject(java.io.ObjectOutputStream out)
     throws IOException;
     
 private void readObject(java.io.ObjectInputStream in)
     throws IOException, ClassNotFoundException;

 private void readObjectNoData()
     throws ObjectStreamException;
```

위 메소드들을 구현하고 Parcelable과 비교해보면, Serializable이 쓰기에서 3배, 읽기에서 1.6배 정도 빠르다.

아래 사이트에서 Serializable을 구현하고 테스트해볼 수 있다.

[https://github.com/GMLim/Android-Serialization-Test](https://github.com/GMLim/Android-Serialization-Test)
