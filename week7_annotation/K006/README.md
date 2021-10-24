### Index

- 어노테이션이 무엇인지?
- 코틀린 어노테이션을 사용하면서 자바 어노테이션과 다른 점?
- 주로 사용하는 코틀린 표준 어노테이션
- 어노테이션 프로세서 맛보기

# Annotation

애너테이션이란 소스코드 안에 다른 프로그램을 위한 정보를 미리 약속된 형식으로 포함시킨 것이다. 

코틀린 문서에서는 애너테이션을 코드에 대한 메타데이터라고 표현하고 있는 이유도 이와 같은 맥락이라고 할 수  있다.

<img src="https://user-images.githubusercontent.com/71161576/138603425-6c21117f-058c-4762-bfde-d713780c04b5.png" width="400">


예를 들어 테스트 코드를 작성할 때 우리는 `@Test` 애너테이션을 사용한다. 이는 이 애너테이션이 붙은 메서드를 테스트해야한다는 것을 테스트 프로그램에 알리기만할 뿐, 메서드가 포함된 프로그램에는 아무런 영향을 미치지 않는다. 

그리고 애너테이션은 반드시 해당 프로그램에 미리 정의된 종류와 형식으로 작성해야 의미가 있다. `@Test` 의 경우에도 이는 테스트 프로그램 이외의 다른 프로그램에서는 의미 없는 정보이다.

# Annotations in kotlin

코틀린 표준 라이브러리에 존재하는 애너테이션의 일부를 살펴보면서 애너테이션에 대한 감을 잡아보자.

이들은  컴파일 후의 코드 산출물에 영향을 미칠수도 있는 것으로, 컴파일러에 메타정보를 알려주는 역할을 한다.

## @JvmName

```kotlin
fun List<String>.filterValid(): List<String>

@JvmName("filterValidInt")
fun List<Int>.filterValid(): List<Int>
```

코틀린을 바이트코드로 변환 시, JVM 시그니처를 변경할 때 사용한다. 즉, 자바에서 호출되는 코틀린 클래스 및 메서드 이름을 변경할 때 사용이 된다.

## @JvmStatic

```kotlin
class C {
    companion object {
        @JvmStatic fun callStatic() {}
        fun callNonStatic() {}
    }
}

// Now, callStatic() is static in Java while callNonStatic() is not:

C.callStatic(); // works fine
C.callNonStatic(); // error: not a static method
C.Companion.callStatic(); // instance method remains
C.Companion.callNonStatic(); // the only way it works
```

자세한 설명은 아니지만, 코틀린에서 companion object 나 object 로 자바의 static 의 동작을 흉내내고 있다. 그래서 이들이 자바로 변환될 때 static 방식으로 사용될 수 있도록 변환하기 위해서 사용한다고 알아두자.

## @JvmOverloads

```kotlin
class Circle @JvmOverloads constructor(
	centerX: Int, centerY: Int, radius: Double = 1.0
) {
		@JvmOverloads fun draw(
			label: String, lineWidth: Int = 1, color: String ="red"
		) { /*...*/ }
}

// For every parameter with a default value, this will generate one additional overload

// Constructors:
Circle(int centerX, int centerY, double radius)
Circle(int centerX, int centerY)

// Methods
void draw(String label, int lineWidth, String color) { }
void draw(String label, int lineWidth) { }
void draw(String label) { }
```

생성자 또는 메서드에 기본값을 가지는 파라미터가 있는 경우에, 자바코드로 변환 시 이들이 존재하지 않는 모든 경우에 대해서 오버로딩 된 생성자 또는 메서드를 만들어준다.

## @Throws

코틀린에는 자바처럼 메서드 선언부에 예외를 throw 하는 구문이 존재하지 않는다. 그리고 이는 자바코드로 변환할 때도 생성되지 않는다. 이때 @Throws 를 사용할 수 있다.

```kotlin
@Throws(IOException::class)
fun writeToFile() {
    /*...*/
	throw IOException()
}
```

