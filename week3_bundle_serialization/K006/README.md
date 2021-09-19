# Bundle


## 1. What is a bundle?

Bundle 이란 단어의 뜻은 한국말로 '묶음' 으로 표현되는데 실생활의 맥락에서는 다음과 같은 의미로 사용이 되기도 한다.

> Bundle은 묶음이라는 뜻이며 보통 상품을 구매했을 때 끼워주는 것을 말한다. 예를들어 스마트폰을 구입했을 때 나누어주는 이어폰이나 PC를 구입할 때 끼워주는 소프트웨어 등이다.

안드로이드에서 번들 또한 비슷한 의미로 사용되는 것 같다. 결국 번들이란 데이터의 묶음을 의미하는데 안드로이드에서 데이터 전달을 위해서 일련의 데이터를 번들로 구성하고 이를 주고 받는 형식으로 데이터 전달을 진행한다.

하나의 간단한 예로 액티비티나 프래그먼트의 생명주기 콜백에서 번들 객체를 사용하는 것을 볼 수 있다. 이는 상태 저장을 위해서 사용하는데, 이때 필요한 데이터 묶음을 번들로 구성하고 이를 생명주기마다 끼워서(?) 저장하고 불러와서 사용할 수 있게 된다.   

## 2. simple usage

안드로이드에서 번들은 Bundle 클래스를 통해서 생성할 수 있다. 안드로이드 번들에 대해서 검색해보면 대부분 다음과 같은 정의를 내린다.

> Bundle은 **여러가지의 타입의 값을 저장하는 Map 클래스**이다.

번들은 내부적으로 **`key - value`** 자료구조로 구성이 되어 있다. Map 과의 차이점이라면 번들은 **여러가지 타입**의 데이터를 저장할 수 있다는 것이다.

번들의 사용법은 간단하다.

```kotlin
// android.os.Bundle 객체 생성
val bundle = Bundle() 

// bundle 에 데이터 저장
bundle.putString("name", "jjinse")
bundle.putInt("age", 99)
bundle.putBoolean("isHungry", true)
bundle.putStringArray("arrayOfSting", arrayOf("A","B","C"))
... 

// bundle 의 데이터 조회
bundle.getString("name")
bundle.getInt("age")
bundle.putBoolean("isHungry")
bundle.putStringArray("arrayOfSting")
```

번들 객체를 생성하고 **`bundle.put...(key, value)`** 의 형태로 저장하고 싶은 데이터를 key-value 형태로 저장하면 된다.

map 자료구조와는 다르게 하나의 번들로 다양한 타입의 데이터를 저장할 수 있다.

그리고 key 값을 통해서 번들에 저장한 데이터를 조회할 수 있다. 

그리고 **`bundleOf( )`** 헬퍼 메서드를 통해서도 편리하게 번들 객체를 생성할 수 있다.

```kotlin
val bundle2 = bundleOf("age" to 25, "name" to "sejin", "isHungry" to true, ...)
```

## 3. When do we use the bundle?

그렇다면 언제 번들을 사용할까? 

#### 액티비티 간의 데이터 전달

가장 흔한 예제는 액티비티 간의 데이터 전송이다. 액티비티는 Intent를 통해서 서로 통신을 하게된다. 하나의 액티비티에서 통신을 위해 Intent를 생성하고, 이 안에 번들을 담을 수 있다. 

액티비티는 하나의 앱에서만 데이터를 공유하기도 하지만, 다른 앱 또는 다른 프로세스들과도 데이터를 공유하기 때문에 번들을 인텐트에 담고 이를 통해 통신을 한다. 프로세스간의 통신은 기본적으로 바이트 스트림으로 이루어지기 때문에 번들의 데이터는 primitive 타입이거나 직렬화된 타입만 담을 수 있는 것이다.

다음과 같이 하나의 액티비티에서 intent에 번들을 담아서 다른 액티비티에 번들의 내용을 전달할 수 있다.

```kotlin
// MainActivity

// Bundle
val bundle = Bundle()
bundle.putInt("age", 99)
bundle.putString("name", "sejin")
bundle.putBoolean("isHungry", true)
bundle.putStringArray("stringArray", arrayOf("A","B","C"))

// we can put bundle in the intent
button.setOnClickListener {
	val myIntent = Intent(applicationContext, SecondActivity::class.java)
	myIntent.putExtra("bundle", bundle)
	startActivity(myIntent)
}
```

