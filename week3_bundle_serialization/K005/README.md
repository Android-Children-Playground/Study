# Bundle과 Serialization

# Bundle

안드로이드에서 Activity를 비롯한 컴포넌트는 시스템에 의해 제어된다. 때문에 컴포넌트 사이의 상호작용을 위해서는 일반적인 객체 처럼 인스턴스를 참조할 수 없고 시스템의 중개가 필요하다.

이때 컴포넌트간에 전달되는 메시지는 `Bundle`의 형태로 송수신된다. `Bundle`의 정의는 다음과 같다.

> *A mapping from String keys to various `Parcelable` values.*

Bundle 자체는 key-value 형태로 데이터를 저장하는 자료 구조에 불과하다. 대체로 Bundle은 Intent에 함께 전달되는 형태로 사용된다.

컴포넌트간의 메시지 전달을 우편에 비유하면 Bundle은 편지에, Intent는 편지 봉투라고 할 수 있다.

## 구조

Bundle 클래스의 구조를 보면 Bundle 클래스는 BaseBundle을 확장하며, Cloneable과 Parcelable 인터페이스를 구현하고 있다.

BaseBundle 부터 살펴보자. BaseVundle의 정의는 다음과 같다.

> *A mapping from String keys to values of various types. In most cases, you should work directly with either the `Bundle` or `PersistableBundle` subclass.*

Bundle 클래스와 마찬가지로 BaseBundle 클래스는 String을 Key로 하여 다양한 타입의 값에 매핑하는 자료 구조이다. 대부분의 경우 BaseBundle을 직접 사용하는 대신 클래스를 상속하는 Bundle이나 PersistableBundle 클래스를 사용한다.

BaseBundle 클래스의 인터페이스를 보면 정말 단순히 키-값 형태로 데이터를 저장하는 역할만 한다는 것을 볼 수 있다.

```java
boolean containsKey(String key)

Object get(String key)

XXX getXXX<Boolean | Double | Int | Long | String>(String key, [XXX defaultValue])

XXX[] getXXX<Boolean | Double | Int | Long | String>Array(String key)

boolean isEmpty()

Set<String> keySet()

void putAll(PersistableBundle bundle)

void putXXX<Boolean | Dbouel | Int | Long | String>(String key, XXX value)

void putXXX<Boolean | Double | Int | Long | String>Array(String key, XXX[] value)

void putBoolean(String key, boolean value)

void remove(String key)

int	size()
```

클래스 구현을 보면 위의 타입 뿐만 아니라 `Byte`, `Char`, `Short`, `Float`, `CharSequence`, `IntegerArrayList`, `StringArrayList`, `CharSequenceArrayList`, `Serializable` 타입 및 배열 타입에 대해서도 get, put 메소드가 있지만 `package private` 가시성으로 제공된다.

Bundle 클래스는 BaseBundle 클래스에서 package private 하게 제공하는 메소드에 대해 public 가시성을 제공하며, Parcelable 타입 등에 대한 데이터 저장과 조회를 제공한다.

## 목적

그렇다면 컴포넌트간의 데이터를 왜 이런 방식으로 전달하는 것일까?

여기서 알아야 할 점은 안드로이드 환경에서 컴포넌트간의 상호작용이 동일한 애플리케이션(프로세스) 내에서의 상호작용으로 국한되지 않는다는 것이다.

사용자 애플리케이션에서 미디어 관련 기능이 필요하다면 애플리케이션 내에서 카메라 앱을 실행하여 얻은 결과를 사용할 수도 있고, 애플리케이션에서 제공하는 데이터를 다른 애플리케이션을 사용하여 공유할 수도 있다.

하지만 각각의 프로세스는 자신만의 메모리 영역을 가지고 있으며, 다른 프로세스의 메모리 영역에 접근할 수 없다. 즉, 프로세스는 메모리 영역을 공유하지 않기 때문에 프로세스 사이의 상호작용에서는 직접적인 호출과 전달이 불가능하다.

따라서 프로세스간의 상호작용을 위해서는 안드로이드 커널의 지원이 필요하며, 제공하려는 데이터를 커널이 이해할 수 있는 형태로 변환하고 다시 커널로부터 복구하는 과정이 필요하다.

Bundle은 이와 같은 프로세스 간 통신(IPC)에서도 데이터를 전달할 수 있도록 설계되었다.

