# [Item 35] ordinal 메서드 대신 인스턴스 필드를 사용하라

대부분 열거 타입 상수는 자연스럽게 하나의 정숫값에 대응됩니다. 모든 엵 ㅓ타입은 해당 상수가 그 열거 타입에서 몇 번째 위치인지를 반환하는 ordinal 이라는 메서드를 제공합니다.

``` java
// ordinal을 잘못 사용한 예
public enum FRUITS {

  APPLE, BANANA, ORANGE;

  public int numberOfFruits() {
    return ordinal() + 1;
  }
}

```
상수 선언 순서를 바꾸는 순간 numberOfFruits는 오작동하며 이미 사용중인 정수와 값이 같은 상수는 추가할 수 없고 중간에 값을 비울 수도 없습니다.</br>
해결책은 매우 간단합니다. ordinal 메서드를 이용하지 않고 인스턴스 필드에 저장하면 됩니다.
``` java
public enum FRUITS {
  APPLE(1), BANANA(2), ORANGE(3);
  
  private final int numberOfFruits;

  FRUITS(int size) {
    this.numberOfFruits = size; 
  }

  public int getNumberOfFruits() {
    return numberOfFruits;
  }
}
```
Enum의 API 문서에는 ordinal에 대해서 "대부분 프로그래머는 이 메서드를 쓸 일이 없습니다. 이 메서드는 EnumSet과 EnumMap 같이 열거 타입 기반의 범용 자료구조에 쓸 목적으로 설계되었습니다."