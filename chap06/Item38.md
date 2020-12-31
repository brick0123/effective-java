# [Item 38] 확장할 수 있는 열거 타입이 필요하면 인터페이스를 사용하라.

열거 타입은 확장할 수 없게 설계 되었습니다. 확장한 타입의 원소는 기반 타입의 원소로 취급하지만 그 반대는 성립하지 않는다면 어폐가 있고 기반 타입과 확장된 타입들의 원소를 모두를 순회할 방법도 마땅치 않으며 확장성을 높이려면 고려할 요소가 늘어나 설계와 구현이 더 복잡해지기 때문입니다.
</br>
그러나 연산 코드는 확장할 수 있는 열거 타입과 어울립니다. API가 제공하는 기본 연산 외에 사용자가 확장 연산을 추가할 수 있도록 열어줘야 할 때가 있습니다. 

``` java
package com.example.jpatest.t;

public interface Operation {

  double apply(double x, double y);
}

public enum BasicOperation implements Operation {
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

  BasicOperation(String symbol) {
    this.symbol = symbol;
  }

  @Override
  public String toString() {
    return symbol;
  }
}
```
`Operation`인터페이스를 구현하여 구현체가 다른 열거 타입을 이용할 수 있습니다.

``` java
// 구현체를 메서드에 명시해서 사용하는 방식
public static void main(String[] args) {
    double x = Double.parseDouble(args[0]); 
    double y = Double.parseDouble(args[1]); 
    test(ExtendedOperation.class, x, y);
}
private static <T extends Enum<T> & Operation> void test( Class<T> opEnumType, double x, double y) {
    for (Operation op : opEnumType.getEnumConstants())
        System.out.printf("%f %s %f = %f%n", x, op, y, op.apply(x, y));
}
```

여기서 class 리터럴은 한정적 타입토큰 역할을 합니다.

``` java
// 두 번째 방법
public static void main(String[] args) {
    double x = Double.parseDouble(args[0]);
    double y = Double.parseDouble(args[1]); 
    test(Arrays.asList(ExtendedOperation.values()), x, y);
}
private static void test(Collection<? extends Operation> opSet, double x, double y) {
    for (Operation op : opSet)
        System.out.printf("%f %s %f = %f%n",x, op, y, op.apply(x, y));
}
```
여러 구현 타입의 연산을 조합해 호출할 수 있게 되므로 더 유연해졌습니다 하지만 특정 연산에서는 EnumMap과 EnumSet을 사용하지 못합니다.</br>
인터페이스를 활용하는 방법에는 사소한 문제가 있습니다. 열거 타입끼리 구현을 상속할 수 없다는 점입니다. 아무 상태에도 의존하지 않는 경우에는 디폴트 구현을 이용해 인터페이스에 추가하는 방방이 있습니다. 하지만 모든 구현체에 공유해야 하므로 공유하는 기능이 많다면 그 부분을 별도의 도우미 클래스나 정적 도우미 메서드로 분리하는 방식으로 코드 중복을 없앨 수 있습니다.

한 줄 요약: 열거 타입 자체는 확장할 수 없지만 인터페이스와 인터페이스를 구현하는 열거 타입을 함께 사용하면 같은 효과를 낼 수 있습니다.
