# 주제

Fragment 상태전환 라이프사이클(Viewbinding fragment 메모리릭 문제)

- Fragment 생성자를 변경하면 안되는 이유
- Fragement 상태 유지 방법 argument, savedInstancestate
- FragmentFactory 이게 뭘까 구글이 왜 만들었을까

# 1. Fragment 생성자

일반적으로 Fragment를 만들 때는 생성자를 오버로딩 하지 않고 생성 시 필요한 파라미터가 생기면 Bundle 객체에 담아서 setArgument() 함수를 호출하는 방식을 사용한다.

- 기본 생성자 외에는 사용할 수 없다?

기본 생성자를 사용하지 않고 Fragment 인스턴스를 생성할 경우 다음과 같은 에러 메시지가 뜬다.

`Error message: android.support.v4.app.Fragment$InstantiationException: Unable to instantiate fragment make sure class name exists, is public, and has an empty constructor that is public`

- 기본 생성자만 사용하게 구현된 이유

Fragment를 만들 때는 생성자를 오버로딩 하지 않고 생성 시 파라미터를 Bundle에 담아 setArgument() 함수를 호출하는 방식이 일반적이다.

안드로이드에 의해 Fragment가 복원될 때는 Fragment의 `기본 생성자`를 호출하기 때문에 오버로딩된 생성자의 호출이 보장되지 않기때문이다.

즉, 파라미터를 그대로 전달하는게 아닌, Bundle 객체를 사용해서 전달해주는 것이 일반적인 사용법이기때문에 이렇게 구현되었다.

Fragment 인스턴스 생성은 instantiate 메소드를 보면 잘 이해할 수 있다.

```java
public static Fragment instantiate(@NonNull Context context, @Nonnull String fname, @Nullable Bundle args) 
```

### 1) Class loader를 사용하여 Fragment class를 생성한다.

```java
Class<? extends Fragment> clazz = 
		FragmentFactory.loadFragmentClass( context.getClassLoader(), fname);
```

### 2) class의 기본 constructor를 가져와서 fragment 인스턴스를 생성한다.

```java
Fragment f = clazz.getConstructor().newInstance();

// getConstructor() class의 기본 생성자를 리턴
// newInstance() 생성자로 인스턴스 생성

/* 
	getConstructor() class의 생성자를 지명해서 가져올 수 있도록 생성자의 파라미터 정보를 함수의 인자로 받아올 수 있다.
	현재 getConstructor 에 아무런 인자가 없기 때문에 기본 생성자를 가져오는 것을 확인할 수 있다.
*/
```

### 3) setArguments로 Bundle 객체로 파라미터로 넘어온 값을 저장한다.

```java
if (args != null) { 
		args.setClassLoader(f.getClass().getClassLoader());
		f.setArguments(args); 
}
```

### 4) fragment 리턴

```java
return f;
```

2) 에서 기본 생성자를 가져와 fragment 인스턴스가 생성된다. (이외 다른 생성자는 가져오지 않음)

custom 생성자를 통해 넘겨준 변수를 사용할 수 없기 때문에 Fragment의 데이터 문제가 생길 수 있다.

- AndroidX에서 인자가 있는 프래그먼트 생성

AndroidX로 업데이트 되면서 기존의 instantiate()는 deprecated 되었다.

이제 FragmentFactory의 instantiate를 사용하여 생성자를 구현할 수 있다.

반드시 빈 생성자를 만들어주어야 사용할 수 있다.

# 2. Fragment 상태 유지 방법

Fragment는 다른 Fragment로 화면이 이동했을 때, 기존의 Fragment를 Destory한다. 그렇기때문에 이전 기록이 남아있지 않게된다.

onDestroy() 가 호출되는 여러가지 경우가 있다.

- 뒤로 가기 버튼 클릭
- Fragment 간의 이동
- 화면 가로/세로 전환
- 언어 설정

하지만 Fragment에 저장된 기록을 유실하고 싶지 않을 수 있다. 이를 위해 여러 Fragment 상태 유지 방법이 존재한다.

### 1) FragmentManager의 Fragment 재사용

FragmentManager에서 replace를 하게 되면 기존에 있던 Fragment는 삭제되고 새로운 Fragment가 생성되기 때문에 기존 데이터는 유지되지 못하지만, 기존의 Fragment가 생성되어 있다면, 이를 재사용하여 Fragment의 상태를 유지할 수 있다.

다음과 같이 Fragment의 화면을 이동할 때, replace transaction을 사용해서 화면을 이동하면, onDestroy() 까지 호출되어, Fragment가 사라지게 된다.

