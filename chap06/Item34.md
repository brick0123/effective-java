# [Item 34] int 상수 대신 열거 타입을 사용하라.

열거 타입이란 일정 개수의 상수 값을 정의하고 그 이외 값들은 허용하지 않는 타입입니다. 대표적으로 사계절, 요일, 태양계의 행성 등이 있습니다.

### 열거 타입을 지원하기 전 코드

``` java
// 정수 열거 패턴 - 상당히 취약하다.
public static final int EAST = 0;
public static final int WEST = 1;
public static final int SOUTH = 2;
public static final int NORTH = 3;
```
이 코드는 타입의 안전성을 보장할 수 없고 표현력도 좋지 않습니다. </br>
정수 열거 패턴을 사용한 프로그램은 깨지기가 쉽습니다. 단지 상수를 나열한 것 뿐이라 컴파일하면 그 값이 클라이언트 파일에 그대로 새겨집니다. 따라서 상수의 값이 바뀌면 클라이언트도 수정해줘야 합니다. 
</br>
정수 상수는 그 값을 출력하거나 디버거로 살펴보면 특별한 의미를 나타내는 것이 아니라 단지 숫자로만 보여서 썩 도움이 되지 않습니다. 정수 대신 문자열 상수를 사용하는 방법도 있지만 이 변형 역시 단점이 많습니다. 상수의 의미를 내포할 수 있지만, 문자열 값 그대로 하드코딩 해야하기 때문에 힘듭니다. 이렇게 힘들게 하드코딩한 문자열이 오타가 있어도 컴파일러는 확인할 수 없습니다.</br>

### 열거 타입
``` java
// 단순한 열거 타입
public enum DIRECTION { EAST, WEST, SOUTH, NORTH }
```

열거 타입 자체는 클래스이며, 상수 하나당 자신의 인스턴스를 하나씩 만들어 **public static final** 필드로 공개합니다. 열거 타입은 컴파일 타임 타입 안전성을 제공합니다. `DIRECTION` 열거 타입을 매개변수로 받는 메서드를 선언 했다면, 건네받은 참조는 null 혹은 DIRECTION의 네 가지 값 중 하나 입니다. 다른 타입의 값을 넘기려 하면 컴파일 오류가 납니다.
``` java
DIRECTION south = "Sounth" // 컴파일 오류 !!!
```
열거타입에 상수를 추가하거나 순서를 변경해도 다시 컴파일 하지 않아도 됩니다. 공개되는 것이 오직 필드의 이름 뿐이라, 정수 열거 패턴과 달리 상수 값이 클라이언트로 컴파일되어 각인되지 않기 때문입니다. 이처럼 열거 타입은 정수 열거 타입의 단점들을 해소해줍니다. 
</br>
열거 타입에는 메서드나 필드를 추가할 수도 있고 임의의 인터페이스를 구현하게 할 수도 있습니다. `Object`, `Comparable`, `Serializable`을 구현 했으며, 그 직렬화 형태도 웬만큼 변형을 가해도 문제없이 동작하게끔 구현해놨습니다.

### 메서드와 필드를 추가한 enum 예제
``` java
public enum Planet {
  MERCURY(3.302e+23, 2.439e6),
  VENUS(4.869e+24, 6.052e6),
  EARTH(5.975e+24, 6.378e6),
  MARS(6.419e+23, 3.393e6),
  JUPITER(1.899e+27, 7.149e7),
  SATURN(5.685e+26, 6.027e7),
  URANUS(8.683e+25, 2.556e7),
  NEPTUNE(1.024e+26, 2.477e7);
  
  private final double mass; // 질량(단위: 킬로그램)
  private final double radius; // 반지름(단위: 미터)
  private final double surfaceGravity; // 표면 중력(단위: m / s^2)
  
  // 중력 상수(단위: m^3 / kg s^2)
  private static final double G = 6.67300E-11;

  // Constructor
  Planet(double mass, double radius) {
    this.mass = mass;
    this.radius = radius;
    surfaceGravity = G * mass / (radius * radius);
  }

  public double mass() {
    return mass;
  }

  public double radius() {
    return radius;
  }

  public double surfaceGravity() {
    return surfaceGravity;
  }

  public double surfaceWeight(double mass) {
    return mass * surfaceGravity;  // F = ma
  }
}
```
열거 타입을 선언한 클래스 혹은 그 패키지에서만 유용한 기능은 private이나 package-private 메서드로 구현하는 게 좋습니다. 일반 클래스와 마찬가지로, 그 기능을 클라이언트에 노출해야 할 합당한 이유가 없다면 private 혹은 package-private으로 선언하라(Item 15)랑 같습니다.
</br>

### 추천하지 않는 코드

