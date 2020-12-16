# [Item 17] 변경 가능성을 최소화하라.

**불변** 클래스(Immutable Class)란 말 그대로 객체가 생성된 후에 더이상 값을 변경할 수 없는 것을 의미합니다. 자바에서는 대표적으로 `String`, `Integer`, `Float`,`Long` 등이 있습니다.</br>
**불변 클래스의 장점**

### 클래스를 불변으로 만들기 위한 규칙
- 객체의 상태를 변경하는 메서드를 제공하지 않습니다.
- 클래스를 확장할 수 없도록 합니다.
- 모든 필드를 final으로 선언합니다.
- 모든 필드를 private로 선언합니다.
- 자신 외에는 내부에 가변 컴포넌트에 접근할 수 없도록 합니다.

``` java
public final class Calculator {
  private final int x;
  private final int y;

  public Calculator(int x, int y) {
    this.x = x;
    this.y = y;
  }

  public Calculator plus(Calculator c) {
    return new Calculator(x + c.x, y + c.y);
  }

  public Calculator minus(Calculator c) {
    return new Calculator(x - c.x, y - c.y);
  }

  ... 생략
}
```
여기서 주목할 점은 메서드들이 인스턴스 자신을 수정하지 않고 새로운 `Calculator` 인스턴스를 만들어 반화하는 것입니다. 
### 불변 객체의 장점
- 불변 객체는 Thread-Safe 하므로 멀티 쓰레드 환경에서 안전하게 사용할 수 있습니다.
- 불변 객체는 하나의 상태만을 갖고 있으므로 데이터를 신뢰할 수 있습니다.

따라서 불변 클래스라면 한번 만든 인스턴스를 최대한 재활용 하길 추천합니다. 재활용 하기 가장 쉬운 방법은 자주 쓰이는 값들을 상수로 제공하는 것입니다.
``` java
public static final Calculator ONE = new Calculator(1, 0);
...
```
이 방식을 더 살펴보면 불변 클래스는 자주 사용되는 인스턴스를 캐싱하여 같은 인스턴스를 중복 생성하지 않게 해주는 정적 팩터리를 제공할 수 있습니다. 대표적으로 박싱된 기본 타입 클래스가 있습니다.</br>
이런 정적 팩터리를 사용하면 여러 클라이언트가 인스턴스를 공유하여 메모리 사용량과 가비지 컬렉션 비용이 줄어듭니다. 하지만 붋변 객체에도 **단점**이 있습니다.**값이 다르면 반드시 독립된 객체로 만들어야** 합니다. (성능 이슈)
### 불변 클래스를 만드는 설계 방법

클래스가 불변임을 보장하려면 자신을 상속하지 못하게 해야하는데 가장 쉬운 방법은 final으로 선언해주는 것입니다. 하지만 이것보다 더 유연한 방법이 있습니다. 모든 생성자를 private 혹은 package-private으로 만들고 public 정적 펙터리를 제공하는 방법입니다.
``` java
public final class Calculator {
  private final int x;
  private final int y;

  private Calculator(int x, int y) {
    this.x = x;
    this.y = y;
  }

  public static Calculator valueOf(int x, int y) {
    return new Calculator(x, y);
  }
  ...

```
정적 팩토리 방식으로 다수의 구현 클래스를 활용해 유연성을 제공하고, 객체 캐싱 기능을 추가해 성능을 끌어올리 수 있습니다.</br
>
 정리
-   **클래스는 꼭 필요한 경우가 아니면 불변이어야 합니다**. 
-   불변 클래스의 장점들도 있지만 성능 문제도 함께 고려해야합니다
-   불변으로 만들 수 없는 클래스라도 변경할 수 있는 부분은 최소한으로 줄입시다.
-   다른 합당한 이유가 없다면 모두 private final이어야 합니다.
-   생성자는 불변식 설정 초기화가 완벽히 끝난 상태의 객체를 생성해야 합니다.