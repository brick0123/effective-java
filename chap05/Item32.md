# [Item 32] 제네릭과 가변인수를 함께 쓸 때는 신중하라.

가변인수와 제네릭은 자바 5에 함께 추가되었는데 이 둘은 서로 어울리지 않습니다.</br>
가변인수(varargs)란 메서드에 넘기는 인수의 개수를 클라이언트가 조절할 수 있게 해주는 것입니다. 구현 방식에 허점이 있습나다. 가변인수 메서드를 호출하면 가변인수를 담기 위한 배열이 자동으로 하나 만들어집니다. 그런데 내부로 감춰야 했을 이 배열을 그만  클라이언트에 노출하는 문제가 생겼습니다. 그 결과 verargs 매개변수에 제네릭이나 매개변수화 타입이 포함되면 알기 어려운 컴파일 경고가 발생합니다.
</br>
실체화 불가 타입은 런타임에 컴파일보다 타입 관련 정보를 적게 담고 있습니다. 그리고 거의 모든 제네릭과 매개변수화 타입은 실체화되지 않습니다. 메서드 선언할 때 실체화 불가 타입으로 varargs 매겨변수를 선언하면 컴파일러가 경고를 보냅니다. 가변인수 메서드를 호출할 때도 varags 매개변수가 실체화 불가 타입으로 추론되면, 그 호출에 대해서도 경고를 냅니다.
</br>
매개변수화 타입의 변수가 타입이 다른 다른 객체를 참조하면 힙 오염이 발생합니다. 이렇게 다른 타입 객체를 참조하는 상황에서는 컴파일러가 자동 생성한 형변환이 실패할 수 있으니, 제네릭 타입 시스템이 약속한 타입 안전성의 근간이 흔들려버립니다. 다음 메서드를 예로 생각해봅시다.

``` java
  static void dangerous(List<String>... stringList) {
    List<Integer> integerList = Collections.singletonList(42);
    Object[] objects = stringList;
    objects[0] = integerList;  // 힙 오염
    String s = stringList[0].get(0); // ClassCaseException
  }
```
가변인수와 제네릭을 사용하는 메서드는 대표적으로 Arrys.asList, Collections.addAll등이 있습니다. 자바 7부터는 @SafeVarargs 에너테이션을 사용해서 그 메서드가 타입 안전성을 보장한다는 걸 알려줄 수 있습니다.</br>
메서드가 이 배열에 아무것도 저장하지 않는다면 괜찮지만 아무것도 저장하지 않고도 타입 안전성을 깨뜨릴 수 있습니다.

``` java
static <T> T[] toArrays(T... args) {
    return args;
}
```
이 메서드가 반환하는 타입은 이 메서드에 인수를 넘기는 컴파일타임에 결정되는데, 그 시점에는 컴파일러에게 충분한 정보가 주이지지 않아 타입을 잘못 판단할 수 있습니다. 따라서 varargs 매개변수 배열을 그대로 반환하면 힙 오염을 이 메서드를 호출한 쪽의 콕스 택으로 까지 전이할 수도 있습니다.

```java
  static <T> T[] pickTwo(T a, T b, T c) {
    switch (ThreadLocalRandom.current().nextInt(3)) {
      case 0: return toArrays(a, b);
      case 1: return toArrays(a, c);
      case 2: return toArrays(c, b);
    }
    throw new AssertionError();
  }
```
toArray 메서드가 돌려준 이 배열이 그대로 pickTwo를 호출한 클라이언트까지 전달되는데 항상 Object[] 타입 배열을 반환하게 됩니다.
``` java
String[] attributes = pickTwo("좋은", "빠른", "저렴한");
```
런타임시 `ClassCastException`을 던집니다. pickTwo에서 Object[]을 반환하는데 String[]으로 변환하려고 해서 예외가 발생합니다.</br>
배열 내용의 일부 함수를 호출하는(varargs를 받지 않는)일반 메서드에 넘기는 것은 안전합니다.

``` java
  @SafeVarargs
  static <T> List<T> flatten(List<? extends T>... lists) {
    List<T> result = new ArrayList<>();
    for (List<? extends T> list : lists) {
      result.addAll(list);
    }
    return result;
  }
```
임의 개수의 리스트를 받아서, 순서대로 그 안의 모든 원소를 하나의 리스트로 옮겨 반환하는 메서드입니다. @SafeVarargs 애너테이션은 안전하지 않은 varargs 메서드에는 절대 작성해서는 안 됩니다. 힙 오염 경고가 뜨면 무조건 검증을 해야합니다. 그리고 재정의할 수 없는 메서드에만 달아야 합니다. 재정의한 메서드도 안전할지는 보장할 수 없기 때문입니다.

``` java
  static <T> List<T> flatten(List<List<? extends T>> lists) {
    List<T> result = new ArrayList<>();
    for (List<? extends T> list : lists) {
      result.addAll(list);
    }
    return result;
  }
```
위와 같이  @SafeVarargs 애너테이션을 사용하지 않고 verargs 매개변수를 List 매개변수로 바꿀 수도 있습니다.
### 정리
- 가변인수와 제네릭은 궁합이 좋지 않습니다.
- 가변인수 기능은 배열을 노출하여 추상화가 완벽하지 못하고, 배열과 제네릭의 타입 규칙이서 서로 다르기 때문입니다.
- 제네릭 verargs 매개변수는 type safe하지 않지만, 허용됩니다.
- verargs 매개변수를 사용하려면 타입이 안전한지 확인하고 @SafeVarargs 애너테이션을 이용합시다.
  