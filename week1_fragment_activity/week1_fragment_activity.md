# Activity와 Fragment

## Activity

`Activity`는 안드로이드의 화면을 구성하는 요소입니다.

각각의 앱이 독립적으로 실행되는 데스크톱 앱과 달리 모바일 앱은 상호 연결성을 가지고 있어 각각의 화면을 다른 앱과 연결하여 사용 할 수 있습니다.

`Activity`는 이런 패러다임에 맞게 설계되었습니다. 각 앱이 다른 앱의 전체를 호출하는 것이 아니라 다른 앱의 `Activity`를 호출 함으로 써 앱의 연결성을 제공합니다.

그래서 안드로이드 앱은 앱의 생명주기와 별개로 각각의 `Activity` 생명주기를 가지고 있습니다.

## Fragment

`Fragmnet`는 재사용 가능한 UI를 만들기 위해 만들어진 클래스로 `Activity`만으로는 태블릿 같이 다양한 화면 크기를 지원하기 어려워 도입 되었습니다.

또  `Activity`와 `Fragment`에 종속되어야만 사용이 가능하고 각각의 `Fragment`도 자신만의 생명주기를 가지고 있습니다.

## Activity vs Fragment

### 화면 구성

일반적인 경우에 `Activity`는 한번에 한 가지 화면밖에 구성하지 못합니다. 하지만 `Fragment`는 `Activity`안에서 여러 `Fragment`를 구성할 수 있기 때문에 재사용성에 자유롭다고 할 수 있습니다. UI를 구성할때 일부 화면은 고정된채 다양한 화면을 교체해나가야 한다면 `Fragment`의 선택은 필수라고 생각합니다.

### 데이터 공유

`Activity`간의 데이터 공유는 `Bundle`을 통해서 이루어 지게 됩니다. `intent`를 통해 `Bundle`을 전달하기 때문에 직렬화를 거쳐 데이터를 전달하게 되고 메모리 사용에도 제약을 가지고 있습니다.

`Fragment`간의 데이터 공유는 `Activity`에 비해 간단한 편입니다. 같은 부모 `Activity`안에서 사용되기 때문에 `activityViewModel`이나 `Activity`의 멤버를 공유하면 손쉽게 데이터를 공유 할 수 있습니다.

### 시스템 자원

![image](https://user-images.githubusercontent.com/13197242/132123906-5320e14a-86ea-48d3-b52f-4dbf034fc06e.png)

`Actvity`는 화면을 보여주고 android 시스템 자원에 접근하기 위해 다양한 Class들을 상속하고 있습니다. 그렇기 때문에 많은 `Activity`를 사용하게 되면 많은 자원을 사용하게 됩니다. 하지만 `Fragment`는 `Actvity`에 종속되어있고 `Activity`의 자원을 공유하고 있기 때문에 가볍다고 말할 수 있습니다.

## Activity의 생명주기

![image](https://user-images.githubusercontent.com/13197242/132123900-fe7d61b9-59b7-456c-ba27-15e89829aa43.png)

## Activity 생명주기

Activity의 기본적인 생명주기는

`onCreate → onStart → onResume → onPause → onStop → onDestroy` 구성되어 있습니다. 생명주기와 별개로  `onRestart onSaveInstanceState`라는 콜백도 가지고 있습니다.

config change : 화면회전, 멀티윈도우 모드로 activity의 상태가 변하는 것

### onCreate

Activity가 생성될 때 호출되는 메서드로 `Activity`가 생성될 때 한 번만 동작해야 하거나 자원을 초기화해주는 역할을 담당합니다

이 생명주기에는 두 가지 상태가 존재합니다.

- 처음 Activity가 생성되었을 때
- process kill, config change

첫 번째 경우는 `savedInstanceState`가 null 일 때 맨 처음 생성되어 있기 때문에 `savedInstanceState`에 아무런 값이 들어있지 않습니다.

두 번째 경우는 `savedInstanceState`에 값이 들어있을 때인데 강제로 종료되거나 config change가 일어났기 때문에 저장해야 할 값을 `onSavedInstanceState`에서 처리해 주었거나 기본적으로 저장되는 내용들이 `savedInstanceState`에 들어있는 상태로 `onCreate` 상태가 시작됩니다.

### onStart

`Activity`가 시작되거나 Background에서 Foreground로 돌아올 때 호출되는 상태입니다. `onStop`에서 멈췄던 동작을 다시 시작할 필요가 있는 부분을 처리해야 합니다.

### onResume

`Activty`가 사용자와 상호작용을 할 수 있는 상태로 이 단계에 접어 들면 앱을 사용할 수 있게 됩니다. 다른 `Activity`가 화면을 가리거나 포커스를 잃기 전까지 이 상태를 유지합니다.

### onPause

`Activity`가 일부 가려지거나 포커스를 잃을 때 호출되는 생명주기로 GPS센서나 카메라같이 포커스를 가지고 있을 때 사용하는 자원을 일시 정지하는 단계입니다.

`onPause`는 매우 짧은 시간이기 때문에 네트워크나 데이터베이스를 사용하는 작업을 사용한다면 `onPause`안에 끝나지 않을 수 있어 `onStop`에서 처리를 권장합니다.

### onStop

`Activity`가 완전히 가려지거나 홈으로 이동해 `Activity`가 Background로 전환하는 상태로 화면에 보여지지 않기 때문에 시스템은 `Activity`를 메모리 상태로 전환합니다.

이 상태에서 다시 `onStart`로 가기 위해서 시스템은 `onRestart`를 호출하게 됩니다.

### onDestroy

Activity가 파괴될 때 호출되는 생명주기로 `Activity`를 종료하거나 config change가 일어날 때 호출된다. `Activity`에서 사용하는 모든 자원을 반납해야 하는 상태다

이 상태에서 `onCreate`가 호출되어 재생성 되지 않는다면`ViewModel`은 `onCleared`를 호출해 데이터를 정리해야 한다.

## Activity 생명주기와 관련된 추가적인 메소드

### onRestart

`Activity`가 Background에서 Foreground로 돌아올 때 호출하는 메서드로 `onStop`에서 `onStart`상태로 들어서기 위한 준비를 하는 단계라고 생각하면 된다.

### onSaveInstanceState

config change나 process kill의 상태에서 사용자의 데이터나 상태를 보존하기 위한 메서드로 필요한 정보를 bundle형태로 저장해 다시 `Activity`가 `onCreate`단계로 돌아올 때 불러 올 수 있도록 처리하는 단계이다.