BaseBundle 클래스는 `writeToParcelInner`와 `readFromParcelInner` 라는 메소드를 가지고 있는데, 이 메소드에서는 ArrayMap으로 관리하던 key-value 데이터를 `Parcel` 객체에 쓰는 작업과 Parcel로부터 데이터를 복구하는 작업을 수행한다.

read 과정에서 Parcel로부터 복구된 데이터는 내부적으로 다시 Parcel 타입의 mParcelledData라는 필드에 저장된다. 이 필드는 Bundle에서 get 또는 put 메소드에서 항상 먼저 실행되는 unparcel이라는 메소드에서 사용되는데, unparcel 메소드에서는 mParcelledData 필드가 null인지 확인하며 내부의 ArrayMap을 초기화하는 작업을 수행한다.

# 직렬화(Serialization)

> 객체를 영속화 하는 메커니즘.
> ⇒ 객체를 파일, 메모리 등에 저장했다가 나중에 다시 재구성 할 수 있게 만드는 과정.

앞에서 소개했듯 `Bundle`에 저장된 데이터는 IPC 과정에서 `Parcel` 객체를 사용하여 직렬화/역직렬화 된다.

하지만 자바는 이미 객체의 직렬화/역직렬화를 위해 `Serializable` 이라는 메커니즘을 제공한다. 자바 클래스의 직렬화/역직렬화가 필요하다면 클래스가 `Serializable` 인터페이스를 구현하면 되는데, `Serializable` 인터페이스 자체는 메소드가 없는 마커 인터페이스이기 때문에 추가적인 구현 코드를 작성할 필요가 없다.

반면에 `Parcel`을 사용한 직렬화의 경우에는 클래스가 `Parcelable` 인터페이스를 구현해야 하며, 추가적으로 객체를 어떻게 직렬화하고 다시 역직렬화할지에 대한 메소드를 작성해야 한다.

그렇다면 왜 안드로이드 진영은 객체의 직렬화를 위해 Parcel을 사용하는 것일까?

## 자바 직렬화

- 자바 직렬화란 자바 시스템 내부에서 사용되는 객체 또는 데이터를 외부의 자바 시스템에서도 사용할 수 있도록 바이트(byte) 형태로 데이터 변환하는 기술과, 바이트로 변환된 데이터를 다시 객체로 변환하는 기술(역직렬화)을 아울러서 표현.
- 시스템적으로 이야기하자면 JVM(Java Virtual Machine 이하 JVM)의 메모리에 상주(힙 또는 스택)되어 있는 객체 데이터를 바이트 형태로 변환하는 기술과, 직렬화된 바이트 형태의 데이터를 객체로 변환해서 JVM으로 상주시키는 형태를 함께 나타낸다.

### 사용 방법

직렬화시킬 클래스가 Serializable 인터페이스를 구현하기만 하면 된다. Serializable 인터페이스를 구현하는 클래스는 ObjectInputStream, ObjectOutputStream을 사용하여 읽기/쓰기 작업을 수행할 수 있다.

```kotlin
data class Test(val str: String, val num: Int, private val age: Int): Serializable
```

```kotlin
fun main() {
    val data = Test("Alice", 25, 100)

    val bos = ByteArrayOutputStream()
    val oos = ObjectOutputStream(bos)

    oos.writeObject(data)

    val byteArray = bos.toByteArray()

    val bis = ByteArrayInputStream(byteArray)
    val ois = ObjectInputStream(bis)

    val restore = ois.readObject() as Test

    println(restore)
}

// output: Test(str=Alice, num=25, age=100)
```

이처럼 자바 직렬화를 사용한 방식은 추가적인 코드가 필요 없고, 쉽게 적용할 수 있다.

### 단점

그렇다면 왜 안드로이드는 자바 직렬화 대신 Parcelable을 사용하는 걸까? 이유는 자바 직렬화가 가지는 근본적인 문제가 있기 때문이다.

**Performance**

자바 직렬화는 사용을 위한 개발자의 부담을 줄여준다. 대신 줄어든 부담은 시스템이 감당하게 된다.

자바 직렬화는 내부적으로 객체의 직렬화를 위해 Reflection을 사용한다. reflection은 런타임중에 수행되며 처리를 위해 추가적인 객체를 다량 생성한다. 이렇게 생성된 객체들은 직렬화가 끝나면 GC 대상이 되며, GC의 빈번한 수행은 성능 저하 및 배터리 성능에 악영향을 끼칠 수 있다.

