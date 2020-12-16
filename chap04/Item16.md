# [Item 16] public 클래스에서는 public 필드가 아닌 접근 메서드를 사용하라.

``` java
// 부적적한 코드
public class Point {
  public int x; 
  public int y;
}

```
위 코드는 객체지향의 특징 중 하나인 캡슐화를 살리지 못했습니다.</br>
다음과 같이 추상화의 이점을 살려서 코드를 수정할 수 있습니다.
``` java
public class Point {
  public int x;
  public int y;

  public Point(int x, int y) {
    this.x = x;
    this.y = y;
  }

  public int getX() { return x; }
  public int getY() { return y; }

  public void setX(int x) { this.x = x; }
  public void setY(int y) { this.y = y; }
}

```
외부에서 접근할 수 있는 클래스라면 접근자를 제공함으로써 클래스 내부 표현 방식을 변경할 수 있는 유연성을 얻을 수 있습니다. </br>
public 클래스의 불변이라면 직접 노출할 때의 단점이 줄어들지만 여전히 문제점은 있습니다. API를 변경하지 않고는 표현 방식을 바꿀 수 없고, 필드를 읽을 때 부수 작업을 수행할 수 없다는 단점이 있습니다. 하지만 불변식은 보장할 수 있습니다. 
> 정리</br>
> public 클래스의 가변 필드는 절대 노출해서는 안 됩니다. 불변 필드라면 위험은 덜 하지만 완전히 안심할 수는 없습니다. 하지만 package-private 클래스나 private 중첩 클래스에서는 필드를 노출하는 편이 나을 때도 있습니다.