## Annotation Class

위에서 살펴본 애너테이션들을 사용할 수 있는 이유는 코틀린 표준 라이브러리에 애너테이션이 구현이 되어있기 때문이고, 이 말을 우리 또한 커스텀 에너테이션을 작성할 수 있다는 것을 의미한다.

우리도 애너테이션을 작성하고, 이 애너테이션을 사용하여 특정 메타 정보를 저장할 수 있다. 그리고 우리가 저장한 메타 정보에 따라서 특정 동작을 하도록 기능을 구현할 수도 있다.

우선 가장 기초가 되는 애너테이션 작성부터 알아보자.

```kotlin
annotation class MyAnnotation
```

애너테이션은 `annotation class`  라는 키워드로 만들 수 았고, 보다시피 이 또한 클래스라는 것을 알 수 있다.

여기까지만 작성을 해도 애너테이션은 만들어진 것이고, 이를 사용할 수 있다. 

```kotlin
class Test @MyAnnotation constructor() {
    	@MyAnnotation
    	val myVal: Int = 10
    	
    	var myVal2: Int = 10
    		@MyAnnotation set(value) { field = value }
    
    	// 람다에 어노테이션 추가
    	val myFun = @MyAnnotation {
    	}
}
```

하지만 단순히 저렇게 사용하게되면 우리가 필요로하는 메타 정보를 효과적으로 애너테이션을 통해서 전단하기 힘들다. 그래서 애너테이션을 효과적으로 사용할 수 있도록 다양한 정보를 추가할 수 있다. 

## Meta Annotation

메타 애너테이션은 애너테이션을 위한 애너테이션이라고 생각하면 된다. 메타 애너테이션을 사용하면 우리가 만드는 애너테이션에 다양한 정보를 추가할 수 있다. 

<img src="https://user-images.githubusercontent.com/71161576/138603488-d4580b77-1b0b-4742-a550-fa7b19c8a58b.png" width="600">


위의 애너테이션들은 우리가 만드는 애너테이션에 다양한 정보를 추가할 수 있는 메타 애너테이션이다. 

## @Target

우리가 만드는 애너테이션이 적용 가능한 대상에 대한 정보를 추가할 수 있다.
<img src="https://user-images.githubusercontent.com/71161576/138603546-f238534f-b867-45d7-84f7-2dddc1ad060c.png" width="600">


## @Retention

우리가 만드는 애너테이션 정보가 언제까지 대상에 존재하게 될 것인지에 대한 정보를 추가할 수 있다.

<img src="https://user-images.githubusercontent.com/71161576/138603565-50f4ba53-de56-47fa-a088-3ee424854f2c.png" width="600">
<img src="https://user-images.githubusercontent.com/71161576/138603578-a5b0e90f-47f4-4652-b4f4-a232513cf0df.png" width="600">

## @Repeatable

우리가 만드는 애너테이션 정보가 하나의 대상에 반복적으로 사용될 수 있는 지 결정할 수 있다.

<img src="https://user-images.githubusercontent.com/71161576/138603601-22bc098c-1b5f-4d6a-9bd9-35ceb5807f62.png" width="600">

## @MustBeDocumented

<img src="https://user-images.githubusercontent.com/71161576/138603629-4ae24efc-0024-44ab-bdb5-369ce838ab04.png" width="600">

애너테이션이 API의 일부분으로 문서화 하기 위해서 사용되어야 한다는 것을 나타낸다.

## Data in Annotation

애너테이션에는 데이터를 저장할 수 도 있는데, 일반 클래스의 형태와 조금 다른 방식으로 데이터를 저장한다. 

애너테이션도 클래스로 구현하지만, 클래스 바디 부분을 사용할 수 없다.

 
<img src="https://user-images.githubusercontent.com/71161576/138603649-b160a40b-8814-495c-9f88-cc201d301405.png" width="600">

그래서 애너테이션은 `주생성자`로 저장할 데이터를 정의할 수 있다. 