```kotlin
private FragmentManager fragmentManager;

@Override
protected void onCreate(@Nullable savedInstanceState: Bundle) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main);
    ButterKnife.bind(this);

    fragmentManager = getSupportFragmentManager();
    fragmentManager.beginTransaction().replace(R.id.fl_main, new A_Fragment()).commit();

}

@OnClick(R.id.btn_main_a) void onMoveA() {
    fragmentManager.beginTransaction().replace(R.id.fl_main, new A_Fragment()).commit();
}

@OnClick(R.id.btn_main_b) void onMoveB() {
    fragmentManager.beginTransaction().replace(R.id.fl_main, new B_Fragment()).commit();
}

@OnClick(R.id.btn_main_c) void onMoveC() {
    fragmentManager.beginTransaction().replace(R.id.fl_main, new C_Fragment()).commit();
}
```

위 replace를 add, show, hide를 사용하여 Fragment를 재사용할 수 있다.

로직은 다음과 같다.

- Fragment를 재사용하기 위해 변수로 선언한다.
- Fragment의 인스턴스가 생성되지 않았다면 인스턴스를 생성하고, add transaction을 통해 Fragment를 추가해준다.

add를 사용하면, 기존의 Fragment를 지우지 않고 그 위에 추가해주게 된다.

- show, hide transaction으로 원하는 Fragment만 보여줄 수 있다.

```kotlin
private lateinit var fa: Fragment
private lateinit var fb: Fragment
private lateinit var fc: Fragment

@Override
protected void onCreate(@Nullable Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main);
    ButterKnife.bind(this);

    fragmentManager = getSupportFragmentManager();

    fa = new A_Fragment();
    fragmentManager.beginTransaction().replace(R.id.fl_main,fa).commit();

}

@OnClick(R.id.btn_main_a) void onMoveA() {

    if(fa == null) {
        fa = new A_Fragment();
        fragmentManager.beginTransaction().add(R.id.fl_main, fa).commit();
    }

    if(fa != null) fragmentManager.beginTransaction().show(fa).commit();
    if(fb != null) fragmentManager.beginTransaction().hide(fb).commit();
    if(fc != null) fragmentManager.beginTransaction().hide(fc).commit();
}

@OnClick(R.id.btn_main_b) void onMoveB() {

    if(fb == null) {
        fb = new B_Fragment();
        fragmentManager.beginTransaction().add(R.id.fl_main, fb).commit();
    }

    if(fa != null) fragmentManager.beginTransaction().hide(fa).commit();
    if(fb != null) fragmentManager.beginTransaction().show(fb).commit();
    if(fc != null) fragmentManager.beginTransaction().hide(fc).commit();
}

@OnClick(R.id.btn_main_c) void onMoveC() {

    if(fc == null) {
        fc = new C_Fragment();
        fragmentManager.beginTransaction().add(R.id.fl_main, fc).commit();
    }

    if(fa != null) fragmentManager.beginTransaction().hide(fa).commit();
    if(fb != null) fragmentManager.beginTransaction().hide(fb).commit();
    if(fc != null) fragmentManager.beginTransaction().show(fc).commit();
}
```

### add() vs replace()

정리하다가 add와 replace의 차이점을 좀 더 알아봤다.

- replace()

fragmentTransaction.replace(containerViewId: Int, fragment: Fragment, tag: String)

컨테이너에 추가되어있던 기존의 Fragment를 바꾼다.

컨테이너 아이디와 대체할 Fragment, 옵션으로 태그를 넣는다.

이전에 추가되었던 모든 Fragment에 remove(Fragment 삭제)를 호출하고 새로운 Fragment를 추가한다.

