# [Item 53] 가변인수는 신중히 사용하라.

가변인수(varargs) 메서드는 명시한 타입의 인수를 0개 이상 받을 수 있습니다. 다음 예제는 int 인자의 합을 구하는 가변인수 메서드입니다.

``` java
static int sum(int... args) {
    int sum = 0;
    for (int args: args) {
        sum += args;
    }
    return sum;
}
```

1개 이상의 인수가 필요한 경우도 있습니다. 다음 예제는 1개 이상의 인수를 필요료하는 가변인수 메서드의 잘못된 구현입니다.

``` java
  static int sum(int... args) {
    if (args.length == 0) {
      throw new IllegalArgumentException("인수가 1개 이상 필요합니다.");
    }
    int min = args[0];
    for (int arg : args)
      if (arg < min)
        min = arg;
    return min;
  }
```
이 방식에는 몇 가지 문제점이 있습니다. 가장 큰 문제는 인수를 0개만 넣으면 **런타임** 시점에 실패한다는 것입니다. 코드도 지저분합니다.</br>
다음 코드는 매개변수를 2개를 받아서 이 문제를 해결하는 방식입니다.

``` java
static int min(int firstArg, int... remainingArgs) {
    int min = firstArg;
    for (int arg : remainingArgs) {
        if (arg < min)
            min = arg
    }
    return min;
}
```
가변인수를 사용하면서 주의할 점이 있습니다. 가변인수 메서드는 호출될 때마다 배열을 새로 하나 할당하고 초기화기 때문에 성능에 민감할 경우 조심히 주의해서 사용해야합니다. 기변인수 메서드의 유용성을 활용하고 싶고 성능에 민감할 경우 다른 방식을 이용할 수 있습니다. 예를 들어 해당 메서드 호출의 95%가 인수 3개 이하로 사용한다고 하면 다음처럼 0개인 것부터 4개인 것까지, 총 5개를 정의해서 사용할 수 있습니다.

``` java
   public void foo() { }
   public void foo(int a1) { }
   public void foo(int a1, int a2) { }
   public void foo(int a1, int a2, int a3) { }
   public void foo(int a1, int a2, int a3, int... rest) { }
```

`EnumSet`의 정적 팩터리도 이 기법을 사용해 열거 타입 집합 생성 비용을 최소화 합니다.