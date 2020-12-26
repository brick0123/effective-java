# [Item 29] 이왕이면 제네릭 타입으로 만들라.

Item 7에서 다루었던 스택 코드를 제네릭으로 변형한 코드입니다.
```java
public class Stack<E> {
  private E[] elements;
  private int size = 0;
  private static final int DEFAULT_INITIAL_CAPACITY = 16;

  public Stack() {
    elements = (E[]) new Object[DEFAULT_INITIAL_CAPACITY]; // 경고 메세지 타입이 안전하지 않음
  }

  public void push(E e) {
    ensoureCapaciy();
    elements[size++] = e;
  }

  public E pop() {
    if (size == 0) {
      throw new EmptyStackException();
    }
    E result = elements[--size];
    elements[size] = null;
    return result;
  }
  ... 중략
}
```
컴파일러는 이 프로그램이 안전한지 증명할 방법은 없지만, 우리는 할 수 있는 한 이 비검사 형변환이 프로그램의  타입 안전성을 해치지 않는지 스스로 확인해야합니다.
</br>
비검사 형변환이 안전하다는 걸 확인했다면 범위를 최소로 좁혀 @SuppressWarnings("unchecked")를 이용하여 해당 경고를 숨깁시다.
``` java
// 배열 elements는 push(E)로 넘어온 E 인스턴스만 담는다.
// 따라서 타입 안전성을 보장하지만
// 이 배열의 런타임 타입은 E[]가 아닌 Object[]입니다.
@SuppressWarnings("unchecked")
public Stack() {
  elements = (E[]) new Object[DEFAULT_INITIAL_CAPACITY];
}
```
위 방법 말고 다른 방식으로는 elements 필드의 타입을 E[]에서 Object[]로 바꾸는 것입니다. 이렇게 하면 pop메서드 부분을 다음과 같이 수정해줘야 합니다.
``` java
E result = (E) elements[--size];
```
E는 실체화 불가 타입이므로 컴파일러는 런타임에 이뤄지는 형변환이 안전한지 증명할 방법이 없습니다. 이번에도 직접 증명하고 경고를 숨길 수 있습니다.
``` java
@SuppressWarnings("unchecked")
E result = (E) elements[--size];
```
첫 번째 방식은 형변환을 배열 생성시 단 한 번만 해주면 되지만, 두 번째 방식에서는 배열에서 원소를 읽을 때마다 해줘야하므로 첫 번째 방식이 더 자주 사용됩니다. 하지만
(E가 Object가 아닌 한) 배열의 런타임 타입이 컴파일타임 타입과 달라 힙 오염을 일으킵니다.</br>
사실 제네릭 타입 안에서 리스트를 사용하는 게 항상 가능하더라도, 꼭 더 좋은 건 아닙니다. 자바가 리스트를 기본 타입으로 제공하지 않으므로 ArrayList같은 제네릭 타입도 결국은 기본 타입인 배열을 사용해 구현해야 합니다. 또한 HashMap 같은 제네릭 타입은 성능을 높일 목적으로 배열을 사용하기도 합니다.
### 정리
- 클라이언트에서 직접 형변환해야 하는 타입보다 제네릭 타입이 더 안전하고 쓰기 편합니다.
- 새로운 타입을 설계할 때는 형변환 없이도 사용할 수 있도록 합시다.
- 기존 타입 중 제네릭이었어야 하는 게 있다면 제네릭 타입으로 변경합시다.


