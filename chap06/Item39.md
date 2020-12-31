# [Item 39] 명명 패턴보다 애너테이션을 사용하라.

명명 패턴의 대표적인 예로 `JUnit 3`까지는 테스트 메서드 이름을 test로 시작하지 않으면 이 메서드를 그냥 지나쳤서 테스트를 통과했다고 오해하는 경우도 있었습니다.</br>
`JUnit 4`부터는 이러한 문제점을 해결하기 위해 **애너테이션**을 전면 도입했습니다. 이번 예제에서는 자동으로 수행되는 간단한 테스트용 애너테이션으로, 예외가 발생하면 해당 테스트를 실패로 처리합니다.

``` java
/**
 * 테스트 메서드임을 선언하는 애너테이션
 * 매개변수 없는 정적 메서드 전용
 */
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface Test {
}
```

`@Retention`, `@Target`과 같이 애너테이션 선언에 다는 애너테이션을 **메타애너테이션**이라 합니다. 쉽게 설명하면 메타 애너테이션은 애너테이션을 위한 애너테이션이라고 생각하시면 됩니다.</br>
`@Retention`은 애너테이션이 유지되는 기간이며 @Retention(RetentionPolicy.RUNTIME)은 런타임까지 존재한다는 뜻입니다.</br>
`@Target`은 적용 대상을 지정할 때 사용하며 @Target(ElementType.METHOD)은 메서드 선언에만 사용할 수 있다는 뜻입니다. 위 코드에서 "매개변수 없는 정적 메서드 전용"라고 주석을 작성했는데, 적절한 애너테이션 처리기를 직접 구현하지 않으면 컴파일 오류 없이 잘 작동합니다.
 </br>

 ``` java
 public class Sample {

  @Test
  public static void m1() {} // 성공해야 한다

  public static void m2() {} 

  @Test
  public static void m3() { // 실패해ㅑ 한다
    throw new RuntimeException("Boom");
  }

  public static void m4() {}

  @Test
  public void m5() {} // 잘못 사용: 정적 메서드가 아님

  @Test
  public static void m7() { // 실패해야 한다
    throw new RuntimeException("Crash");
  }

  public static void m8() {}
}
 ```
 @Test 애너테이션은 Sample클래스에 직접적인 영향을 주지 않습니다. 그저 이 애너테이션에 관심 있는 프로그램에게 추가 정보를 제공할 뿐입니다.

 ``` java
 // 마커 애너테이션을 처리하는 코드
 public class RunTest {

  public static void main(String[] args) throws Exception {
    int tests = 0;
    int passed = 0;
    Class<?> testClass = Class.forName(args[0]);

    for (Method m : testClass.getDeclaredMethods()) {
      if (m.isAnnotationPresent(Test.class)) {
        tests++;
        try {
          m.invoke(null);
          passed++;
        } catch (InvocationTargetException wrappedExc) {
          Throwable exc = wrappedExc.getCause();
          System.out.println(m + " failed: " + exc);
        } catch (Exception exc) {
          System.out.println("Invalid @Test: " + m);
        }
      }
    }
    System.out.printf("Passed: %d, Failed: %d%n", passed, tests - passed);
  }
}
 ```
@Test 애너테이션이 달린 메서드를 차례로 호출합니다. `isAnnotationPresent`메서드가 실행할 메서드를 찾아줍니다. `InvocationTargetException`외의 예외가 발생한다면 @Test 애너테이션을 잘못 사용했다는 뜻입니다.인스턴스 메서드, 매개변수가 있는 메서드, 호출할 수 없는 메서드 등에 사용했다는 뜻입니다. </br></br>
이번에는 특정 예외를 던져야만 성공하는 테스트를 지원하도록 해봅시다.
``` java
/**
 * 명시한 예외를 던져야면 성공하는 테스트 메서드용 에너테이
 */
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface ExceptionTest {
  Class<? extends Throwable> value();
}
```
`Throwable`을 확장한 클래스의 Class 객체라는 뜻이며, 따라서 모든 예외,오류 타입을 수용합니다. 이는 한정적 타입 토큰을 활용한 사례입니다. 다음은 이 애너테이션을 실제 활용한 모습입니다.

``` java
public class Sample2 {

  @ExceptionTest(ArithmeticException.class)
  public static void m1() {
    int i = 0 / 0;
  }

  @ExceptionTest(ArithmeticException.class)
  public static void m2() {
    int[] a = new int[0]; // 실패해야 한다. (다른 예외 발생)
    int i = a[1];
  }

  @ExceptionTest(ArithmeticException.class)
  public static void m3() { } // 실패해야 한다. (예외 발생 x)
}
```

