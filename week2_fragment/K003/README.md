Fragment 상태전환 라이프사이클(부록1 Viewbinding fragment 메모리릭 문제)

- Fragment 생성자를 변경하면 안되는 이유

- Fragement 상태 유지 방법 argument, savedInstancestate

- Navigation Component, Transaction 뭘 고를까 차이가 뭘까

- FragmentFactory 이게 뭘까 구글이 왜 만들었을까

  

## Fragment의 생명주기 

<img src="https://miro.medium.com/max/694/1*ALMDBkuAAZ28BJ2abmvniA.png" alt="img" style="zoom:75%;" />

`Fragment`의 생명주기는 `Activity`와 비슷하면서도 약간의 차이가 있다. 차이는 간단하게 `Activity`에서 `onAttach`, `onCreateView`, `onDestroyView`, `onDetach`가 추가된 정도이다. 

이러한 콜백들이 생명주기에 추가된 이유는 뭘까? 간단한 예제로 실제 생명주기를 확인하며 알아보도록하자

아래 생명주기는 MainActivity -> HomeFragment(Add) -> DashBoardFragment(Replace) ->  DashBoardFragment(Pop) ->  HomeFragment(backstack Restore) 순으로 진행된다

#### Fragment 생성 

<img src=".\res\week2\fragment생명주기생성.png" alt="fragment생명주기생성" style="zoom:50%;"/>


먼저 `Activity`와 `Fragment`가 처음 생성될때이다 `Activity`의 `onCreate`와 함께 `Fragment`가 `onAttach`되고 `onStart`까지 진행해 시작될준비를 마치고 `Activity`의 `onResume`과 동시에 시작된다

#### Fragment 전환

이때 다른 `Fragment`로 전환되면 어떻게 될까?

<img src=".\res\week2\fragment생명주기교체.png" alt="fragment생명주기교체" style="zoom:50%;"  />

화면에 보이고 있던 `HomeFragment`는 `onStop`까지 실행되고 `DashboardFragment`는 처음 생성되는 `onAttach` ~ `onResume` 순서로 실행된다 (이때 `Activity`와 달리 `onResume`전에 `onDestroyView`가 실행) 

한마디로 새로운 `Fragment`가 생성되면 이전 `Fragment`는 View만 파괴된채 그대로 메모리에 남아있는다 이는 BackStack에 저장되어 다시 복원될때 사용하기 위함이다

이제 DashboardFragment를 백키를 통해 제거하면  

#### Fragment 제거 및 복원

<img src=".\res\week2\fragment생명주기파괴.png" alt="fragment생명주기파괴" style="zoom:50%;"  />

다시 `DashboardFragment`는 `onStop`까지 호출되고 `backstack`에 저장되어있던 `HomeFragment`는 새로운View를 생성하고 시작된다 그리고 backstack`에서` 사라지는 `DashboardFragment`는 `onDetach`까지 호출되며 메모리 상에서 사라진다 .

위 예제에서 볼수 있듯 Fragment에 새로운 생명주기들이 추가된이유는 Activity내에서 FragmentManager를 통해 Fragment를 관리하고 생성 복원하기 위함을 유추 할 수 있다.

#### Fragment와 Activity 생명주기의 연관

`Fragment`는 `Activity`의 생명주기에 종속되어있다고 말하는데 실제로 그럴까?

새로운 Activity 생성

<img src=".\res\week2\새로운 Activity.png" alt="새로운 Activity" style="zoom:60%;"/>

새로운 Activitiy 파괴

<img src=".\res\week2\새로운Activity 종료.png" alt="새로운Activity 종료" style="zoom:60%; " />

이전 예제에서 Fragment의 생성파괴는 Activity에 어떠한 영향을 주지 않았다. 하지만 Fragment는 Activity의 생명주기를 따라가고 있는것을 볼 수 있다.



### FragmentManager

![FragmentActivity](.\res\week2\FragmentActivity.png)

<img src=".\res\week2\getSupportFragmentManager.png" alt="getSupportFragmentManager" style="zoom:65%;" />

Fragment의 생성과 관리는 FragmentManager를 통해 동작하는데 이는 Activity가 아닌 FragmentActivity가 가지고 있는 객체이다 그러므로 Fragment를 사용하려면 FragmentActivity를 상속하고 있는 Activity에서 사용해야 한다.

~~Activity FragmentManager의 자세한내용은 너무길어져 생략~~

Fragment를 관리하는 FragmentManager는 최초 FragmentActivity에서 생성되는데 Fragment에서는 fragmentManager가 4개나 존재한다

<img src=".\res\week2\fragmentmanager.png" alt="fragmentmanager" style="zoom:80%;" />

##### getFragmentManager(), getRequireFragmentManager() 

deprecated된 초기 `fragmentManager` getter이다 현재 남아있는 코드에서 `getFragmentManager()`는 nullable한 멤버변수`mFragmentManager`를 가져오고 `getRequireFragmentManager()`는 `getParentFragmentManager()`를 호출해 결과를 반환한다  

##### getParentFragmentManager()