하지만 자바 직렬화의 성능에 대해선 여러 의견이 있다.

클래스 내에서 readObject, writeObject 등의 메소드를 사용한다면, 객체가 직렬화/역직렬화되는 과정을 직접 정의해줄 수 있고, 이렇게 정의된 객체는 직렬화 과정에서 좋은 성능을 보여준다는 의견이 있다.

**클래스 변경**

기본적으로 자바 직렬화에 사용되는 클래스는 변경할 수 없다.

이미 객체에 대한 직렬화 작업이 수행된 후 클래스에 프로퍼티가 추가된다면, 역직렬화 과정에서 해당 타입으로 캐스팅할 수 없고 예외가 발생한다.

이를 해결하기 위해선 클래스의 static 필드로 serialVersionUID를 지정해줄 수 있다.

```kotlin
data class Test(val str: String, val num: Int): Serializable {
    companion object {
        const val serialVersionUID = 1L
    }
}
```

serialVersionUID가 지정된 클래스는, 직렬화 이후 클래스의 프로퍼티가 추가, 삭제 되더라도 값의 누락만 발생할 뿐 정상적으로 동작할 수 있다.

하지만 이런 방식이 모든 경우에 정상적으로 동작하지는 않는다.

코틀린의 경우 생성자에 default 파라미터를 정의할 수 있다.

```kotlin
data class Test(val str: String, val num: Int, private val age: Int = 10): Serializable {
    companion object {
        const val serialVersionUID = 1L
    }
}
```

클래스가 위와 같이 변경된 상황에서 자바의 역직렬화를 수행하면 age의 값이 10 대신 0이 들어가는 것을 볼 수 있다. 이 경우엔 readObject 메소드 내에서 기본 값을 설정하는 과정이 필요하다.

또한 자바 직렬화는 타입에 매우 엄격하다.

프로퍼티의 추가, 삭제 등의 변경은 serialVersionUID를 사용하여 어느 정도 처리할 수 있지만, 프로퍼티의 타입이 변경된다면 역직렬화가 불가능해진다.

**저장 공간**

자바 직렬화시에 기본적으로 타입에 대한 정보 등 클래스의 메타 정보도 가지고 있기 때문에 상대적으로 다른 포맷에 비해서 용량이 큰 문제가 있으며, 특히 클래스의 구조가 거대해지게 되면 용량 차이는 더 커진다.

그래서 JSON 같은 최소의 메타정보만 가지고 있으면 같은 데이터에서 최소 2배 최대 10배 이상의 크기를 가질 수 있다.

## Parcelable

> *Container for a message (data and object references) that can be sent through an IBinder.*

Parcelable은 안드로이드 SDK에서 제공하는 인터페이스로 리플렉션을 사용하지 않는 직렬화 메커니즘을 제공한다.

인터페이스의 구현을 위해선 객체의 직렬화와 역직렬화를 어떻게 처리할지에 대한 메소드를 명시적으로 구현해줘야 하기 때문에 리플렉션 과정이 필요 없으며, 이로 인해 성능상 이점을 가져올 수 있다.

### 사용 방법

```kotlin
data class Test(val str: String, val num: Int, private val age: Int = 10) : Parcelable {
    constructor(parcel: Parcel) : this(
        parcel.readString() ?: "",
        parcel.readInt(),
        parcel.readInt()
    )

    override fun describeContents() = 0

    override fun writeToParcel(dest: Parcel, flags: Int) {
        dest.writeString(str)
        dest.writeInt(num)
        dest.writeInt(age)
    }

    companion object CREATOR : Parcelable.Creator<Test> {
        override fun createFromParcel(parcel: Parcel): Test {
            return Test(parcel)
        }

        override fun newArray(size: Int): Array<Test?> {
            return arrayOfNulls(size)
        }
    }
}
```

Parcelable을 구현하는 클래스는 위의 코드 처럼 객체의 직렬화 과정에 대한 코드를 작성해야 한다. 따라서 클래스가 변경된다면 Parcelable 관련 메소드 또한 함께 변경해줘야 하기 때문에 Serializable을 사용한 방식보다 유지보수성 측면에서 떨어진다.

또한 아래와 같은 조건을 만족해야 한다.

- 직렬화 class의 패키지 명이 같아야 한다.
- 역직렬화는 직렬화 때 데이터를 저장한 순서대로 읽어야 한다.

### 주의 사항

