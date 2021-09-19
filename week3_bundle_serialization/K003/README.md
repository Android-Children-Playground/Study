Bundle

### Android Process

Android는 애플리케이션이 시작될 때 그리고 애플리케이션에서 실행되고 있는 컴포넌트가 하나도 없을 때 하나의 스레드로 구성된 프로세스를 시작한다.

기본적으로 모든 컴포넌트는 동일한 프로세스에서 동작한다 하지만 매니페스트에 별도의 설정을 통한다면 Actvitiy, Service, Reciver, Provider와 같은 컴포넌트에 별도의 프로세스를 명시하고 사용할 수 있다.

### IPC, AIDL

Process 간의 통신은 IPC(internal process communication)를 통해서 진행된다. IPC 기술의 베이스로 내부 통신과 보안을 위해 객체를 직렬화하는 기술이 RPC이고 안드로이드에서 사용하는 RPC가 AIDL이다.

### 직렬화 Serialization

Process 간 통신은 IPC를 통해서 일어난다 했다 그런데 여기에 직렬화는 왜 필요한 것일까? 메모리에 데이터는 두 가지 방식으로 존재한다.

Reference Type : 힙 메모리안에 저장된 객체의 메모리 주소를 가지는 타입

Value Type : 원시 타입 같은 변수로 스택 메모리에 주소에 값이 들어가 있는 형태

IPC를 통한 데이터 통신은 Value Type만 가능한데 그 이유는 시스템마다 메모리 주소 공간은 변하고 같은 시스템이라도 메모리에 할당되는 주소는 매번 변하기 때문이다. 그래서 이런 Reference Type의 데이터를 통신 가능하게 Value Type으로 변환해 주는 방식이 `직렬화`이다.



### Android 에서의 직렬화

안드로이드에서 직렬화를 사용하는 방식은 두가지가 있다. 

#### Java Serializable

```kotlin
import java.io.Serializable

data class Position(
    val x: Int,
    val y: Int
) : Serializable
```

먼저 `Serializable`은 Java에서 제공하는 직렬화 방식으로 간단하게 직렬화하려는 객체에 `Serializable`을 구현하면 된다. 이렇게 간단한 이유는 리플렉션을 사용해 Class의 내부 변수를 찾고 이를 직렬화하기 때문이다.

직렬화된 객체를 읽는 과정은 아래와 같다. 직렬화를 위해 `writeObject`를 호출하고 `writeObject`는 내부에서 리플렉션을 이용해 객체를 직렬화한다

