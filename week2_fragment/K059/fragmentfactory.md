## Fragment 생성자를 변경하면 안되는 이유 + FragmentFactory

우선 예제를 보면서 확인해보겠습니다.

`MainActivity` 에 Fragment를 올리고 button 을 누르면 Fragment를 replace 하도록 했습니다.

```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="<http://schemas.android.com/apk/res/android>"
    xmlns:app="<http://schemas.android.com/apk/res-auto>"
    xmlns:tools="<http://schemas.android.com/tools>"
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
class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        Log.d("Life", "MainActivity : onCreate()")
        if (savedInstanceState == null) Log.d("Test","MaintActivity OnCreate()  savedInstanceState null")
        else Log.d("Test","MaintActivity OnCreate()  savedInstanceState not null")
        
				super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        val gameFragment = GameFragment()
        val settingFragment = SettingFragment()

        if (savedInstanceState == null) {
            supportFragmentManager.beginTransaction()
                .replace(R.id.fcv, gameFragment)
                .setTransition(FragmentTransaction.TRANSIT_FRAGMENT_OPEN)
                .commit()
        }

        findViewById<Button>(R.id.btn_game).setOnClickListener {
            supportFragmentManager.beginTransaction()
                .replace(R.id.fcv, gameFragment)
                .setTransition(FragmentTransaction.TRANSIT_FRAGMENT_OPEN)
                .commit()
        }
        findViewById<Button>(R.id.btn_setting).setOnClickListener {
            supportFragmentManager.beginTransaction()
                .replace(R.id.fcv, settingFragment)
                .commit()
        }

    }

    override fun onStart() {
        super.onStart()
        Log.d("Life", "MainActivity : onStart()")
    }

    override fun onResume() {
        super.onResume()
        Log.d("Life", "MainActivity : onResume()")
    }

    override fun onPause() {
        super.onPause()
        Log.d("Life", "MainActivity : onPause()")
    }

    override fun onStop() {
        super.onStop()
        Log.d("Life", "MainActivity : onStop()")
    }

    override fun onDestroy() {
        super.onDestroy()
        Log.d("Life", "MainActivity : onDestroy()")
    }

    override fun onSaveInstanceState(outState: Bundle) {
        super.onSaveInstanceState(outState)
        Log.d("Life", "MainActivity : onSaveInstanceState()")
    }

    override fun onSaveInstanceState(outState: Bundle, outPersistentState: PersistableBundle) {
        super.onSaveInstanceState(outState, outPersistentState)
        Log.d("Life", "MainActivity : onSaveInstanceState() outPersistentState")
    }

    override fun onRestart() {
        super.onRestart()
        Log.d("life", "MainActivity : onRestart()")
    }
}

class GameFragment: Fragment() {
    
    override fun onCreateView(
        inflater: LayoutInflater,
        container: ViewGroup?,
        savedInstanceState: Bundle?
    ): View? {
        //Log.d("Test", a)
        Log.d("Life", "GameFragment onCreateView()")
        return super.onCreateView(inflater, container, savedInstanceState)
    }

    override fun onAttach(context: Context) {
        super.onAttach(context)
        Log.d("Life", "GameFragment onAttach()")
    }

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        Log.d("Life", "GameFragment onCreate()")
    }

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        Log.d("Life", "GameFragment onViewCreate()")
        super.onViewCreated(view, savedInstanceState)
    }

    override fun onViewStateRestored(savedInstanceState: Bundle?) {
        super.onViewStateRestored(savedInstanceState)
        Log.d("Life", "GameFragment onViewStateRestored()")
    }

    override fun onStart() {
        super.onStart()
        Log.d("Life", "GameFragment onStart()")
    }

    override fun onResume() {
        super.onResume()
        Log.d("Life", "GameFragment onResume()")
    }

    override fun onPause() {
        super.onPause()
        Log.d("Life", "GameFragment onPause()")
    }

    override fun onStop() {
        super.onStop()
        Log.d("Life", "GameFragment onStop()")
    }

    override fun onSaveInstanceState(outState: Bundle) {
        super.onSaveInstanceState(outState)
        Log.d("Life", "GameFragment onSaveInstanceState()")
    }

    override fun onDestroyView() {
        super.onDestroyView()
        Log.d("Life", "GameFragment onDestroyView()")
    }

    override fun onDestroy() {
        super.onDestroy()
        Log.d("Life", "GameFragment onDestroy()")
    }
}

class SettingFragment : Fragment(R.layout.fragment_setting)
```

