# [Item 23] 태그 달린 클래스보다는 클래스 계층구조를 활용하자.

때때로 두 가지 이상의 의미를 표현하고 인스턴스의 특징을 알려주는 태그 필드로 나타내는 클래스를 본 적이 있을겁니다.

### 안 좋은 예시
``` java
public class Figure {

  enum Shape { RECTANGLE, CIRCLE };

  // 태그 필드 - 현재 모양을 나타냅니다.
  final Shape shape;

  // 모양이 사각형(RECTANGLE)일 때만 쓰입니다.
  double length;
  double width;

  // 다음 필드 모양이 원(CIRCLE)일 때만 쓰입니다.
  double radius;

  // 원 생성
  Figure(double radius) {
    shape = Shape.CIRCLE;
    this.radius = radius;
  }

  Figure(double length, double width) {
    shape = Shape.RECTANGLE;
    this.length = length;
    this.width = width;
  }

  double area() {
    switch (shape) {
      case RECTANGLE:
        return length * width;
      case CIRCLE:
        return Math.PI * (radius * radius);
      default:
        throw new AssertionError();
    }
  }
}
```

태그 달린 클래스에는 단점이 많습니다. 열거 타입 선언, 태그 필드, switch문 등 잡다한 코드가 너무 많습니다. 여러 구현이 한 클래스에 있어서 구현도 나쁩니다. 인스턴스가 다른 특징에 속하는 관련없는 필드로 인해 메모리도 많이 사용하게 됩니다. 또 다른 의미를 추가하려면 switch문을 찾아 새 의미를 처리하는 코드를 추가해야하는 데 하나라도 빠뜨리면 문제가 될 수도 있습니다. 마지막으로, 인스턴스의 타입만으로는 현재 나타내는 의미를 알 길이 없습니다. 한 마디로 **태그 달린 클래스는 장황하고, 오류를 내기 쉽고, 비효율적입니다.**
</br>
클래스 계층구조를 활용하는 **서브타이핑**을 이용해서 타입 하나로 다양한 의미의 객체를 표현할 수 있습니다. 태그 달린 클래스는 클래스 계층구조를 어설프게 흉내낸 아류일뿐입니다.
</br>
### 태그달린 클래스를 클래스 계층 구조로 바꾸는 방법
1. 계층구조의 루트가 될 추상 클래스를 정의하고, 태그 값에 따라 동작이 달라지는 메서드들을 루트 클래스의 추상 메서드로 선언합니다.
2. 태그 값에 상관없이 동작이 일정한 메서드들을 루트 클래승 일반 메서드로 추가합니다.
3. 모든 하위 클래스에서 공통으로 사용하는 데이터 필드들도 전부 루트 클래스로 올립니다.
4. 루트 클래스를 확장한 구체 클래스를 의미별로 하나씩 정의합니다.

Figure 클래스에서는 태그 값과 상관없는 메서드가 하나도 없고, 모든 하위 클래스에서 사용하는 공통 데이터 필드가 없습니다. 그 결과 루트 클래스에는 추상메서드 area하나만 남게 됩니다.</br>
다음은 Figure 클래스를 계층구조 방식으로 구현한 코드입니다.
``` java
abstract class Figure {
  abstract double area;
}

public class Circle extends Figure{
  final double radius;

  public Circle(double radius) {
    this.radius = radius;
  }

  @Override
  double area() {
    return Math.PI * (radius * radius);
  }

}

// Rentangle 클래스 위와 같은 방식으로 구현하면 됩니다.
```
태그 달린 클래스에 비교하면 쓸데 없는 코드들은 모두 제거했고, 각 의미를 독립된 클래스에 담아 관련 없던 데이터 필드를 모두 제거했습니다. 살아 남은필드들은 모두 final이며 각 클래스의 생성자가 모든 필드를 남김없이 초기화하고 추상 메서드를 구현했는지 컴파일러가 확인해줍니다.
### 정리
- 태그 달린 클래스를 써야 하는 상황은 거의 없습니다.
- 새로운 클래스를 작성할 때 태그 필드가 등자했다면 계층 구조롤 대체하는 방법을 고려해보자.
- 기존 클래스가 필드를 사용하고 있다면 걔층구조로 리팩터링 하는 걸 고려민해보자.