```kotlin
// SecondActivity

val name = intent.getBundleExtra("bundle")
val textView = findViewById<TextView>(R.id.textView_second)
textView.text = "${name?.getString("name")} ${name?.getInt("age")} ${name?.getBoolean("isHungry")}"
```

#### 프래그먼트 간의 데이터 전달

다음으로는 프래그먼트 간의 데이터 전달에도 번들을 사용할 수 있다.  프래그먼트는 **FragmentManager**를 통해서 관리되기 때문에 이를 매개로 하여 데이터를 전달할 수 있다.

<img src="https://user-images.githubusercontent.com/71161576/133922414-0feb0c28-e619-48d0-bc01-7d439001e7c0.png" width="600" />

출처 : [https://developer.android.com/training/basics/fragments/pass-data-between?hl=ko](https://developer.android.com/training/basics/fragments/pass-data-between?hl=ko)

데이터를 수신하는 프래그먼트에서는 **`setFragmentResultListener()`**를 통해 해당 데이터에 대한 리스너를 설정한다. 리스너의 콜백에는 번들 객체가 인자로 들어오고 콜백을 구현하면 된다.

그리고 데이터를 전송한느 프래그먼트에서는 **`setFragmentResult`**로 전송할 결과 데이터를 key 값으로 등록하고, 여기에 전송할 번들을 담는다.

다음 예제는 HomeFragment 에서 DashboardFragment로 번들 데이터를 전송하는 과정이다.

```kotlin
// DashboardFragment

override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setFragmentResultListener("requestKey") { key, bundle ->
            result = bundle.getString("bundleKey")
            Log.i("Dashboard Fragment", "received bundle data by Home Fragment : $result ")
        }
        Log.i("Dashboard Fragment", "onCreate")
    }
```

```kotlin
// HomeFragment

override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setFragmentResult("requestKey", bundleOf("bundleKey" to "Hello Dashboard! I'm Home"))
        Log.i("Home Fragment", "onCreate")
    }
```

수신하는 DashbardFragment는 결과 데이터(번들)을 수신하고, 이에 대한 리스너 콜백은 onStart() 가 호출되고 나서 수행된다.(`START` 상태)

<img src="https://s3.us-west-2.amazonaws.com/secure.notion-static.com/34776298-e247-424c-b37f-8c730e493c34/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAT73L2G45O3KS52Y5%2F20210919%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20210919T092946Z&X-Amz-Expires=86400&X-Amz-Signature=81209d75fe08c27c5b2365e98f7da2df8f36baeee62fac9d1fad1aa32f3c7b36&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22Untitled.png%22" width="600" />

이는 마찬가지로 상위 / 하위 프래그먼트 간의 데이터 전송에도 동일하게 적용이 된다.

다만 상위 프래그먼트는 **ChildFragmentManager**를 통해서 리스너를 등록해야한다.

<img src="https://user-images.githubusercontent.com/71161576/133922521-616b6f78-c64a-4dc1-bf29-d5631efed861.png" width="400" />
출처 : [https://developer.android.com/training/basics/fragments/pass-data-between?hl=ko](https://developer.android.com/training/basics/fragments/pass-data-between?hl=ko)

프래그먼트 간에 **`setFragmentResultListener`  `setFragmentResult`**    를 통해 데이터를 전송하기 위해서는 **fragment-ktx** 모듈이 필요하기 때문에 별도로 의존성을 추가해주어야한다.

### onSaveInstanceState & onRestoreInstanceState (feat. savedInstanceState)

액티비티의 콜백 함수들을 보면 **`onSaveInstanceState`** 와 **`onRestoreInstanceState`** 라는 것들을 확인할 수 있다. 메서드 이름에서 유추할 수 있듯이 이는 액티비티의 상태를 저장하고 불러오는 기능을 수행한다.

액티비티가 화면 회전 등에 의해서 종료되는 경우 이 두 메서드를 통해서 상태를 저장할 수 있는데, 이때 액티비티의 상태가 번들 객체 형태로 저장이 된다.

화면 회전이 발생할 때마다 액티비티는 Destroy → Create 를 반복하게 된다. 그럴 때마다 해당 콜백 함수를 통해서 상태를 저장하고 불러와서 이전의 상태를 유지할 수 있다.

```kotlin
private var count = 0

...

override fun onSaveInstanceState(outState: Bundle) {
    super.onSaveInstanceState(outState)
    count ++
    outState.putInt("count", count)
    Log.i(TAG, "onSaveInstanceState() 호출")
}

override fun onRestoreInstanceState(savedInstanceState: Bundle) {
    super.onRestoreInstanceState(savedInstanceState)
    Log.i(TAG, "onRestoreInstanceState() 호출")
    savedInstanceState?.let {
        count = it.getInt("count")
        Log.i(TAG, "savedInstanceState in onCreate : $count")
    }
}
```

<img src="https://user-images.githubusercontent.com/71161576/133922554-b9e117dc-bf81-435c-9fb4-b3227958c56e.png" width="800">

**onSaveInstanceState** 콜백은 액티비티가 소멸(**onDestroy**)하기 전, **onRestoreInstanceState**는 액티비티가 시작되고(**onStart**) 나서 수행되는 것을 알 수 있다. 

그리고 액티비티의 생명주기 콜백 중 **onCreate**의 인자로 `**savedInstanceState: Bundle?** 이 들어오는 것을 알 수 있다. 마찬가지로 여기서도 **onSaveInstanceState**에서 설정해주는 번들 데이터를 받아올 수 있다.

그렇게 생각하면 **onRestoreInstanceState** 에서 받아오는 번들 데이터와  **onCreate**에서 받아오는 번들 데이터는 동일한 것을 알 수 있다. 공식 문서에서도 이는 동일하다고 나와있고, **onCreate** 콜백에서 필요한 부분을 사용하고 남은 부분은 **onRestoreInstanceState** 콜백에서 마저 처리하면 된다고 나와있다.

```kotlin
// This callback is called only when there is a saved instance that is previously saved by using
// onSaveInstanceState(). We restore some state in onCreate(), while we can optionally restore
// other state here, possibly usable after onStart() has completed.
// The savedInstanceState Bundle is same as the one used in onCreate().
override fun onRestoreInstanceState(savedInstanceState: Bundle?) {
    textView.text = savedInstanceState?.getString(TEXT_VIEW_KEY)
}

/* 
https://developer.android.com/guide/components/activities/activity-lifecycle?hl=ko
*/
```

# 직렬화 - Serialization

---

## what is a serialization?

우리의 프로그램은 정말 다양한 데이터 타입이 존재한다. 정수형, 실수형, 불리언, 문자열, 그리고 객체지향 프로그래밍에서는 우리가 생성하는 클래스 인터페이스 모두 데이터 타입이 될 수 있다. 하지만 컴퓨터 상에서 이 모든 데이터들은 결국 0 과 1의 숫자로 표현이 되고, 프로그램 상에서는 조금 더 큰 단위인 바이트 단위로 구성이 된다.  ( 물론 프로그램 상에서도 비트 연산은 가능하지만 데이터 타입의 가장 작은 단위는 바이트 단위가 된다.)

결국 이 말은 프로그램에서 모든 데이터는 바이트 단위로 표현이 가능하고, 모든 데이터는 바이트 단위로 관리가 가능하다는 것을 의미한다. ( 파일 입출력의 예를 떠올려보자. 모든 데이터를 바이트로 변환해서 입출력을 수행한다.)

프로그램에서 기본형 타입(primitive)에 해당하는 데이터들은 알아서 바이트 단위로 변환이 된다. 코드 상에서 우리가 명시적으로 바이트 단위로 변환을 할 수도 있다.  하지만 우리가 만들거나 라이브러리에 존재하는 클래스와 그 객체들을 바이트 단위로 변환하는 방법은 자동으로 생성되지 않는다. 하지만 객체들 또한 전송, 저장 등의 이유로 인해서 바이트 단위로 변환해야할 필요성이 있을 것이다. 이러한 동작을 수행하는 것이 **직렬화**의 개념이다.

1. **직렬화 - serialization**

    직렬화는 객체를 다루기 쉬운 바이트 단위로 변환하는 것을 의미한다.

2. **역직렬화 - deserialization**

    역직렬화는 직렬화 된 바이트 정보를 다시 객체의 형태로 변환하는 것을 의미한다.

<img src="https://user-images.githubusercontent.com/71161576/133922589-fdf4201e-0678-4af2-9b5d-fd9ed9502c96.png" width="600">

출처 :  [https://hazelcast.com/glossary/serialization/](https://hazelcast.com/glossary/serialization/)

앞서 살펴본 Bundle에서도 직렬화된 데이터 형태를 다룰 수 있는 메서드가 존재한다.

```jsx
// android.os.Bundle

public void putSerializable (String key, Serializable value)

public void putParcelable (String key, Parcelable value)
```

이러한 이유로 번들에서 Any 타입으로 모든 객체를 그 클래스 타입으로 다루는 메서드 대신, 직렬화 된 데이터를 다루는 메서드를 만든 것으로 생각이 된다.

## How can we serialize it?

안드로이드에서는 직렬화를 하는 여러가지 방법이 존재한다. 

1. **Serializable 인터페이스**

Serializable는 표준 Java 의 인터페이스로, 이 인터페이스를 구현한 클래스의 객체는 직렬화가 가능하다는 것을 의미한다. ( 인터페이스 구현만 하면 끝이다.)

Serializable 은 해당클래스가 직렬화 대상이라고 알려주기만 할 뿐 어떠한 메서드도 가지지 않는 Marker Interface 이다. 그렇기 때문에 직렬화 방법(?) 은 단순히 인터페이스를 implement하는 것으로 끝이다. 세상에 공짜로 얻어지는 것은 없다. 사용 방법이 쉽기 때문에 시스템 관점에서의 비용이 높다. Serializable 인터페이스로 직렬화 / 역직렬화를 진행하게 되면 **리플렉션**을 사용하게 된다. 리플렉션은 동작 중에 많은 추가 객체를 생성하고, 가비지 컬렉션을 일으키기 때문에 성능 저하 및 배터리 소모가 발생한다고 한다.

```kotlin
data class Data(val name: String, val age: Int) : Serializable
```

```kotlin
override fun onSaveInstanceState(outState: Bundle) {
    super.onSaveInstanceState(outState)
		val userData = Data("jjinse", 99)
    outState.putSerializable("userData", userData)
    Log.i(TAG, "onSaveInstanceState() 호출")
}

override fun onRestoreInstanceState(savedInstanceState: Bundle) {
    super.onRestoreInstanceState(savedInstanceState)
    Log.i(TAG, "onRestoreInstanceState() 호출")
    savedInstanceState?.let {
        val userData = it.getSerializable("userData")
        userData as Data
        Log.i(TAG, "savedInstanceState in onRestoreInstanceState : $userData")
    }
}
```

클래스가 **Serializable 인터페이스만 구현**해주면, 직렬화가 가능하다. 

1. **Parcelable 인터페이스**

Parcelable 인터페이스는 Android SDK의 인터페이스로, 이 방식도 마찬가지로 직렬화 하고자 하는 클래스가 해당 인터페이스를 구현하는 방식으로 동작한다. 

Parcelable 인터페이스 방식은 직렬화 방식을 코드상에서 명시하는 데, 이는 Serializable 방식과 가장 큰 차이점이다. Serializable 방식으로 직렬화를 자동 처리하기 위해서 리플렉션을 사용했지만, Parcelable은 직렬화 방식을 직접 명시하기 때문에 내부적으로 리플렉션이 사용되지 않는다. 

하지만 직렬화 방식을 명시해주기 위해 클래스 내부에 보일러 플레이트 코드가 생기게 된다.

```kotlin
data class Data(val name: String?, val age: Int) : Parcelable {
    constructor(parcel: Parcel) : this(
        parcel.readString(),
        parcel.readInt()
    )

    override fun writeToParcel(parcel: Parcel, flags: Int) {
        parcel.writeString(name)
        parcel.writeInt(age)
    }

    override fun describeContents(): Int {
        return 0
    }

    companion object CREATOR : Parcelable.Creator<Data> {
        override fun createFromParcel(parcel: Parcel): Data {
            return Data(parcel)
        }

        override fun newArray(size: Int): Array<Data?> {
            return arrayOfNulls(size)
        }
    }
}
```

```kotlin
override fun onSaveInstanceState(outState: Bundle) {
    super.onSaveInstanceState(outState)
		val userData = Data("jjinse", 99)
		outState.putParcelable("userData", userData)
    Log.i(TAG, "onSaveInstanceState() 호출")
}

override fun onRestoreInstanceState(savedInstanceState: Bundle) {
    super.onRestoreInstanceState(savedInstanceState)
    Log.i(TAG, "onRestoreInstanceState() 호출")
    savedInstanceState?.let {
        val userData = it.getParcelable<Data>("userData")
        Log.i(TAG, "savedInstanceState in onRestoreInstanceState : $userData")
    }
}
```

Parcelable 인터페이스를 구현하게 되면 직렬화 방식을 직접 명시해 줄 수 있는 반면에, 직렬화를 위한 보일러 플레이트 코드가 클래스 내부에 생기게 된다. 이는 클래스의 역할을 흐리게 할 수도 있을 것 같고, 유지보수를 어렵게 하기도 한다.

추가적으로 kotlin-parcelize 플러그인을 설치하면, **`@Parcelize`** 어노테이션을 사용할 수 있다.

Parcelize 어노테이션은 Parcelable의 보일러 플레이트 코드를 작성한 것과 동일하게 동작한다고 한다. 그릭고 컴파일 타입에 바이트 코드를 조작하기 때문에 런타임시 오버헤드도 발생하지 않는다고 한다.

1. **Serializable vs Parcelable**

안드로이드 직렬화에 대해서 자료를 찾아보면 항상 나오는 내용이다. 많은 자료가 말하길 Serializable이 내부적으로 리플렉션을 사용하기 때문에 편리하지만 시스템 성능에서 비용이 많이 든다고 한다. 

<img  src="https://user-images.githubusercontent.com/71161576/133922609-60c9d085-a9c4-43e1-9c91-e5f5a6f2b04a.png" width="600">

하지만 이에 대한 다른 의견 또한 있었다. Serializable 이 성능상으로 불리한 이유는 당연 리플렉션 때문이고, 이는 Java 의 자동 직렬화 프로세스에서 리플렉션이 사용되기 때문이다.  하지만 Serializable 또한 이 과정을 커스텀 할 수 있다. **writeObject() readObject()** 메서드를 사용하면 자동 직렬화 프로세스를 대체할 수 있고, 리플렉션을 방지할 수 있다.

실제로 이 방식으로 테스트가 진행된 것이 있는데, 이 경우에는 Serializable 의 경우가 Parcelable 보다 더 빠른 속도를 보여줬다고 한다. 

결국 직렬화의 trade-off 를 판단하기 위해서는 그 기준을 Serializable vs Parcelable 이 아니라 

> **사용의 간편함(인터페이스만 구현하고 자동 직렬화 처리를 할 것인지) vs 직렬화 로직을 직접 처리할 것인지**

로 판단을 하는 것이 더 정확한 기준일 것 같다.

1. **gson**

gson 라이브러리는 JSON 형태의 데이터를 Java객체로 직렬화 / 역직렬화 하는 Java 오픈 소스 라이브러리이다.  거꾸로 생각하면 Java 객체를 JSON 형태로 직렬화 / 역직렬화가 가능하다는 의미이다. JSON 형태는 결국 String 타입으로 표현이 되기 때문에 객체를 String 형태로 직렬화가 가능하다. 

```kotlin
val data = Data(name = "jjinse", age = 99)
val gson = Gson()
val jsonData = gson.toJson(data) // jsonData: String!
Log.i(TAG, "$jsonData")
val fromJson = gson.fromJson(jsonData, Data::class.java) // fromJson: Data!
Log.i(TAG, "$fromJson")
```

Data 객체를 String 타입으로 직렬화 / 역직렬화가 가능하다. String 타입은 언제든지 바이트 단위로 변환이 가능하기 때문에 이 방식으로 객체의 전송, 저장 등이 용이해진다.

<img src="https://user-images.githubusercontent.com/71161576/133922637-47c84114-dfea-4387-bda9-5ad12b22e0a1.png" width="600">

1. **kotlinx - serialization**

코틀린 확장팩의 serialization 라이브러리를 사용해도 JSON 타입과 객체 사이의 직렬화 / 역직렬화 가 가능하다.

<img src="https://user-images.githubusercontent.com/71161576/133922640-f4751ffa-8bf2-449f-8d61-9a0574d0dd98.png" width="600">

출처 : [https://blog.jetbrains.com/ko/kotlin/2021/05/kotlinx-serialization-1-2-released/](https://blog.jetbrains.com/ko/kotlin/2021/05/kotlinx-serialization-1-2-released/)

이 라이브러리는 코틀린 라이브러리라는 점이 gson과는 차이점이라고 할 수 있겠다.

사용법은 gson 만큼이나 간단하기 때문에 바로 적용하는 데 문제는 없을 것이다. 직렬화 하고자 하는 클래스에 **`@Serializable`** 어노테이션을 적용하고, JSON / 객체 간의 변환을 하면 된다.