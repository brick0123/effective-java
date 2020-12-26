# [Item 30] 이왕이면 제네릭 메서드로 만들라

제네릭 메서드는 대표적으로 `Collections`의 알고리즘 메서드(binarySearch, sort등)가 있습니다. 사용 방법은 리턴타입 앞에다 타입을 명시해주면 됩니다. 다음은 두 집합의 합집합을 반환하는 문제가 있는 메서드입니다.
``` java
// raw tyoe 사용 - 수용 불가
  public static Set union(Set s1, Set s2) {
    Set result = new HashSet(s1);
    result.addAll(s2);
    return result;
  }
```
 컴파일은 되지만 경고가 발생합니다. 경고를 없애려면 이 메서드 타입을 안전하게 만들어야 합니다. 다음 코드에서 타입 매개변수 목록은 <E>이고 반환 타입은 Set<E>입니다.
 ``` java
   public static <E> Set<E> union(Set<E> s1, Set<E> s2) {
    Set<E> result = new HashSet<>(s1);
    result.addAll(s2);
    return result;
  }
 ```
 다음 코드는 이 메서드를 사용하는 프로그램입니다. 직접 형변환 하지 않아도 어떤 오류나 경고 없이 컴파일 됩니다.
``` java
public static void main(Sting[] args) {
    Set<String> guys = Set.of("pual", "jin");
    Set<String> stooges = Set.of("jerry", "sia");
    Set<String> aflCio = union(guys, stooges);
    System.out.println(aflCio);
}
```
이를 한정적 와일드카드 타입을 사용하면 더 유연하게 개선할 수 있습니다.
</br>
제네릭은 런타임에 타입 정보가 소거되므로 하나의 객체를 어떤 타입으로든 매개변수화 할 수가 있습니다. 하지만 이렇게 하려면 요청한 타입 매개변수에 맞게 매번 그 객체의 타입을 바꿔주는 정적 팩터리를 만들어야 합니다. 이 패턴을 제네릭 싱글턴 팩터티라 하며, Collections.reverseOrder 같은 함수 객체나 Collections.emptySet 같은 컬렉션용으로 사용합니다.
</br>
이번에는 항등함수를 담은 클래스를 만들고 싶다고 해봅시다. 자바 라이브러리의 Function.idenify를 사용해도 되지만 직접 작성해 보겠습니다. 항등함수 객체는 상태가 없으니 요청할 때마다 새로 생성하는 것은 낭비입니다. 자바의 제네릭이 실체화된다면 항등함수를 타입별로 하나씩 만들어야 했겠지만, 소거 방식을 사용한 덕에 제네릭 싱글턴 하나면 충분합니다.

``` java
  private static UnaryOperator<Object> IDENTIFY_FN = (t) -> t;

  @SuppressWarnings("unchecked")
  public static <T> UnaryOperator<T> identifyFunction() {
    return (UnaryOperator<T>) IDENTIFY_FN; 
  }
```
T가 어떤 타입이든 UnaryOperator<T>를 사용해도 type safe합니다. </br>
상대적으로 드물긴 하지만 자기 자신이 들어간 표현식을 사용하여 타입 매개변수의 허용 범위를 한정할 수 있습니다. 바로 재귀적 타입 한정이라는 개념입니다. 주로 타입의 자연적 순서를 정하는 `Comparable` 인터페이스와 함께 쓰입니다.

``` java
public interface Comparable<T> {
    int compareTo(T o);
}
```
타입 매개변수 T는 `Comparable<T>`를 구현한 타입이 비교할 수 있는 원소의 타입을 정의합니다. 실제로 거의 모든 타입은 자신과 같은 타입의 원소와만 비교할 수 있습니다. 따라서 String은 Comparable<String>을 구현하고 Integer는 Comparable<Integer>를 구현한 식입니다.

``` java
public static <E extends Comparable<E>> E max(Collection<E> c);
```
타입 한정인 <E extends Comparable<E>>는 "모든 타입 E는 자신과 비교할 수 있다"라고 해석할 수 있습니다.
</br>
다음은 방금 선언한 메서드의 구현입니다. 컬렉션에 담긴 원소의 자연적 순서를 기준으로 최댓값을 계산하며, 컴파일 오류나 경고는 발생하지 않습니다.

``` java
  public static <E extends Comparable<E>> E max(Collection<E> c) {
    if (c.isEmpty()) {
      throw new IllegalArgumentException("컬렉션이 비어 있습니다.");
    }

    E result = null;
    for (E e : c) {
      if (result == null || e.compareTo(result) > 0) {
        result = Objects.requireNonNull(e);
      }
    }
    return result;
  }
```
### 정리
- 제네릭 타입과 마찬가지로, 클라이언트에서 입력 매개변수와 반환값을 명시적으로 형변환해야 하는 메서드보다 제네릭 메서드가 더 안전하며 사용하기도 쉽습니다.