`getParentFragmentManager()`는 간단한 Null check를 한 뒤 멤버변수 `mFragmentManager`를 반환한다.

##### getChildFragmentManager()

<img src=".\res\week2\fragmentManager생성.png" alt="fragmentManager생성" style="zoom:67%;" />

Fragment가 생성될때 같이 생성된 childFragmentManager를 반환한다.

##### mFragmentManager 너는 어디서 오니?

mChildFragmentManager같은 경우는 Fragment가 생성될때 멤버로 같이 생성된다 하지만 mFragmentManager는 Fragment내에서는 사용하는 곳만 존재할뿐 어디서 오는지 알수 없다 

 <img src=".\res\week2\fragmentManageraddFragment.png" alt="fragmentManageraddFragment" style="zoom:55%;" />

해답은 간단하다 FragmentManager에서 fragment를 생성하고 넣어주는 메소드에서 자신을 주입해준다. 이는 부모자식 관계로 구성하기 위함으로 생각된다. 

### Fragment Transaction

Fragment Transaction은 크게 3가지로 구성되어있다. 

#### Add



#### Replace



#### Show, Hide



#### Attach, Detach





#### 부록1. Fragment의 생성자를 오버로딩하면 안되는 이유?

EmptyActivity와 달리 BlankFragment는 주렁주렁 이상한 코드들이 같이 생성된다 

<img src=".\res\week2\blankFragment.png" alt="blankFragment" style="zoom:33%;" />

자동으로 생성된 코드의 주요내용은 Fragment를 생성하는 argument를 이용한 newInstance의 메소드가 대부분이다. 구글은 왜 이렇게 긴 코드를 기본으로 넣었을까?

그 이유를 알아보기 위해 Fragment를 생성할때 newIntance를 사용하지 않고 생성자에 arugment를 넘겨 보았다. 

<img src=".\res\week2\fragment클래스.png" alt="image-20210912035652123" style="zoom:80%;" />

그 결과 이런 에러와 함께 앱이 종료되었다.

<img src=".\res\week2\fragment생성자.png" alt="fragment생성자" style="zoom:80%;" />

could not find fragment constructor라는 에러메시지가 나타나는데 이 에러를 배출하는 instantiate를 한번 살펴보자

이중에 getConstructor를 살펴보면 리플렉션의 메소드인것을 알수 있다 (클래스에 정의된 생성자를 반환하는 메소드) 

<img src=".\res\week2\reflection.png" alt="reflection" style="zoom:80%;" />

그리고 newInstance()를 통해 새로운 Fragment객체를 생성하는 것을 볼수 있다 이때 우리가 생성자를 변경했다면 newInstance()를 통해 클래스를 생성할때 매개변수가 없는 생성자는 존재하지 않아 NoSuchMethodException을 발생하게 되고 앱이 종료되는 것이다.

그렇다면 구글은 왜이런 방식을 선택했을까?

그것은 안드로이드 앱은 언제든지 메모리에 저장되고 파괴되었다 복원될 수 있어야하기 때문이다

Fragment는 가볍다고 하지만 다양한 정보들을 가지고 있다 언제 복원되어야 할지 모르는 데이터들을 모두 저장해두기보단 데이터 로드를 위한 일부 인자만 시스템에 저장하고 인자를 통해 다시 Fragment를 생성하는게 효율적이라고 판단했다고 생각된다.

이때 상태저장에 사용되는 argument는 메모리에 저장되지만 saveinstancestate는 디스크에 직렬화 되어 저장된다고 한다. 



#### 부록2. ViewBinding 메모리릭 이슈
<img src=".\res\week2\activityViewbinding.png" alt="activityViewbinding" style="zoom:50%;" />

<img src=".\res\week2\fragmentViewbinaing.png" alt="fragmentViewbinaing" style="zoom:50%;"/>

새 프로젝트를 생성하고 Fragment에서 ViewBinding을 사용하는 코드가 생성될때 Activity와 다르게 이상한 코드가 생겨있고 이상한 주석도 생겨있다.

Backing Property로 binding 변수를 사용하고 onDestroyView에서 null로 없애 주는 모습을 볼수 있는데 

그 이유는 onDestroyView에서 View를 파괴하는데 binding 객체가 View를 가지고 있다는 점이 문제가된다. Fragment는 앞선 예제에서 BackStack에 저장되고 차례가 되면 복원되는 과정이 존재함을 확인했다.  이때 BackStack상태인 Fragment의 View는 파괴되었지만 binding 객체가 참조를 가지고 있기때문에 GC가 정상적으로 수행되지 못하고 계속 View에대한 참조를 유지하고 있는 것이다.

해결책은 나와있다. 아주 간단하게. onDestroyView에서 binding에 대한 참조를 풀어주면 된다. 그런데 모든 Fragment에서 이 코드를 전부 넣어줘야 할까? 

https://github.com/androidbroadcast/ViewBindingPropertyDelegate

다른 개발자가 이것을 간단하게 해주는 라이브러리를 만들어 두었다. 알고 쓰자!

끝!



