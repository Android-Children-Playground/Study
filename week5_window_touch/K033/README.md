# 주제

안드로이드 화면에서 View는 어떻게 그려지고 이벤트처리를 수행하는가

1. window는 무엇일까?
- **dialog**, **bubble**, status bar, navigation bar
1. window에서 view까지 터치 과정
- 제스쳐, 멀티터치, **dispatchTouchEvent**, **nestedScroll**

# 화면을 구성하는 요소들과 크기 순서

Window > Surface > Canvas > View

## 1. Window

안드로이드에서 화면을 구성하는 최대 단위로 일반적으로 Activity가 Window를 가지게 된다.

다음 사진에서 사각형으로 나뉘어진 부분이 안드로이드 화면에서 나뉜 Window이다.



하나의 화면 안에서는 여러개의 Window를 가질 수 있고 이러한 Window들은 WindowManager가 관리한다.

보통 하나의 Surface를 가지고 있으며, Application은 Window Manager와 상호작용하여 Window를 만든다.

WindowManager는 각각의 Window 표면에 Componenet를 그리기 위해 Surface를 만들고, Application은 Surface에 원하는 것을 그린다.

## 2. Surface

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/0712b5a1-412c-4346-b0e9-6102d83fe10f/Untitled.png)

Window는 자신만의 Suface를 가지는데, Surface는 화면에 합성되는 픽셀을 고정하는 객체입니다.

Surface Flinger가 여러 소스로부터 그래픽 데이터 버퍼를 받고, 그것들을 합성해서 Display로 보냅니다.

## 3. Canvas

Canvas는 모든 Draw 방법을 포함하는 클래스이다.

원이나 사각형 등을 그리는 방법에 대한 모든 로직이 Canvas에 포함되어있다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/0279f7c8-86d3-41ef-ae38-a7fec8dbc879/Untitled.png)

## 4. View

View는 Window 내부의 대화형 UI component이다.

Window에는 단일 뷰 계층 구조가 연결되어  Window의 모든 동작을 제공한다.

window가 다시 그려질때마다 Surface는 locked되고 canvas를 리턴한다.

계층을 따라 canvas는 각각 view를 거치게된다.

view들은 각자 순서에 각자 필요한 부분을 canvas에 그리게 된다.

그리는 과정이 끝나면 Surface를 unlocked하고 posted 된다.

이 부분은 3~4주차 과제에서 Canvas 구현하기 부분에서 확인할 수 있었다.

### 참고

[https://velog.io/@haejung/Android-Window-View-Surface-Canvas-Bitmap](https://velog.io/@haejung/Android-Window-View-Surface-Canvas-Bitmap)

# Android Window 예시

## 1. Dialog

사용자에게 결정을 내리거나 추가 정보를 입력하라는 메시지를 표시하는 작은 창

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/5b8f845f-1355-4ce1-8ca8-86ed0be7893b/Untitled.png)

## 2. Bubble

도움말 풍선은 알림 시스템에 내장되어 있으며, 버블을 펼쳐 앱 기능과 정보를 살펴볼 수 있고 사용하지 않을 때는 접을 수 있다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/6c1c3add-23ba-4b62-b78d-8d4b1951b0ae/Untitled.png)

# window에서 view까지 터치 과정

Android에서 Activity는 사용자와 상호작용할 수 있는 구성요소이다. 여러 상호작용 중에 기본적인 상호작용으로는 터치 이벤트가 있다.

onTouchEvent()나 OnClick() 등 터치와 관련된 이벤트를 수행하는 코드를 짜봤음에도 안드로이드가 터치 이벤트를 내부적으로 어떻게 처리하는지에 대해서는 잘 모른다.

Button 처럼 터치와 클릭 이벤트가 공존하는 경우, 우선적으로 처리하는 이벤트는 무엇이며 어떤 기준인지 알아볼 필요가 있다.

## 안드로이드 터치 이벤트의 흐름

사용자가 기기를 터치했을 때, 안드로이드 앱은 어디서부터 이벤트를 받아서 처리할까?

먼저 터치를 이벤트를 처리하기 전에 터치를 감지해야한다.

터치가 발생하면 제일 먼저 Activity가 알게되고 이어서 Viewgroup > View 순으로 Notify된다.

반대로 이벤트 처리는 View > ViewGroup > Activity 순으로 처리된다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/cf012f89-fa71-4e72-b838-a9ef04276f4c/Untitled.png)

Activity에서 터치 신호를 받게되면 바로 `onTouchEvent` 하지 않고 ViewGroup으로 `dispatch`한다.

ViewGroup 역시 View에게 `dispatch`한다. View는 더이상 `dispatch`할 대상이 없기때문에 `dispatchTouchEvent()` 메소드 안에서 `onTouchEvent()`를 실행한다.

View가 `onTouchEvent()` 의 반환으로 `true` 를 리턴하면 상위 ViewGroup이나 Activity는 `onTouchEvent()`를 수행하지 않는다. 반대로 false를 리턴하면 상위 ViewGroup의 `onTouchEvent()`가 실행된다.

## onInterceptTouchEvent()

터치가 발생했을 때, 하위 대상에 알리지 않고 자기선에서 처리하고 싶을 때 사용하는 메소드이다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/bb6f9f0b-bad7-42dc-9e4e-7ba82b0ab7b8/Untitled.png)

예를 들어, ViewGroup이 하위 View에게 전달되어야할 터치 신호를 보내지않고 Intercept 할 경우, View는 터치이벤트를 실행하지 않을 것이다. 그리고 ViewGroup의 `onTouchEvent()` 가 실행될 것이다.

## 연속된 TouchEvent의 처리

드래그와 같은 연속된 터치이벤트가 발생시, 이벤트가 dispatch되지 않고 생략될 수 있다.

드래그는 누르기 > 움직이기 > 떼기 의 작업이 필수적이기때문에 차례대로 `ACTION_DOWN` > `ACTION_MOVE` > `ACTION_UP` 이벤트가 연속 발생한다.

첫 번째 이벤트인 `ACTION_DOWN` 일 때, Activity에서 touchEvent를 `true`로 리턴한다고 가정하면(나머지는 `false` 리턴),

사용자의 `ACTION_DOWN` 터치는 View까지 `dispatch` 되어 `onTouchEvent` 발생 후 Activity에 와서야  `true` 값을 리턴 받는다.

이렇게되면, 터치에 대한 관심은 Activity만 가지고 있다고 할 수 있고, 그렇게되면 뒤에 이어지는 `ACTION_MOVE` > `ACTION_UP` 은 굳이 View까지 `dispatch` 되지 않고 Activity에서 모든 일을 처리하게 된다.

## 이미지로 보는 터치 이벤트 처리 과정

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/fb78ed00-16c6-46f2-94a3-d27ce2292109/Untitled.png)