이제 이 애너테이션을 다룰 수 있는 코드를 작성해봅시다. (앞서 작성한 코드의 일부분을 수정했습니다)
``` java
      if (m.isAnnotationPresent(ExceptionTest.class)) {
        tests++;
        try {
          m.invoke(null);
          System.out.printf("테스트 %s 실패: 예외를 던지지 않음%n", m);
        } catch (InvocationTargetException wrappedEx) {
          Throwable exc = wrappedEx.getCause();
          Class<? extends Throwable> excType =
              m.getAnnotation(ExceptionTest.class).value();
          if (excType.isInstance(exc)) {
            passed++;
          } else {
            System.out.printf(
                "테스트 %s 실패: 기대한 예외 %s, 발생한 예외 %s%n",
                m, excType.getName(), exc);
          }
        } catch (Exception exc) {
          System.out.println("잘못 사용한 @ExceptionTest: " + m);
        }
      }

```
문제 없이 컴파일이 되면애너테이션 매개변수가 가리키는 예외가 올바르단 뜻입니다. 예외를 하나가 아닌 여러개를 명시하고 그 중 하나가 발생하면 성공하게 만들 수도 있습니다.
</br>

``` java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface ExceptionTest {
  Class<? extends Throwable>[] value();
}
```

``` java
@ExceptionTest({ IndexOutOfBoundsException.class,
                NullPointerException.class }) 
public static void doSomething() {
    ...
}
``` 
다음은 클라이언트 코드를 수정한 모습입니다.
``` java
      if (m.isAnnotationPresent(ExceptionTest.class)) {
        tests++;
        try {
          m.invoke(null);
          System.out.printf("테스트 %s 실패: 예외를 던지지 않음%n", m);
        } catch (Throwable wrappedExc) {
          Throwable exc = wrappedExc.getCause();
          int oldPassed = passed;
          Class<? extends Throwable>[] excTypes =
              m.getAnnotation(ExceptionTest.class).value();
          for (Class<? extends Throwable> excType : excTypes) {
            if (excType.isInstance(exc)) {
              passed++;
              break;
            }
          }
          if (passed == oldPassed) {
            System.out.printf("테스트 %s 실패: %s %n", m, exc);
          }
        }
      }
```
자바 8에서는 여러 개의 값을 받는 애너테이션을 다른 방식으로 만들 수 있습니다. 배열 매개변수를 사용하는 대신 애너테이션에 `@Repeatabl` 메타 애너테이션을 다는 방식입니다.</br>
@Repeatabl를 사용 할 때 주의사항이 있습니다. @Repeatable을 단 애너테이션을 반환하는 '컨테이너 애너테이션'을 하나 더 정의하고, @Repeatable에 컨테이너 애너테이션에 class객체를 매개변수로 전달해야 합니다. 또 컨테이션 애너테이션은 내부 애너테이션 타입의 배열을 반환하는 value 메서드를 정의해아 합니다. 

``` java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
@Repeatable(ExceptionTestContainer.class)
public @interface ExceptionTest {
  Class<? extends Throwable> value();
}

// 컨테이너 애너테이션
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface ExceptionTestContainer {
  ExceptionTest[] value();
}

@ExceptionTest(IndexOutOfBoundsException.class)
@ExceptionTest(NullPointerException.class) 
public static void doSomething() {
    ...
}
```
다음은 RunTests 프로그램이 @ExceptionTest의 반복 가능 버전을 사용하도록 수정한 코드입니다.
``` java
      if (m.isAnnotationPresent(ExceptionTest.class)
          || m.isAnnotationPresent(ExceptionTestContainer.class)) {
        tests++;
        try {
          m.invoke(null);
          System.out.printf("Test %s failed: no exception%n", m);
        } catch (Throwable wrappedExc) {
          Throwable exc = wrappedExc.getCause();
          int oldPassed = passed;
          ExceptionTest[] excTests =
              m.getAnnotationsByType(ExceptionTest.class);
          for (ExceptionTest excTest : excTests) {
            if (excTest.value().isInstance(exc)) {
              passed++;
              break;
            }
          }
          if (passed == oldPassed) {
            System.out.printf("Test %s failed: %s %n", m, exc);
          }
        }

```
이번 Test예제에서는 애노테이션으로 할 수 있는 일들 중 극히 일부 입니다. 하지만 일반적으로 프로그래머가 애너테이션을 직접 정의할 일은 드뭅니다. 하지만 자바 프로그래머라면 자바가 제공하는 애너테이션 타입들은 사용할 줄 알아야합니다.