![a4](https://kmongcom.files.wordpress.com/2014/03/a4.png?w=584)

```java
private void writeObject0(Object obj, boolean unshared)
    throws IOException
{
    boolean oldMode = bout.setBlockDataMode(false);
    depth++;
    try {
        // handle previously written and non-replaceable objects
        int h;
        if ((obj = subs.lookup(obj)) == null) {
            writeNull();
            return;
        } else if (!unshared && (h = handles.lookup(obj)) != -1) {
            writeHandle(h);
            return;

        Object orig = obj;
        Class<?> cl = obj.getClass();
        ObjectStreamClass desc;

        Class repCl;
        desc = ObjectStreamClass.lookup(cl, true);
        if (desc.hasWriteReplaceMethod() &&
            (obj = desc.invokeWriteReplace(obj)) != null &&
            (repCl = obj.getClass()) != cl)
        {
            cl = repCl;
            desc = ObjectStreamClass.lookup(cl, true);
        }
        .   
        .
        중략
```

`Serializable`은 큰 단점을 가지고 있다 그 단점은 내부적으로 리플렉션을 사용하는데 이때 많은 임시 값을 생성하기 때문에 GC가 자주 일어나 성능에 좋지 않다는 단점을 가지고 있다.

그래서 구글은 이런 단점을 보완하기 위해 `Parcelable`을 만들게 되었다.

#### Parcelable

`Parcelable`은 Android SDK에서 제공하는 직렬화 인터페이스로 `Serializable`과 다르게 내부적으로 리플렉션을 사용하지 않고 리플렉션이 처리하는 부분을 개발자가 직접 처리하도록 설계되었다.

```kotlin
import android.os.Parcel
import android.os.Parcelable

data class Position(
    val x: Int,
    val y: Int
) : Parcelable {
    constructor(parcel: Parcel) : this(
        parcel.readInt(),
        parcel.readInt()
    ) {
    }

    override fun writeToParcel(parcel: Parcel, flags: Int) {
        parcel.writeInt(x)
        parcel.writeInt(y)
    }

    override fun describeContents(): Int {
        return 0
    }

    companion object CREATOR : Parcelable.Creator<Position> {
        override fun createFromParcel(parcel: Parcel): Position {
            return Position(parcel)
        }

        override fun newArray(size: Int): Array<Position?> {
            return arrayOfNulls(size)
        }
    }
}
```

위 코드는 Android Studio가 `Parcelable`을 사용할 때 자동으로 생성해 주는 코드이다 하지만 `Serializable`과 다르게 긴 코드를 작성해 주어야 하는 것을 알 수 있다.

그리고 최근에는 이과정마저 자동으로해주는 **kotlin-parcelize** 플러그인 까지 생겨 코드를 작성해야 하는 보일러 플레이트를 크게 줄일수 있게 되었다. 

### Serializable vs Parcelable

![img](https://www.charlezz.com/wordpress/wp-content/uploads/2018/10/1_d4iAcVhmfIrbGR4c0yqCvw.png)

위 사진은 `Serializable`과 `Parcelable`의 성능 차이를 보여준다. 위 결과에서는 `Parcelable`이 압도적으로 빠른 것을 볼 수 있는데, **실제로는 Serializable가 속도가 느리다고만 볼 수 없다.** 이전 설명에서 직렬화에 사용하는 `readObject`와, `writeObject`를 Override 하여 `Parcelable`처럼 처리한다면

`Serializable`은 `Parcelable`보다 쓰기 3배 읽기는 1.6배나 빠른 결과가 나온다

구글에서는 `Parcelable`을 권장한다 kotlin-parcelize 플러그인과 더불어 안드로이드에 한정에서 개발되었기 때문에 더 적절하다고 생각될 수도 있다.

우리는 또 모듈에 대해 고민을 하기도 해야 한다 우리가 디자인 패턴을 통해 `View`와 `Model`을 분리해 서로의 관심사를 분리하는 것처럼 Android에 대한 의존성을 분리해야 한다면 `Serializable`을 사용해야 할 것이다.

결론적으로 우리는 적절한 트레이드오프를 결정해야 한다 `Serializable`을 선택해 개발의 편리함을 느낄 것인가? `Parcelable`을 사용해 성능을 얻을 것인가? 우리가 개발하는 아키텍처의 목표는 무엇인가? 이런 다양한 관점을 고민하고 그때에 맞는 최선의 방법을 고르면 될 것이다.



### Bundle 

Bundle은 Android에서 IPC 통신을 위한 직렬화 객체이다. Parcelable를 구현하고 있어 직렬화를 처리하고 있고 내부에 원시타입과 직렬화가 가능한 객체만 입력을 받을 수 있는 ArrayMap을 가지고 있어 여러 가지 데이터를 한 번에 담아 전달할 수 있다

```java
@UnsupportedAppUsage
ArrayMap<String, Object> mMap = null;
```

```java
@Override
public void putObject(@Nullable String key, @Nullable Object value) {
    if (value instanceof Byte) {
        putByte(key, (Byte) value);
    } else if (value instanceof Character) {
        putChar(key, (Character) value);
    } else if (value instanceof Short) {
        putShort(key, (Short) value);
    } else if (value instanceof Float) {
        putFloat(key, (Float) value);
    } else if (value instanceof CharSequence) {
        putCharSequence(key, (CharSequence) value);
    } else if (value instanceof Parcelable) {
        putParcelable(key, (Parcelable) value);
    ... 중략
```



#### 왜 Bundle인가?

우리는 앱의 상태저장에서 `Bundle`을 사용해 데이터를 저장 복원한다 상태 저장에는 앱과 `Activity`마다 필요한 데이터의 수나 형태가 다양할것이다 그래서 Bundle이라는 하나의 객체에 다양한 정보를 담을 수 있는 `Map`형태를 이용한 것으로 생각된다. 

또 생각해보면 앱이 파괴되었다 복원될때 복원의 주체는 `Android System Process`라고 볼수 있다 그런데 그 `Process`는 앱이 파괴되었을때 정보를 받아야하고 이를 저장했다가 다시 앱을 복원할때 전달해야 주어야 하는데 이때 직렬화된 객체인 `Bundle`을 통해 이과정을 처리한다고 생각 할 수 있다. 

한마디로 다양한 앱과 컴포넌트에 범용성을 가지고 직렬화 가능한 객체를 통해 이를 간단하게 처리하려고 `Bundle`을 사용한다고 볼수 있다. 