```kotlin
annotation class MyAnnotation(val data: String)

@MyAnnotation("hello!")
class Test{}
```

애너테이션에 저장할 데이터로 사용할 수 있는 타입은 다음과 같다.

- 기본형 타입(Int, Long, Double, ..)
- Strings
- Enums
- Classes(KClass) Ex) Test::class
- Other annotations
- Arrays of type listed above
- 모든 타입은 non-nullable 해야한다. JVM이 애너테이션 필드 값의 null 값 저장을 지원하지 않음.
- 그리고 Other annotations의 경우에는 @를 붙히지 않는다.

## Annotation with lambda

애너테이션은 람다식에 사용할 수도 있다. 이는 사실 람다식의 바디부분이 생성될 때 호출되는 `invoke()` 메서드에 지정되는 것이다.

```kotlin
val firstMeet = @MyAnnotation { test.sayHello(10) }
```

## **Annotation use-site targets**

우리가 프로퍼티나 생성자의 파라미터에 애너테이션을 사용하는 경우, 이는 자바코드의 관점에서는 그 역할이 여러가지가 될 수 있다. 

다음과 같은 경우에 애너테이션을 사용하였다.

```kotlin
class Example(@Ann val foo,    
              @Ann val bar,    
              @Ann val quux)   
```

이 경우에 이들은 생성자의 파라미터인 동시에 클래스의 파라미터 역할을 한다. 그렇기 때문에 애너테이션의 target이 모호해지게된다. 

이때 더 구체적인 target을 정의할 수 있는데 이것을 `use-site target` 이라고 한다. 사용의 관점에서 target을 바라보는 느낌이다.

```kotlin
class Example(
    @field:Annval foo,    // annotate Java field
	@get:Annval bar,      // annotate Java getter
	@param:Annval quux)   // annotate Java constructor parameter
```

이렇게 파라미터, 프로퍼티, 게터, 세터 등의 관점에서 구체적인 target을 정의할 수 있다. 

use-site target으로 제공되는 속성은 다음과 같다. 자세한 내용은 [코틀린 공식문서](https://kotlinlang.org/docs/annotations.html#annotation-use-site-targets)를 참고하자.

<img src="https://user-images.githubusercontent.com/71161576/138603736-b4be602c-ebf8-44c1-860a-00e3011e988a.png" width="600">

## Java Annotation In Kotlin

Java로 만들어진 애너테이션은 당연히 코틀린에서 완벽히 호환이 되고, 모두 사용할 수 있다. 

하지만 Java 에서 애너테이션에 데이터를 저장하는 방식이 코틀린과 다르기 때문에 이를 주의해야한다.

우리는 위에서 `주생성자의 파라미터` 형태로 애너테이션에 필요한 데이터를 정의하는 것을 알아봤다.

Java 에서는 애너테이션에 데이터를 저장하기 위해서는 `메서드`를 사용한다.

그래서 애너테이션에 저장할 데이터(파라미터)의 순서를 보장해주지 못하기 때문에, 반드시 파라미터 명(Java에서는 메서드명) 을 반드시 명시해야한다.

```java
// Java
public @interface Ann {
    int intValue();
    String stringValue();
}

// Kotlin
@Ann(intValue = 1, stringValue = "abc") class C
```

하지만 애너테이션에 사용되는 데이터가 단 하나인 경우에 이는 value 라는 이름을 사용하면, 메서드명을 명시하지 않고 사용할 수 있다.

```java
// Java
public @interface AnnWithValue {
    String value();
}

// Kotlin
@AnnWithValue("abc") class C
```

## Annotation Processor

[[Android] Annotation Processor 만들기 | 찰스의 안드로이드](https://www.charlezz.com/?p=1167)

[Annotation 안에서 무슨 일이 일어나는 거지? 1편](https://blog.gangnamunni.com/post/kotlin-annotation/)

[Annotation 안에서 무슨 일이 일어나는 거지? 2편](https://blog.gangnamunni.com/post/kotlin-annotation-codegeneration/)