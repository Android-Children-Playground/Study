### Annotation 

우리는 Java기반으 개발을 진행할 때 annotation을 자주 보게 된다. @Override, @Deprecated 같이 기본적으로 제공하는 annotation은 물론 android에서 사용되는 @RequiresApi 같이 다양한 annotation이 존재한다. 

```java
@RequiresApi(Build.VERSION_CODES.M)
public void onViewCreated(view: View, savedInstanceState: Bundle?) {
    super.onViewCreated(view, savedInstanceState)
}
@Deprecated
public void startActivityForResult(@SuppressLint("UnknownNullness") Intent intent)
}
```

그렇다면 annotation은 정확히 뭘까? 

annotation의 단어의 뜻은 주석으로 해석하자면 소스코드에 다는 주석(설명)으로 Java 소스코드에 추가 하여 사용 할 수 있는 일종의 MetaData다 

```java
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target(value={CONSTRUCTOR, FIELD, LOCAL_VARIABLE, METHOD, PACKAGE, PARAMETER, TYPE})
public @interface Deprecated {
}
```

이런 annotation을 만들기 위해서 사용하는 Meta Annotation은 아래 5개로 이를 조합해 새로운 Custom Annotation을 만들 수 있다.

#### @Retention

컴파일러가 annotation을 다루는 방법을 정하고, 어떤 시점까지 영향을 미칠지 결정한다.

- RetentionPolicy.SOURCE : 컴파일 전까지만 사용

- RetentionPolicy.CLASS : 컴파일러가 클래스를 참조할 때까지 유요

- RetentionPolicy.RUNTIME : 컴파일 이후에도 JVM에 의해 계속 참조 가능

#### @Target

annotation이 적용되는 위치를 정하는 것으로

- ElementType.PACKAGE : 패키지
- ElementType.TYPE : 타입
- ElementType.ANNOTATION_TYPE : 어노테이션 타입
- ElementType.CONSTRUCTOR : 생성자
- ElementType.FIELD : 멤버 변수
- ElementType.LOCAL_VARIABLE : 지역 변수
- ElementType.METHOD : 메서드
- ElementType.PARAMETER : 전달인자 
- ElementType.TYPE_PARAMETER : 전달 인자
- ElementType.TYPE_USE : 타입 사용

#### @Documented

annotation을 javadoc에 포함시킨다

#### @Inherited 

annotation의 상속을 가능하게 한다.

#### @Repeatable

Java8 부터 지원하며, 연속 적인 annotation을 선언할 수 있게 해준다.



### Custom Annotation

Custom Annotation을 구현 하기 위해서는 3가지 단계가 필요하다

1. Annotation Interface만들기
2. Annotation Handler 만들기
3. Handler를 이용해 Instance 생성

새로운 Custom Annotation을 만들수 있다.

간단하게 숫자를 대신 넣어주는 새로운 Annotation을 만들어 보자  

#### Annotation Interface

먼저 첫번째로 Annotation Interface를 만들어야 하는데 

위에서 언급한 @Retention, @Target, @Documented, @Ingerited, @Repeatable을 조합하면 간단하게 만들 수 있다.

```java
@Target(ElementType.FIELD)
@Retention(RetentionPolicy.RUNTIME)
public @interface InjectNumber {
    int num() default 0;
}
```

```java
class Number {
    @InjectNumber(num = 10)
    int data;
}
```

annotation interface를 만들고 Number class에 새롭게 만든 annotation을 적용 한다.



#### Annotation Handler

사실 Number class에 설정한 @InjectNumber와 Annotation Interface는 Handler를 사용하게 하는 마킹에 불가하고 실제로는 Annotation Handler가 실질적인 처리를 맡는다

