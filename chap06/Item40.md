# [Item 40] @Override 애너테이션을 일관되게 사용하여라.

`@Override`애너테이션은 상위 타입 메서드를 재정의 했다는 뜻으로 메서드 선언에 달 수 있습니다.

``` java
// 버그를 찾아보자
 public class Bigram {

  private final char first;
  private final char second;

  public Bigram(char first, char second) {
    this.first = first;
    this.second = second;
  }

  public boolean equals(Bigram b) {
    return b.first == first && b.second == second;
  }

  public int hashCode() {
    return 31 * first + second;
  }

  public static void main(String[] args) {
    Set<Bigram> s = new HashSet<>();
    for (int i = 0; i < 10; i++) {
      for (char ch = 'a'; ch <= 'z'; ch++) {
        s.add(new Bigram(ch, ch));
      }
    }
    System.out.println(s.size());
  }
}
```
`Set`은 중복이 허용되지 않아서 26을 출력할 거 같지만 실제로 260이 출력됐습니다. 어디가 잘못 됐을까요?
</br>
정답은 `equals`를 '재정의' 한 게 아니라 '다중정의'를 해서 그렇습니다. Object의 equals를 재정의 하려면 매개변수 타입을 `Object`로 해야하는데 다른 타입을 지정해서 새로운 equals를 만들게 된 것입니다. 따라서 Hash를 사용할 때 원하는 데로 결과값을 도출할 수 없었던 것입니다.

``` java
  @Override
  public boolean equals(Bigram b) {
    return b.first == first && b.second == second;
  }
```
이런식으로 애너테이션을 달면 컴파일 오류가 발생하므로 쉽게 오류를 수정할 수 있습니다. 그러니 상위 클래스의 메서드를 재정의할 때는 @Override 애너테이션을 달도록 합시다. 예외적으로 구체 클래스에서 상위 클래스의 추상 메서드를 재정의 할 때는 굳이 사용하지 않아도 되긴 합니다. 구체 클래스인데 아직 구현하지 않은 추상 메서드가 남아 있다면 컴파일러가 알려주기 떄문입니다. 그래도 일관성과 가시성을 위해 달아주는 게 좋습니다.


