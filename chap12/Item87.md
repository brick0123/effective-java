# [Item 87] 커스텀 직렬화 형태를 고려해보라.

이상적인 직렬화는 물리적인 모습과 독립된 논리적인 모습을 표현해야한다. 하지만 기본 직렬화 형태는 객체가 포함한 모든 데이터와 그 객체에서부터 시작해 접근할 수 있는 모든 객체를 담아낸다.

``` java
public class Name implements Serializable {
    /*
     * 성. null이 아니어야 함.
     * @serial
     */

    private final String lastName;

    /*
     * 이름. null이 아니어야 함.
     * @serial
     */
    private final String firstName;

    /*
     * 이름. null이 아니어야 함.
     * @serial
     */
    private final String middleName;

    .. 
}
```
성명은 논리적으로 이름, 성, 중간이라는 3개의 문자열로 구성되며, 앞 코드의 인스턴스 필드들은 이 논리적 구성요소를 정확히 반영했음.
</br></br>

``` java
// 기본 직렬화 형태에 적합하지 않은 클래스
public final class StringList implements Serializable {
  
  private int size = 0;
  private Entry head = null;
  
  private static class Entry implements Serializable {
    
    String data;
    Entry next;
    Entry previous;
  }
  ..
}
```
### 객체의 물리적 표환과 논리적 표현의 차이가 클 때 기본 직렬화를 사옹하면 발생하는 문제점
1. 공개 API가 현재 내부 표현 방식에 영구히 묶인다. 위 코드에서는 StringList.Entry가 공개 API가 되어버린다. </br>
다음 릴리스에서 내부 표현 방식을 변경하더라도 StringList 클래스는 연결 리스트로 펴현된 입력도 처리할 수 있어야한다. 

2. 너무 많은 공간을 차지할 수 있다.위 코드에서 직렬화에 필요없는 연결리스트의 모든 엔트리와 연결 정보까지 저장했다.</br>이처럼 직렬화 형태가 너무 커져서 디스크에 저장하거나 네트워크에 전송하는 속도가 느려질 수 있다.
3. 시간이 너무 많이 걸릴 수 있다. 직렬화 로직은 객체 그래프의 위상에 관한 정보가 없어서 그래프를 직접 순회해볼 수 밖에 없다.
4. 스택 오버플로를 일으킬 수 있다. 기본 직렬화 과정은 객체 그래프를 재귀 순회하는데 크기가 크지 않는 객체 그래프에서도 오버플로를 일으킬 수 있다.

StringList를 합리적으로 직렬화려면 리스트가 포함한 문자열의 개수를 적은 뒤, 그 뒤로 문자열을 나열하는 수준이면 된다. 논리적인 구성만 담는 것이다.

``` java
public final class StringList implements Serializable {

  private transient int size   = 0;
  private transient Entry head = null;
  
  // 더이상 직렬화 x
  private static class Entry {
    String data;
    Entry next;
    Entry previous;
  }
  // 지정한 문자열을 이 리스트에 추가
  public final void add(String s) { ... }

  /**
   * Serialize this {@code StringList} instance.
   *
   * @serialData 이 리스트의 크기(포함된 문자열의 개수) 를 기록한 후 ({@code int}), 이어서 모든 원소를(각각은 {@code String})순서대로 기
   */
  private void writeObject(ObjectOutputStream s) throws IOException {
    s.defaultWriteObject();
    s.writeInt(size);
    // 모든 원소를 올바른 순서로 기록한
    for (Entry e = head; e != null; e = e.next)
      s.writeObject(e.data);
  }
  private void readObject(ObjectInputStream s) throws IOException, ClassNotFoundException {
    s.defaultReadObject();
    int numElements = s.readInt();
    // 모든 원소를 읽어 이 리스트에 삽입한다
    for (int i = 0; i < numElements; i++)
      add((String) s.readObject());
  }
  ..
}

```

`transient`이어도 defaultWriteObject, defaultReadObject를 호출해야 한다. 그래야 향후 릴리스에서 trasient가 아닌 인스턴스 필드가 추가되더라도 상호 호환이 되기 때문이다. 
</br>
기본 직렬화 사용 여부와 상관없이 객체의 전체 상태를 읽는 메서드에 적용해야 하는 동기화 메커니즘을 직렬화에도 적용해야한다.

``` java
// 기본 직렬화를 사용하는 동기화된 클래스를 위한 writeObject 메서드
private synchronized void writeObject(ObjectOutputStream s) 
    throws IOException {
    s.defaultWriteObject();
}
```

writeObject 메서드 안에서 동기화하고 싶으면 클래스의 다른 부분에서 사용하는 락 순서를 똑같이 해야한다. 그렇지 않으면 교착상태에 빠질 수 있다.

</br>
</br>

**어떤 직렬화 형태를 택하든 직렬화 가능 클래스 모두에 직렬버전 UID를 명시적으로 부여해주자**.</br>
이렇게 해야 잠재적인 호환성 문제가 사라진다. 성능도 조금 빨라지는데 명시하지 않을 경우 런타임에 UID 값을 생성하는 연산을 수행하기 때문이다. 구버전하고 호환성을 끊으려면 값을 변경해주면 된다.