# [Item 18] 상속보다는 컴포지션을 사용하라.
우선 이번 아이템에서 다루는 상속은 클래스가 다른 클래스를 확장하는 것을 말합니다. </br>
상속 같은 경우 상위 클래스가 구현 방식에 따라 하위 클래스 동작에 영향을 미칠 수 있습니다.
</br>
**예제를 위한 코드**
``` java
public class CustomHashSet<E> extends HashSet<E> {
  private int addCount = 0;

  public CustomHashSet() { }

  @Override
  public boolean add(E e) {
    addCount++;
    System.out.println("hello");
    return super.add(e);
  }

  @Override
  public boolean addAll(Collection<? extends E> c) {
    addCount += c.size();
    return super.addAll(c);
  }

  public int getAddCount() {
    return addCount;
  }

}
```
```java
    CustomHashSet<String> set = new CustomHashSet<>();
    set.addAll(Arrays.asList("가","나","다"));
    System.out.println(set.getAddCount());
  }
```
원소를 3개 삽입했지만 addCount출력하면 결과값은 `6`이 나옵니다.

디버깅을 해본 결과 상위 클래스에서 addll메서드에서 add를 호출하는데 여기서 add는 오버라이딩 된 add메서드를 결과적으로 3번 더 호출하게 됐습니다.</br>
추가로 다음 릴리즈에서 상위 클래스에 새로운 메서드를 추가했는데, 하필 하위 클래스에 추가한 메서드와 시그니처가 같고 반환 타입이 다를 경우 컴파일조차 되지 않습니다.</br>
이러한 문제점들은 컴포지션을 통해서 쉽게 피해갈 수 있습니다. 컴포지션은 기존 클래스를 확장하는 대신 새로운 클래스를 만들고 private 필드로 기존 클래스의 인스턴스를 참조해서 이용할 수 있습니다.
<br>


상속은 클래스 B가 클래스 A와 **is-a** 관계일 때만 클래스 A를 상속해야 합니다.
 클래스 B가 A를 상속하기 전에 B가 A인가? 자문해보고 "그렇다"라는 확신이 들지 않으면 **컴포지션(has-a)** 을 이용하는 게 좋습니다.
 </br>

컴포지션을 써야 할 상황에서 상속을 사용하는 건 내부 구현을 불필요하게 노출하는 꼴입니다. 그 결과 API가 내부 구현에 묶이고 그 클래스의 성능도 영원히 제한됩니다. 더 심각한 문제는 클라이언트가 노출된 내부에 **직접 접근**할 수 있다는 점입니다.
</br>
컴포지션 대신 상속을 사용하기로 결정하면 마지막으로 자문해야 될 질문이 있습니다. "확장하려는 클래스의 API에 아무런 결함이 없는가" 결함이 있다면, 이 결함이 여러분 클래스의 API까지 전파돼도 괜찮은가?. 상속은 상위 클래스의 API를 그 결함까지도 그대로 승계합니다.
</br>
### **정리**
- 상속은 강력하지만 캡슐화를 해친다는 단점이 있습니다.
- 상속은 상위 클래스와 하위클래스가 순수한 is-a 관계일 때만 사용해야합니다.
- 상속의 단점을 피라려면 컴포지션을 활용합시다.