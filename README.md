# 주제

1. Activity, Fragment 차이
2. Lifecycle 차이점

    각 Lifecycle에 맞는 동작은 뭐가 있을까?

---

# Activity, Fragment 특징

## 1. Activity

안드로이드 앱을 만드는 주요 컴포넌트 중 하나로 사용자와 상호작용하기 위한 진입점이다.

UI를 포함하는 하나의 화면을 말한다.

안드로이드 앱은 대부분 여러개의 Activity로 이루어져있다.

안드로이드 시스템은 실행 중인 앱의 Activity Stack을 관리한다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/37276c17-bcb2-4edb-8bd1-eab066ef7bda/Untitled.png)

앱을 실행할 때 액티비티 #1을 실행시킨다고하면 Activity Stack에는 액티비티 #1이 담겨있다.

그 후 어떠한 작업(버튼 클릭 등)으로 액티비티 #2를 실행시키면 Activity Stack에 액티비티 #2를 push한다.

앱은 Activity Stack의 Top을 Foreground Activity로 두고 화면에 띄운다.

뒤로가기를 누르면 액티비티 #2가 pop되고 액티비티 #1이 화면에 띄워진다.

## 2. Fragment

layout을 정의할 수 있어 화면을 구성할 수 있다.

하지만 단일로는 사용할 수 없고 항상 Activity 내에서 호스팅되어야하고 생명주기가 Activity에 종속적이다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/c3bf38a0-21ed-4f24-a9d0-216885926cc1/Untitled.png)

Fragment간 이동은 Fragment Manager가 관리하고 메소드 호출에 의해 이동한다.

Fragment 역시 Back Stack을 사용할 수 있다. 이런 경우 현재 Fragment의 상태를 기억하고 다른 Fragment를 화면에 띄울 수 있다.

돌아올 때는 뒤로가기 등으로 돌아올 수 있다.

## 3. 왜 Fragment를 사용할까 (Activity와의 차이점)

Activity만 사용해서 충분히 앱을 구현할 수 있다.

하지만 이렇게 된다면 한 Activity에는 하나의 layout만 나타나게 된다.

우리는 Fragment를 사용해서 Activity에 여러 layout을 구현할 수 있다.

Activity는 Activity 내에 있는 Fragment를 바꿈으로써 사용자에게 보여줄 화면을 전환할 수 있다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/cd5991d5-c5de-4d1b-b706-37547b1d4bfa/Untitled.png)

또한 Fragment는 모듈로써 사용되기때문에 여러 Activity에 재사용할 수 있다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/622bd52a-69e7-4737-b972-fd64ab837883/Untitled.png)

위 기능들을 사용하면 Activity만으로 안드로이드 앱 개발한 것보다 효율적으로 할 수 있다.

# 생명주기

## 1. Activity

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/20834f0d-27c3-4c9d-a68e-f1fef4c20aed/Untitled.png)

### 1) onCreate : UI 준비

App을 처음 실행할 때

메서드에서 활동의 전체 수명 주기 동안 한 번만 발생해야 하는 기본 애플리케이션 시작 로직을 실행한다.

[주로 처리하는 작업]

- Widget을 inflate해 View 객체로 생성한 후 화면에 보여준다.
- 사용자와의 상호 작용을 처리하기 위해 Widget에 listener(이벤트 응답 처리 객체)를 설정한다.
- 멤버 변수를 정의한다.
- 외부의 모델 데이터를 연결한다.

### 2) onStart : UI 변경 시작

다중 창 모드에서 앱이 선택되는 경우

[주로 처리하는 작업]

- 애니메이션, 데이터 갱신

### 3) onResume : 현재 상호작용하는 Activity

오버뷰 화면에서 Activity를 선택하는 경우

하나의 Activity만 Resumed 상태일 수 있다.

Activity가 focus를 가진다.

 여기까지가 일반적으로앱이 실행될 때의 과정

---

### 4) onPause

사용자가 다중창 모드를 시작해서 다른 앱을 사용중인 경우

Dialog가 띄워졌을 때

사용자가 다중창 모드를 시작해서 다른 앱을 사용중인 경우

Dialog가 띄워졌을 때

### 5) : UI 변경 중단

홈버튼을 눌러 다른 앱을 사용할 때