```java
class AnnotationHandler {
    private <T> T checkAnnotation(T targetObj, Class annotationObj) {
        Field[] fields = targetObj.getClass().getDeclaredFields();
        for (Field f : fields) {
            if (annotationObj == InjectNumber.class) {
                return checkAnnotationInsertInt(targetObj, f);
            }
        }
        return targetObj;
    }

    private <T> T checkAnnotationInsertInt(T targetObj, Field field) {
        InjectNumber annotation = field.getAnnotation(InjectNumber.class);
        if (annotation != null && field.getType() == int.class) {
            field.setAccessible(true);
            try {
                field.set(targetObj, annotation.num());
            } catch (IllegalAccessException e) {
                System.out.println(e.getMessage());
            }
        }
        return targetObj;
    }

    public <T> Optional<T> getInstance(Class targetClass, Class annotationClass) {
        Optional optional = Optional.empty();
        Object object;
        try {
            object = targetClass.newInstance();
            object = checkAnnotation(object, annotationClass);
            optional = Optional.of(object);
        } catch (InstantiationException | IllegalAccessException e) {
            System.out.println(e.getMessage());
        }
        return optional;
    }
}
```

단계를 요약하면 

1. checkAnnotation에서 reflection으로 annotation class를 확인하고 각 annotation에 맞는 처리를 진행한다
2. annotation에 대한 처리를 checkAnnotationInsertInt에서 동작한다 
3. 각 annotation에 대한 처리를 마친후 생성된 instance를 반환한다.



#### Handler를 이용해 Instance 생성

```java
public class Main {

    public static void main(String[] args) {
        AnnotationHandler handler = new AnnotationHandler();
        Number num = handler.getInstance(Number.class, InjectNumber.class)
                .map(o -> (Number) o)
                .orElse(new Number());
        Number num2 = new Number();
        System.out.println(num.data);
        System.out.println(num2.data);
    }

}

> Task :Main.main()
10
0
```

Annotation도 사실은 마법같이 작동하는 것이 아니라 내부적으로 개발자가 모든 부분을 처리해 주는 것을 알수 있다. 그리고 간단히 초기화 시점에 숫자를 넣어줄 뿐인데도 Reflection이나 복잡한 처리를 거쳐야 annotation을 만들 수 있음을 알수 있었다.



### Annotation Processer

![img](https://www.charlezz.com/wordpress/wp-content/uploads/2019/04/charles-2019-12-21-%EC%98%A4%ED%9B%84-4.57.09-1024x575.png)

Annotation Processer는 Java 컴파일러의 플러그인의 일종으로 annotation에 대한 검사 수정 생성하는데 사용됩니다. 그리고 일반 annotation 처리에 비해 빠르고, 리플렉션을 사용하지 않고, 보일러 플레이트를 생성해주는 기능을 가지고 있습니다.

한마디로 Annotation을 좀 더 잘 활용 하기 위한 플러그인으로

이를 통해 Retrofit, Room 같은 라이브러리가 우리 대신 코드를 생성해주고 편하게 사용 할 수 있게됩니다.



### Android Annotation

android 에도 기본적으로 제공하는 다양한 annotation이 존재하는데

자주 사용하는 것으로

`@StringRes` : string resId

`@DrawableRes` : drawable resId 

`@IdRes` : id 값

`@ColorRes` : color resId

`@LayoutRes` : layout resId

처럼 Resourece Type을 지정해 다른 Type의 매개변수나 값을 사용하게 되면 Error를 보여주고

`@AnyThread` : 모든 Thread에서 호출가능

`@MainThread(@UiThread)` : MainThread에서만 호출가능

`@WorkerThread` : Worker Thread에서만 호출가능

처럼 Thread관련 annotation도 존재한다

마지막으로

@CheckResult : 리턴값을 사용하는지 여부 확인

@RequiredApi : 지정된 Api레벨 이상에서만 호출이 가능하다

@RequirePermission : 권한이 필요함을 나타낸다.

등등의 annotation도 존재하므로 Android를 개발하면서 유용한 annotation이 많으니 잘 활용 하면 좋을 것 같다