``` java
public enum Operation {

  PLUS, MINUS, TIMES, DIVIDE;

  // 상수가 뜻하는 연산을 수행한다.
  public double apply(double x, double y) {
    switch (this) {
      case PLUS: return x + y;
      case MINUS: return x - y;
      case TIMES: return x * y;
      case DIVIDE: return x / y;
    }
    throw new AssertionError("알 수 없는 연산: " + this);
  }
}
```
동작은 하지만 맹점이 있습니다. 예컨대 새로운 상수를 추가하면 해당 case 문도 추가해야 합니다. 열거 타입에 apply를 이용하여 코드를 보완할 수 있습니다. 
### 상수별 메서드 구현을 활용한 열거 타입
``` java
// 상수별 메서드 구현
public enum Operation {

  PLUS {
    public double apply(double x, double y) {
      return x + y;
    }
  }, MINUS {
    public double apply(double x, double y) {
      return x - y;
    }
  }, TIMES {
    public double apply(double x, double y) {
      return x * y;
    }
  }, DIVIDE {
    public double apply(double x, double y) {
      return x / y;
    }
  };

  public abstract double apply(double x, double y);
}
```
보시다시피 apply 메서드가 상수 선언 바로 옆에 붙어 있으니 새로운 상수를 추가할 때 apply도 재정의해야 한다는 사실을 깜빡하기는 어려울 것입니다. 그 뿐만 아니라 apply 메서드가 추상 메서드이므로 재정의 하지 않았다면 컴파일 오류로 알려줍니다. 상수별 메서드 구현을 상수별 데이터와 결합할 수도 있습니다. 
예컨대 다음은 Operation의 toString을 재정의해 해당 연산을 뜻하는 기호를 반환하도록 하는 예제 코드입니다.

``` java
public enum Operation {
  PLUS("+") {
    public double apply(double x, double y) {
      return x + y;
    }
  },
  MINUS("-") {
    public double apply(double x, double y) {
      return x - y;
    }
  },
  TIMES("*") {
    public double apply(double x, double y) {
      return x * y;
    }
  },
  DIVIDE("/") {
    public double apply(double x, double y) {
      return x / y;
    }
  };
  private final String symbol;

  Operation(String symbol) {
    this.symbol = symbol;
  }

  @Override
  public String toString() {
    return symbol;
  }

  public abstract double apply(double x, double y);
}
```

열거 타입에는 상수 이름을 입력받아 그 이름에 해당하는 상수를 반환하는 valueOf(String) 메서드가 자동 생성됩니다. 
</br>
</br>
상수별 메서드 구현에는 열거 타입 상수끼리 코드를 공유하기 어렵다는 단점이 있습니다. 다음은 급명세서에서 쓸 요일을 표현하는 로직을 포함한 열거 타입 예제입니다.

``` java
// 위험한 코드

enum PayrollDay {
  MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY,
  SATURDAY(PayType.WEEKEND), SUNDAY(PayType.WEEKEND);

  private final PayType payType;

  PayrollDay(PayType payType) {
    this.payType = payType;
  }

  PayrollDay() {
    this(PayType.WEEKDAY);
  }  // Default

  int pay(int minutesWorked, int payRate) {
    return payType.pay(minutesWorked, payRate);
  }

  // The strategy enum type
  private enum PayType {
    WEEKDAY {
      int overtimePay(int minsWorked, int payRate) {
        return minsWorked <= MINS_PER_SHIFT ? 0 :
            (minsWorked - MINS_PER_SHIFT) * payRate / 2;
      }
    },
    WEEKEND {
      int overtimePay(int minsWorked, int payRate) {
        return minsWorked * payRate / 2;
      }
    };

    abstract int overtimePay(int mins, int payRate);

    private static final int MINS_PER_SHIFT = 8 * 60;

    int pay(int minsWorked, int payRate) {
      int basePay = minsWorked * payRate;
      return basePay + overtimePay(minsWorked, payRate);
    }
  }
}
```
하지만 기존 열거 타입에 상수별 동작을 혼합해 넣을 때는 switch 문이 좋은 선택이 될 수도 있습니다.

``` java
// switch문을 이용해 원래 열거 타입에 없는 기능을 수행
public static Operation inverse(Operation op) {
  switch(op) {
    case PLUS:   return Operation.MINUS;
    case MINUS:  return Operation.PLUS;
    case TIMES:  return Operation.DIVIDE;
    case DIVIDE: return Operation.TIMES;
    default: thrownewAssertionError("알 수 없는 연산: "+op); 
  }
}
```
추가하려는 메서드가 의미상 열거 타입에 속하지 않는다면 이 방식을 적용하는 게 좋습니다. 열거 타입은 메모리에 올라가는 공간과 초기화하는 시간이 좀 들긴 하지만 크게 체감될 정도는 아닙니다.