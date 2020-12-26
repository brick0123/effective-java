# [Item 31] 한정적 와일드카드를 사용해 API 유연성을 높여라

때론 불공변 방식보다 유연한 무언가가 필요할 때가 있습니다. 아이템 29의 Stack 클래스를 떠올려보면

``` java
public class Stack<T> {
    public Stack();
    public void push (E e);
    public E pop();
    public boolean isEmpty();
}
```
여기서 일련의 원소를 스택에 넣는 메서드를 추가한다고 하면
``` java
public void pushAll(Iterable<E> src) {
    for (E e : src) {
        push(e);
    }
}
```
Iterable src의 원소 타입의 스택의 원소 타입과 일치하면 잘 작동합니다. 하지만 Stack<Number>로 선언한 후 pushAll(intVal)을(Iteger 타입) 호출하면 오류가 뜹니다. 매개변수 타입이 불공변이기 떄문입니다.</br>
이러한 상황에서는 한정된 와일드카드(unbounded wildcard)를 이용해서 해결할 수 있습니다. pushAll의 입력 매개변수 타입은 'E의 iterable'이 아니라 'E의 하위타입 Iterable'이어야 하며, 와일드 카드 Iterable<? extends E>가 정확히 이런뜻입니다.
``` java
public void pushAll(Iterable<? extends E> src) {
    for (E e : src) {
        push(e);
    }
}
```
위와 같이 수정할 수 있습니다. 이번에는 popAll 와일드카드 타입을 사용하지 않은 메서드를 작성해 보겠습니다.

``` java
// 와을드카드 타입을 사용하지 않은 메서드 - 결함이 있습니다.
public void popAll(Collection<E> dst) {
    while (!isEmpty()) {
        dst.add(pop());
    }
}
```

``` java
Stack<Number> numberStack = new Stack<>();
Collection<Object> objects = ...;
numberStack.popAll(objects);
```
컴파일 하면 "Collection<Object>는 Collection<Nummber>의 하위 타입이 아니다"라는 오류가 발생합니다. 이번에는 반대로 'E의 Collection'이 아니라 'E의 상위 타입의 Collection'이어야 합니다.

``` java
// 와을드카드 타입을 사용하지 않은 메서드 - 결함이 있습니다.
public void popAll(Collection<E super E> dst) {
    while (!isEmpty()) {
        dst.add(pop());
    }
}
```
위와 같이 수정할 수 있습니다. 메세지는 분명합니다. 유연성을 극대화하려면 원소의 생산자나 소비자용 매개변수에 와일드 카드를 사용합시다. 한편,입력 매개변수가 생상자와 소비자 역할을 동시에 한다면 와일드카드 타입을 써도 좋을 게 없습니다. 타입을 정확히 지정해야 하는 상황으로, 이때는 와일드카드 타입을 쓰지말아야합니다.</br>
다음 공식을 외워두면 어떤 와일드 카드 타입을 써야 하는지 도움이 될 것입니다.
> PECS: producer-extends, consumer-super

즉 매개변수화 타입 T가 생성자라면 <? extends T>를 사용하고, 소비자라면 <?super T>를 사용합시다. **클래스 사용자가 와일드카드 타입을 신경 써야 한다면 그 API에 무슨 문제가 있을 가능성이 큽니다.**

``` java
// Item 30에서 사용했던 코드
public static <E extends Comparable<E>> E max(List<E> c);
```
이 코드를 와일드카드 타입을 사용해 다듬은 모습입니다.
``` java
// Item 30에서 사용했던 코드
public static <E extends Comparable<? super E>> E max(List<? extends E> c);
```

위 코드는 PECS 공식을 두 번 적용했습니다. 입력 매개변수에서는 E 인스턴스를 생산하므로 원래의 List<E>를 List<? extends E>로 수정했습니다. 원래 선언에서는 E가 Comparale<E>를 확장한다고 정의헸는데, 이때 Comparable<E>
는 E 인스턴스를 소비합니다.(그리고 선후 관계를 뜻하는 정수를 생산합니다) 그래서 매개변수화 타입 Comparable<E>는 E 한정적 와일드카드 타입 Comparable<? super E>로 대체 했습니다. Comparable은 언제나 소비자이므로, 일반적으로 Comparable, 일반적으로 Comparable<E>보다는 Comparable<? super E>를 사용하는 편이 낫습니다.</br>