`MainActivity` 에 `Fragment`를 하나 올리고 화면을 회전시켜 보면 다음과 같이 Lifecycle callback이 호출됩니다.

![image](https://user-images.githubusercontent.com/55446114/132986557-d3568d71-ff1b-4486-9655-c7b4bb2b0342.png)

여기서 중요한 것이 `onSaveInstanceState()` 이 호출된 다는 점 입니다. 회전을 시키게 되면 Fragment와 Activity의 상태를 저장한 후, Activity와 Fragment를 완전히 Destroy 하고 Activity의 onCreate() 부터 다시 시작하게 됩니다.

여기서 GameFragment에 생성자를 추가해보겠습니다.

```kotlin
constructor(value: Int){
    Log.d("Test", value.toString())
}
```

이 상태에서 GameFragment의 부모인 Fragment의 생성자와 GameFragment의 생성자는 어떻게 호출 될까요?

Fragment의 기본 생성자가 먼저 호출되고, GameFragment의 생성자가 호출 됩니다.

```kotlin
class Fragment {
		...

		public Fragment() {
        initLifecycle();
    }
		...
}
```

그리고 이 상태에서 회전을 시켜보면, 에러가 발생하여 앱이 죽어버리게 됩니다 .

왜 에러가 발생했는지 에러 메시지를 추적해 보겠습니다.

![image](https://user-images.githubusercontent.com/55446114/132986581-0adaef40-b0b4-4b26-b634-9ccf18edeca1.png)

회전을 하게 되면 `onSaveInstanceState()` 에서 Fragment와 Activity의 상태를 저장하게 됩니다. 그리고 Activity의 onCreate() 부터 다시 시작하게 된다고 위에서 설명했는데요,

에러 메시지에서도 onCreate() 할때 문제가 있음을 알 수 있습니다.

예제 코드의 MaintActivity 에서 다음의 코드를 보면,

```kotlin
if (savedInstanceState == null) Log.d("Test","MaintActivity OnCreate()  savedInstanceState null")
else Log.d("Test","MaintActivity OnCreate()  savedInstanceState not null")
```

최초에 앱을 실행 시켰을 때, savedInstanceState 는 null 이고 , 회전을 시키면 savedInstanceState는 null 이 아니게 됩니다. 상태를 저장 했으니까요.

savedInstanceState가 null 이 아니라면, FragmentManager가 상태를 `restore`하는 작업을 하게 됩니다. 위의 에러 메시지에서 FragmentManager.restoreSaveState 이 호출 된다는 것을 확인할 수 있습니다.

restore 과정에서 다음의 `instantiate` 함수가 호출 됩니다

```java
/**
* Create a new instance of a Fragment with the given class name.  This is
* the same as calling its empty constructor, setting the {@link ClassLoader} on the
* supplied arguments, then calling {@link #setArguments(Bundle)}.
* ... 
* ...
**/
@Deprecated
@NonNull
public static Fragment instantiate(@NonNull Context context, @NonNull String fname,
      @Nullable Bundle args) {
  try {
      Class<? extends Fragment> clazz = FragmentFactory.loadFragmentClass(
              context.getClassLoader(), fname);
      Fragment f = clazz.getConstructor().newInstance();
      if (args != null) {
          args.setClassLoader(f.getClass().getClassLoader());
          f.setArguments(args);
      }
      return f;
  } 
	...
  catch (NoSuchMethodException e) {
      throw new InstantiationException("Unable to instantiate fragment " + fname
              + ": could not find Fragment constructor", e);
  } 
	...
}
```

class name 을 넘겨 받아 Fragment의 instance를 생성하는 함수 입니다. https://kotlinworld.com/73 이 블로그에서는 이 함수를 다음과 같이 설명하고 있습니다.

```java
1. Class Loader를 이용하여 Fragment class 생성 
Class<? extends Fragment> clazz = FragmentFactory.loadFragmentClass( context.getClassLoader(), fname); 

2. class의 기본 constructor을 가져와서 fragment 인스턴스 생성 
Fragment f = clazz.getConstructor().newInstance(); 

3. Bundle 객체인 args를 이용하여 fragment에 전달할 값 세팅 
if (args != null) { 
	args.setClassLoader(f.getClass().getClassLoader()); 
	f.setArguments(args); 
}

4. 생성된 fragment인 f를 return 
return f;
```

에러가 발생하는 부분이 `clazz.getConstructor()` 이 부분 입니다. `getConstructor()` 이렇게 호출하면 생성자에 파라미터가 없는 기본 생성자를 호출하게 되는데요,

```java
public Constructor<T> getConstructor(Class<?>... parameterTypes)
    throws NoSuchMethodException, SecurityException {
    return getConstructor0(parameterTypes, Member.PUBLIC);
}
```

즉, getConstructor() 의 파라미터에 생성자의 parameterTypes 을 넘겨줘야  우리가 만들었던 생성자가 정상적으로 호출이 됩니다. 하지만 파라미터가 없는 기본 생성자를 호출하고 있죠.

따라서 파라미터가 없는 생성자는 GameFragment에 없으니, `NoSuchMethodException` 이 발생하게 되는 것입니다.

그러면 우리는 어떻게 Fragment가 생성될 때 인자를 넘겨 줘야 할까요?

출처: https://medium.com/capital-one-tech/android-fragmentfactory-75823af015fd

### Bundle + 팩토리 메서드 패턴

```java
class MyFragment : Fragment() {
    private lateinit var arg: String
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        arguments?.getString(ARG) ?: ""
    }
    companion object {
        fun newInstance(arg: String) =
            MyFragment().apply {
                arguments = Bundle().apply {
                    putString(ARG, arg)
                }
            }
    }
}
```

내부에 빈 생성자를 사용하여 인스턴스를 생성하는 팩토리 메서드를 하나 만들고, 생성할 때 필요한 인자를 bundle에 넣고 전달하는 방법입니다.

Fragment에선  getArguments() 메서드로 전달된 데이터를 꺼내서 사용할 수 있습니다.

이 방법의 문제는 bundle 을 계속해서 관리해줘야 한 다는 점이 있습니다.

### FragmentFactory

위에서 `instantiate` 함수를 자세히 보면 deprecated 어노테이션이 붙어있는 것을 확인 할 수 있습니다. android docs 에서 FragmentFactory 의 `instantiate` 을 사용하라고 설명하고 있네요.

![image-20210912205517201](/Users/choijh/Library/Application Support/typora-user-images/image-20210912205517201.png)

그렇다면 이 `FragmentFactory`는 과연 무엇일까요? 왜 만들어졌을까요?

출처: https://proandroiddev.com/android-fragments-fragmentfactory-ceec3cf7c959

위 블로그에서는 `FragmentFactory` 존재 이유를 다음과 같이 설명합니다.

> *전통적으로 Fragment 인스턴스는 기본 빈 생성자를 사용해서만 인스턴스화할 수 있었습니다. 이는 구성 변경 및 앱의 프로세스 재생성과 같은 특정 상황에서 시스템을 다시 초기화해야 하기 때문입니다. 기본 생성자 제한이 없다면 시스템은 Fragment 인스턴스를 다시 초기화하는 방법을 알지 못할 것입니다.*

그렇죠. 초기화 하는 경우는 반드시 생기는 것이라 어쩔 수 없습니다. 그렇다면 초기화할 때 인스턴스를 다시 초기화해야 하는데, 시스템은 어떤 생성자를 써야할 지 알 수가 없죠. 개발자가 어떤 생성자를 이용했는지 알 길이 없으니까요.

> *FragmentFactory는 이 제한을 해결하기 위해 만들어졌습니다. 프래그먼트를 인스턴스화하는 데 필요한 필수 인수/종속성을 제공하여 시스템이 프래그먼트 인스턴스를 생성하는 데 도움이 됩니다*.

이런 문제를 해결하기 위해 `FragmentFactory` 가 만들어졌다고 합니다.

아래와 같이 사용할 수 있습니다.

```java
class FragmentFactoryImpl(private val arg: String): FragmentFactory() {
    override fun instantiate(classLoader: ClassLoader, className: String): Fragment {
        return when (className) {
            MyFragment::class.java.name -> MyFragment(arg)
            MyFragment2::class.java.name -> {
                MyFragment2().apply {
                    arguments = Bundle().apply {
                        putString("key", arg)
                    }
                }
            }
            else -> super.instantiate(classLoader, className)
        }
    }
}
```

그런데 중요한 점은 FragmentManager 가 어떻게 우리가 만든 FragmentFactory 구현체를 알고 사용하게 하냐 인데요. FragmentManager에 FragmentFactory를 할당하는 작업이 필요합니다.

**그렇다면 언제 어디에서 할당을 해주어야 할까요?**

위 블로그에서는 다음과 같이 설명합니다.

> 구성 요소(Activity 또는 상위 Fragment) 내에서 Fragment 초기화를 담당하는 FragmentFactory는 Fragment가 생성되기 전에 설정되어야 합니다.

이는 다음을 의미합니다.

- Fragment가 xml에 정의된 경우(<Fragment> 태그 or FragmentContainerView) , 이 경우에는 component’s View 가 created 되기 전에 설정되어야 합니다
- Fragment가 FragmentTransaction을 이용하여 동적으로 추가되는 경우, 이 경우에는 component가 Fragment를 생성하기 전에 설정되어야 합니다.
- 구성이 변경되거나 앱의 프로세스가 다시 시작된 후 Fragment가 다시 생성되는 경우, 이 경우에는 시스템이 restore 하기전에 설정되어야 합니다. 우리가 위의 예제에서 살펴본 경우와 같은 케이스가 되겠네요.

이런 여러 제한 사항들을 감안하면,

> *Activity#onCreate() 및 Fragment#onCreate()(이는 Activity 및 Fragment의 기본 클래스)가 호출되기 전에 구성 요소의 FragmentManager에 FragmentFactory를 할당하는 것이 안전합니다.*

이것은 super#onCreate()를 호출하기 전에 FragmentFactory를 할당해야 함을 의미합니다.

```java
class HostActivity : AppCompatActivity() {
    private val customFragmentFactory = CustomFragmentFactory(Dependency())

    override fun onCreate(savedInstanceState: Bundle?) {
        supportFragmentManager.fragmentFactory = customFragmentFactory
        super.onCreate(savedInstanceState)
        // ...
    }
}
```

### 그 외의 방법

그러면 FragmentFactory를  무조건 사용해야 할까요?

그건 아닙니다. 하지만 특정 상황에서는 FragmentFactory가 더 나은 디자인 옵션일 수 있습니다.

다른 방법으론 `Dagger` 또는 `Koin` 라이브러리를 사용하는 방법이 있습니다.

지금까지 기본 생성자를 사용하여 프래그먼트를 생성한 다음 Dagger 또는 Koin과 같은 라이브러리를 사용하여 필요한 종속성을 주입하거나 사용되기 전 특정 시점에서 프래그먼트 내부에서 간단히 초기화했습니다.

> *즉, Fragment에 기본 빈 생성자가 있으면 FragmentFactory를 사용할 필요가 없습니다. 그러나 Fragment가 생성자에서 인수를 사용하는 경우 FragmentFactory를 사용해야 합니다. 그렇지 않으면 사용되는 기본 FragmentFactory가 Fragment 인스턴스를 인스턴스화하는 방법을 모르기 때문에 Fragment.InstantiationException이 발생합니다.*

더 자세한 내용은 위 블로그를 참고하면 좋을 것 같습니다.