![Untitled](https://user-images.githubusercontent.com/50517813/132989980-526e13de-72fd-4c1a-af3e-93f85a472aa7.png)

- add()

fragmentTransaction.add(containerViewId: Int, fragment: Fragment, tag: String)

replace()와 마찬가지로 컨테이너 아이디와 대체할 Fragment, 옵션으로 태그를 넣는다.

기존에 추가되었던 Fragment를 지우지않고 그 위에 추가한다.

![Untitled](https://user-images.githubusercontent.com/50517813/132989918-9b2eb006-63a9-43b3-b4f2-2d0aa5cbf71f.png)

### BackStack

BackStack을 사용하면 Fragment 전환을 유연하게 관리할 수 있다.

### 2) onSaveInstanceState(bundle: Bundle)

onDestroy() 호출 전에 실행되며 함수의 인 자에 key-value로 여러가지 데이터를 넣을 수 있다.

onSaveInstanceState 메소드를 오버라이드하여, SaveState에 저장할 값을 Bundle 형태로 저장한다.

```kotlin
// Bundle에 값을 저장할 땐 put~~() 를 사용한다.
override fun onSaveInstanceState(outState: Bundle) {
    super.onSaveInstanceState(outState)
    outState.putBoolean(IS_EDITING_KEY, isEditing)
    outState.putString(RANDOM_GOOD_DEED_KEY, randomGoodDeed)
}
```

그 후 onCreate(savedInstanceState: Bundle)에 인자로 전달한다.

```kotlin
// Bundle에 저장된 값을 사용할 땐 get~~() 를 사용한다.
override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    isEditing = savedInstanceState?.getBoolean(IS_EDITING_KEY, false)
    randomGoodDeed = savedInstanceState?.getString(RANDOM_GOOD_DEED_KEY)
            ?: viewModel.generateRandomGoodDeed()
}
```

아주 적은 값을 저장할 때 간편하게 사용할 수 있다.

onSaveInstanceState는 onStop이 호출되기 전에 호출되었는데, API 28 버전 이후부터 onSaveInstanceState() 함수가 더 늦게 호출되어 onStop()까지 FragmentTransaction을 안전하게 수행할 수 있게되었다.

![Untitled](https://user-images.githubusercontent.com/50517813/132990003-1db4746e-5282-4b7d-b1b6-4bde2f7149f1.png)

# 3. FragmentFactory

AndroidX부터 FragmentFactory를 사용하여 Fragment를 선언할 때 인자가 있는 생성자를 사용할 수 있다.

일반적으로 Fragment 생성은 기본 생성자를 통해 생성해야하고, 데이터 전달은 Bundle을 사용했다. 

하지만 FragmentFactory를 사용하면 Fragment를 생성할 때 Fragment가 기본 생성자뿐만 아니라, 다른 인자를 가진 생성자를 만들 수 있다.

현재 [androidX.fragment.app](http://androidx.fragment.app) 패키지에서 Fragment 문서를 보면 기존의 instantiate는 deprecated 되었고, FragmentFactory의 instatntiate를 사용하는 것을 권장한다고 한다.

### 사용법

FragmentFactory로 Int형 변수 하나를 인자로 갖는 생성자를 만들고 인스턴스를 만들어보자

먼저 생성자가 있는 Fragment를 만든ㄷ

```kotlin
class MyFragment(private val index: Int) : Fragment() {

    //...
}
```

instantiate 메소드를 오버라이드한 후 현재 인스턴스를 생성하려는 className에 따라 생성자를 다르게 호출하게끔 한다.

```kotlin
class MyFragmentFactory(private val index: Int): FragmentFactory() {

    override fun instantiate(classLoader: ClassLoader, className: String): Fragment {
        return when (className) {
            MyFragment::class.java.name -> MyFragment(index)
            else -> super.instantiate(classLoader, className)
        }
    }
}
```

그러면 Activity에서는 만들어준 생성자로 Fragment 생성이 가능하다.

```kotlin
override fun onCreate(savedInstanceState: Bundle?) {
    supportFragmentManager.fragmentFactory = MyFragmentFactory()
    super.onCreate(savedInstanceState)
    binding = DataBindingUtil.setContentView(this, layoutId)
    
    val fragment = supportFragmentManager.fragmentFactory.instantiate(classLoader, MyFragment::class.java.name)
    supportFragmentManager.commit {
        add(containerId, fragment, MyFragment.TAG)
        addToBackStack(null)
    }
}
```

유의할 점은 구성 요소 내에서 Fragment를 초기화하는 작업을 담당하는 FragmentFactory는 Fragment를 만들기 전에 설정해야 한다. 이것은 다음을 의미합니다.

구성 요소가 Fragement를 만들기 전에 FragmentTransaction을 사용하여 Fragment을 동적으로 추가하는 경우.

시스템이 Fragment을 복원하기 전에 구성 변경 또는 앱의 프로세스를 다시 시작한 후 Fragment을 재생성하는 경우.

onCreate()가 호출되기전에 FragmentFactory를 설정해준다.

### 항상 사용해야 하는가??

빈 생성자를 사용해야 하는 경우에는 굳이 FragmentFactory를 사용할 필요는 없다.

또한 koin같은 라이브러리를 사용하면 FragmentFactory를 직접 구현하지 않고 더 간편하게 의존성 주입을 처리할 수 있다.