Parcel 공식 문서에 따르면 아래와 같은 주의 사항을 보여준다.

> Parcel is **not** a general-purpose serialization mechanism. This class (and the corresponding Parcelable API for placing arbitrary objects into a Parcel) is designed as a high-performance IPC transport. As such, it is not appropriate to place any Parcel data in to persistent storage: changes in the underlying implementation of any of the data in the Parcel can render older data unreadable.

Parcel은 범용 직렬화 메커니즘이 아니다. Parcel 클래스는 IPC에서 빠른 전송을 위해 설계되었다. 따라서 Parcel 데이터를 persistent에 저장하는 것은 적절하지 않다.

이는 객체의 상태를 Parcel 클래스를 사용하여 데이터베이스등의 persistence에 저장하는 것은 적절하지 않은 사용법이며, 프로세스 간 통신을 목적으로만 사용해야 함을 말한다.

데이터베이스, 네트워크 등 크로스 플랫폼에서 호환성을 유지하기 위해선 JSON 등의 방식을 사용하는 것이 좋다.

## Plugin

안드로이드와 코틀린 진영은 코틀린 클래스의 Parcelize와 JSON 직렬화를 위한 플러그인을 제공한다.

이를 사용하면 Parcelable을 구현하는데 필요한 보일러 플레이트 코드를 줄일 수 있으며 JSON 변환을 쉽게 적용할 수 있다.

### Parcelize

Parcelize 플러그인을 사용하려면 Gradle 파일에서 아래의 의존성을 추가한다.

```groovy
plugins {
    ...
    id 'kotlin-parcelize'
}
```

플러그인이 추가되면 클래스 상단에 `@Parcelize` 어노테이션을 붙이는 것으로 Parcelable을 구현할 때 필요했던 코드를 제거할 수 있다.

```kotlin
@Parcelize
data class Test(val str: String, val num: Int, private val age: Int = 10): Parcelable
```

### Serialization

kotlin-serialization 을 사용하기 위해선 Gradle을 다음과 같이 수정한다.

```groovy
plugins {    ...    id 'org.jetbrains.kotlin.plugin.serialization' version '1.5.30'}dependencies {    ...    implementation 'org.jetbrains.kotlinx:kotlinx-serialization-json:1.3.0-RC'}
```

이후 클래스에 `@Serializable` 어노테이션을 추가하는 것으로 JSON 직렬화가 가능하다.

```kotlin
@Serializabledata class Test(val str: String, val num: Int, private val age: Int = 10)
```

사용 예시는 아래와 같다.

```kotlin
val json = Json.encodeToString(Test("alice", 10, 20))println(json)val testData = Json.decodeFromString<Test>("""{"str":"alice","num":10,"age":20}""")println(testData)
```

다만 API에 따라 아직 Experimental 기능인 경우가 있어 사용에 주의가 필요하다.

# 참고

[https://developer.android.com/reference/android/os/Parcelable](https://developer.android.com/reference/android/os/Parcelable)

[https://developer.android.com/guide/components/activities/parcelables-and-bundles?hl=ko](https://developer.android.com/guide/components/activities/parcelables-and-bundles?hl=ko)

[https://techblog.woowahan.com/2551/](https://techblog.woowahan.com/2551/)

[https://www.charlezz.com/?p=823](https://www.charlezz.com/?p=823)

[https://d2.naver.com/helloworld/47656](https://d2.naver.com/helloworld/47656)

[https://medium.com/@limgyumin/parcelable-vs-serializable-정말-serializable은-느릴까-bc2b9a7ba810](https://medium.com/@limgyumin/parcelable-vs-serializable-%EC%A0%95%EB%A7%90-serializable%EC%9D%80-%EB%8A%90%EB%A6%B4%EA%B9%8C-bc2b9a7ba810)

[https://www.youtube.com/watch?v=3iypR-1Glm0&t=30s](https://www.youtube.com/watch?v=3iypR-1Glm0&t=30s)

[https://velog.io/@devseunggwan/Android-Serializable-Parcelable](https://velog.io/@devseunggwan/Android-Serializable-Parcelable)

[https://parkho79.tistory.com/112](https://parkho79.tistory.com/112)

[https://kotlinlang.org/docs/serialization.html#example-json-serialization](https://kotlinlang.org/docs/serialization.html#example-json-serialization)

[https://developer.android.com/kotlin/parcelize?hl=ko](https://developer.android.com/kotlin/parcelize?hl=ko)