onStop 호출 이후에 Activity는 종료시킬 대상이 된다.

[주로 처리하는 작업]

- 애니메이션, 데이터 갱신 중단, 데이터 저장

### 6) onDestroy : Activity 소멸

back 버튼을 탭해서 앱을 종료시킬 때

오버뷰 화면에서 해당 앱을 목록에서 제거할 때

[주로 처리하는 작업]

- 이전에 해제되지 않은 모든 리소스를 해제한다.

## 2. Fragment

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/f342714f-9b5d-47ff-bfa0-abe3a6c9f094/Untitled.png)

### 1) onCreate

Fragment Instance를 구성한다. (Model 등)

### 2) onCreateView

Fragment 뷰의 Layout을 inflate한다.

inflate된 뷰를 hosting activity에 반환한다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/0c992718-0063-4775-a153-2ad46bd2713c/Untitled.png)

- container : 위젯을 올바르게 구성하는 데 필요한 view의 부모
- false : inflate된 fragmente의 뷰는 Activity의 Container 뷰에 호스팅되어야하기때문에, 부모에 즉시 추가하지 않아야 한다.

### 3) onViewCreated

Activity에서 onCreate가 호출되고 난 후 호출되는 메소드

Fragment의 view는 모두 생성된 상태이다.

[주로 처리하는 작업]

onCreateView에서 생성한 view 객체로 작업을 처리한다.

- 변수의 초기값 설정
- 리스너 설정

### 4) onViewStateRestored

저장해둔 모든 state 값이 Fragment 의 View 계층구조에 복원 됐을 때 호출된다. 

[주로 처리하는 작업]

체크박스 위젯이 현재 체크 되어있는지 등 각 뷰의 상태값을 체크할 수 있다.

### 5) onStart

데이터가 설정될 때 작동되는 Listener를 설정한다.

→ 장치 회전 등으로 데이터가 복원된 후에 실행되어야하기 때문이다.

위 작업들은 Fragment를 FragmentManager에 추가할 때 호출된다.

---

### 6) onResume

사용자와 상호작용할 수 있는 상태일 때 호출된다.

Activity의 onResume가 동일하다.

### 7) onPause

Activity의 onPause와 동일하다.

### 8) **onStop()**

부모 액티비티나 프래그먼트가 중단됐을 때 뿐만 아니라, 부모 액티비티나 프래그먼트의 상태가 저장될 때도 호출된다.

### 9) **onDestroyView()**

모든 exit animation 과 transition 이 완료되고, Fragment 가 화면으로부터 벗어났을 경우 호출된다.

**해당 시점에서 가비지 컬렉터에 의해 수거될 수 있도록 Fragment View 에 대한 모든 참조가 제거되어야 합니다.**

### 10) **onDestroy()**

Fragment 가 제거되거나 FragmentManager 가 destroy 됐을 경우 호출된다.

## 생명주기의 중요성

Activity 상태 변화에 따라 발생하는 문제를 사전에 방지할 수 있다.

대표적인 예)

- **사용자가 앱을 사용하는 도중에 전화가 걸려오거나 다른 앱으로 전환할 때 비정상 종료되는 문제**

→ 전화나 다른 앱으로 전환될 때 onStop, 재개될 때 onResume이 실행된다.

- **사용자가 앱을 활발하게 사용하지 않는 경우 귀중한 시스템 리소스가 소비되는 문제(백그라운드에서 실행)**

→ onDestroy, onStop을 통해 문제되는 작업을 종료시킨다.

- **사용자가 앱에서 나갔다가 나중에 돌아왔을 때 사용자의 진행 상태가 저장되지 않는 문제**

→ 상태를 임시 저장해야 할 경우, onStop 이후에 onSavedInstanceState라는 메서드가 자동으로 실행된다.

→ 활동을 복구해야 할 때는 onrestoreInstance 메서드가 자동으로 실행된다.

- **화면이 가로 방향과 세로 방향 간에 회전할 경우, 비정상 종료되거나 사용자의 진행 상태가 저장되지 않는 문제**

→ onSvaedInstanceState로 해결할 수 있지만 회전의 경우 configChanges를 이용해 다시 시작되지 않게 사전 처리할 수 있다.
