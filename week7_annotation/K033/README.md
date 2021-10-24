# 주제

Annotation

- annotation은 무엇인가, custom annotation 만들기
- annotation process 맛보기

# Annotation

코드에 부가 정보를 달기 위해 클래스, 함수, 프로퍼티 선언 앞에 추가하는 구문이다.

@ 기호로 시작하며 사용 목적은 다음과 같다.

- 컴파일러에게 코드 문법 에러를 체크하기 위한 정보를 제공
- 개발 툴이나 빌더에게 코드 자동 추가를 위한 정보 제공
- 실행 시 특정 기능을 실행하기 위한 정보 제공

## 종류

1. Kotlin/Android 에 내장되어 있는 built in annotation
2. Annotation에 대한 정보를 나타내기 위한 meta annotation
3. 개발자가 직접 만드는 custom annotation
    - Reflection
    - Code Generation

# Custom Annotation

위에서 작성한대로, 개발자가 annotation을 만드는 방법은 두 가지입니다. 하나는 Reflection을 사용하는 방법이고, 다른 하나는 Code Generation 방법입니다.. 이번에는 Reflection을 사용해서 annotation을 만들어보겠습니다.

## 1. 선언

일반 클래스와 동일하게 class라는 예약어를 사용햇 ㅓ선언하지만 annotation 예약어를 앞에 추가해 선언한다. 객체 생성은 물론 클래스 내에 프로퍼티나 함수를 추가할 수 없다.

```kotlin
annotation class TestAnnotation
```

위 custom annotation을 사용하려면 사용하려는 코드 앞(또는 위)에 `@TestAnnotation` 을 붙이면 된다. 함수나 프로퍼티는 물론 주 생성자, 프로퍼티의 getter/setter에도 추가할 수 있으며, 주 생성자에 어노테이션을 추가하려면 constructor 키워드를 함께 작성해야 한다.

```kotlin
annotation class TestAnnotation

class Test @TestAnnotation constructor() {
	@TestAnnotation
	val myVal: Int = 10
	
	var myVal2: Int = 10
		@TestAnnotation set(value) { field = value }

	// 람다에 어노테이션 추가
	val myFun = @TestAnnotation{
	}
}
```

하지만 위 annotation은 껍데기만 존재하고 아무런 기능을 하지 않는다. 따라서 위 annotation이 달린 곳을 특별하게 처리해주는 무엇인가가 필요하다.

## 2. 설정

### 데이터 설정

annotation에 특정 데이터를 설정하고 그 데이터를 사용할 수 있다. annotation 클래스는 구현체 `{}` 를 포함할 순 없지만, 생성자는 포함할 수 있다. 주 생성자만 포함할 수 있고, 주 생성자를 이용해 특정 데이터를 설정하는 annotation을 만들 수 있다.

```kotlin
annotation class TestAnnotataion(val count: Int)
    
class Test {
	@TestAnnotation(count=3)
	fun some(){
		println("some.....")
	}
}
```

생성자의 매개변수에는 꼭 `val`을 추가해야 한다. 주 생성자의 프로퍼티 선언에 사용할 수 있는 데이터 타입은 다음과 같다.

- 자바 기초 타입(Int, Long, etc)
- String
- Classes(Foo::class)
- enums
- 다른 annotation
- 위에 열거한 유형의 배열

```kotlin
annotation class TestAnnotation1(val count: Int)
    
annotation class TestAnnotation2(val otherAnn: TestAnnotation1, val arg1: KClass<*>)

class User

annotation class TestAnnotation3(val user: User) // Error!

@TestAnnotation1(10)
@TestAnnotation2(TestAnnotation1(20), String::class)
class Test { }

const val myData: Int = 10
@TestAnnotation1(myData)
class Test2 { }
```

annotation에서 다른 annotation이 주 생성자 프로퍼티를 지정할 때는 @기호를 추가하지 않는다.

그리고 다른 클래스를 annotation의 주 생성자 프로퍼티로 지정할 때는 타입을 KClass<>으로 지정해야 한다. KClass<>로 지정하면 컴파일러가 자동으로 전달받는 클래스 타입으로 변경한다.

annotation을 선언하면서 주 생성자를 명시했더라도, 주 생성자의 프로퍼티 타입이 클래스 객체 타입이라면 에러가 발생한다. (TestAnnotation3)

또한 데이터를 명시할 때 값이 아닌 변수로 명시할 수 있는데 그 변수는 꼭 const로 선언해야 한다.(변수가 아닌 상수를 의미)

## 3. 옵션

어노테이션은 클래스, 함수, 프로퍼티에 추가해서 이용할 수 있다. 그런데 어노테이션을 이용해 특정 영역으로 제한 할 수 있다. 즉, 어떤 어노테이션은 클래스에만 추가할 수 있고 함수나 프로퍼티에는 추가할 수 없게 제한할 수 있다.

- @Target: 어느 곳에 사용하는 어노테이션인지 설정(classes, functions, properties, expressions)
- @Retention: 어노테이션을 컴파일한 클래스에 보관할지, 런타임 때 리플렉션에 의해 접근할 수 있는지 설정(source, binary, runtime)
- @Repeatable: 이 어노테이션을 한 곳에 반복 사용 가능하게 설정
- @MustBeDocumented: 어노테이션을 API에 포함해야 하는지, API 문서에 포함해야 하는지 설정

