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
정답은 `equals`를 '재정의' 한 게 아니라 '다중정의'를 해서 그렇습니다. Object의 equals를 재정의 하려면 매개변수 타입을 `Object`로 해야하는데 다른 타입을 지정해서 새로운 equals를 만들게 된 것입니다.

``` java
  @Override
  public boolean equals(Bigram b) {
    return b.first == first && b.second == second;
  }

```