@Retention은 특히 어노테이션을 어디까지 유지할 것인지에 대한 것이다. 기본값은 모든 곳에서 어노테이션을 유지하는 것이다. 만약 옵션 값이 source라면 컴파일된 클래스에는 어노테이션이 추가되지 않는다. 주로 컴파일러에 의해 어노테이션이 추가된 곳에 특정 코드가 자동으로 만들어지게 할 때 사용한다. 컴파일러에게 무엇을 어떻게 해달라는 정보 전달을 목적으로 하므로 컴파일이 완료된 코드에는 추가될 필요가 없는 것이다.

## 적용 대상 지정(Use-site Targets)

프로퍼티에 추가한 annotation이 자바로 변형될 때 getter 함수에만 추가하고 싶을 때가 있다. 이처럼 annotation이 추가되는 특정 대상을 지정하는 것을 '이용 측 대상(Use-site Targets)' 지정이라고 한다.

```kotlin
annotation class TestAnnotation

// getter 함수에만 Annotaion 추가 (Java 변환 시)
class Test {
	@get:TestAnnotation
	var no: Int = 10
}
```

코틀린에서 추가 대상을 지정할 수 있는 것은 다음과 같다.

- file - 코틀린 파일이 자바로 변형될 때 개발자가 정의하는 이름으로 변형되게 할 때 사용 (@file:JvmName(“MyTest”))
- property - 프로퍼티에 적용되는 이용 측 대상의 기본값. 나열한 이용 측 대상을 지정하지 않은 채로 프로퍼티의 어노테이션을 선언하면 자동으로 적용됨.
- field - 자바로 변형될 때 변수 선언에 어노테이션이 추가됨.
- get
- set
- receiver - 확장 함수나 프로퍼티에 추가됨.
- param - 생성자의 매개변수에 추가됨.
- setparam - 프로퍼티의 setter 함수 매개변수에 어노테이션이 추가됨.

추가로 여러 annotation을 줄 때 @get:[TestAnnotation TestAnnotation2]을 하면 된다.

참고 [https://colinch4.github.io/2020-11-29/어노테이션/](https://colinch4.github.io/2020-11-29/%EC%96%B4%EB%85%B8%ED%85%8C%EC%9D%B4%EC%85%98/)

# Custom Annotation 직접 만들기!

위에서 공부한 Custom Annotation 방법으로 간단한 annotation을 만들어보자!

## 목표

서로 협업하는 과정에서 누가 어떤 코드를 작성했고 수정했는지 알고싶어진 나는 작성자를 알 수 있는 Annotation을 만드려고 한다!

## 1. annotation 생성

```kotlin
import java.time.LocalDate
import kotlin.reflect.KClass

@Target(AnnotationTarget.CLASS, AnnotationTarget.FUNCTION)
annotation class Writer(val name: String, val date: String)
```

코드 작성자의 이름과 코드 작성 시각을 알기위해 annotation의 생성자에 name과 date를 준다.

또한 작성한 코드는 클래스와 클래스 내 메소드만 확인하고 싶기에 Target은 Class와 Function만 추가해준다.

### 2. 기능 추가

```kotlin
fun showWriterInfo(arg: Class<*>) {
    if (arg.isAnnotationPresent(Writer::class.java)) {
        val annotation = arg.getAnnotation(Writer::class.java)
        val name = annotation.name
        val date = annotation.date
        val methods = arg.methods

        Log.d("Writer", "작성된 클래스 ${arg.name.substringAfterLast(".")}, 작성자 : ${name}, 작성일 : $date")

        for(method in methods) {
            if (method.isAnnotationPresent(Writer::class.java)) {
                val annotation = method.getAnnotation((Writer::class.java))
                val name = annotation.name
                val date = annotation.date
                Log.d("Writer", "작성된 메소드 ${method.name}, 작성자 : ${name}, 작성일 : $date")
            }
        }

    }
}
```

클래스 또는 메소드에 Writer annotation이 있는지 확인하고, 있다면 클래스, 메소드 이름과, 작성자, 작성일을 출력한다.

### 3. 테스트 코드

```kotlin
@Writer("은석", "2021-10-25")
class A {
    init {
        showWriterInfo(this.javaClass)
    }
    @Writer("민석", "2021-10-25")
    fun foo1() {}

    @Writer("세진", "2021-10-25")
    fun foo2() {}
}

class B {
    init {
        showWriterInfo(this.javaClass)
    }

    @Writer("성환", "20201-10-23")
    fun foo1() {}

    @Writer("정헌", "2021-10-24")
    fun foo2() {}
}

@Writer("정헌", "2021-10-21")
class C {
    init {
        showWriterInfo(this.javaClass)
    }

    fun foo1() {}

    @Writer("성환", "20201-10-23")
    fun foo2() {}

    @Writer("민석", "2021-10-25")
    fun foo3() {}
}

class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        val a = A()
        val b = B()
        val c = C()
    }
}
```

### 4. 결과

![Untitled (1)](https://user-images.githubusercontent.com/50517813/138603434-14c8ab3c-8df0-4c62-85ad-4c4b391218d3.png)

Writer annotation을 달아준 클래스와 메소드의 정보만 출력되는 것을 확인할 수 있다!!
