# [Item 3] private 생성자나 열거 타입으로  싱글턴임을 보증하라

싱글턴(singletone)이란 인스턴스를 오직 하나만 생성할 수 있는 클래스를 말합니다. 즉 객체를 호출할 때마다 new해서 새로 생성하지 않고 하나의 인스턴스를 계속 사용하는 것입니다.</br>
싱글턴을 만드는 방식은 보통 둘 중 하나입니다. 

### **public static 멤버가 final인 방식**
``` java
public class Elvis {

  public static final Elvis INSTANCE = new Elvis();

  private void Elvis() { ... }

  private void leaveTheBuilding() { ... }

}
```
public이나 protected 생성자가 없으므로 Elvis 클래스가 초기화 될 때 만들어진 인스턴스는 하나 뿐입니다. 단 권한이 있는 클라이언트는 리플렉션 API인 AccessibleObject.setAccessible을 사용해 private 생성자를 호출할 수 있습니다.</br>public 필드 방식의 첫번째 장점은 해당 클래스가 싱글턴임이 API에 명백히 드러나는 것입니다.</br>
두 번째 장점은 간결하다는 것입니다.
이 점은 생성자를 수정하여 두 번째 객체가 생성되려고 하면 예외를 단지면 됩니다. 

</br>
싱글턴을 만드는 두 번째 방법에는 정적 팩토리 메서드를 public static 멤버로 제공합니다.

### **정적 팩토리 방식의 싱글톤**
``` java
public class Elvis {

  private static final Elvis INSTANCE = new Elvis();

  private void Elvis() { ... }
  public static Elvis getInstance() { return INSTANCE; }


  private void leaveTheBuilding() { ... }

}
```

이 역시 리플렉션을 통한 예외는 똑같이 적용됩니다.</br>
장점
- API를 바꾸지 않고 싱글턴이 아니게 변경할 수 있습니다.
- 유일한 인스턴스를 반환하던 팩터리 메서드가 호출하는 스레드별로 다른 인스턴스를 넘겨주게 할 수 있습니다.
- 정적 팩토리 메서드 참조를 공급자(supplier)로 사용할 수 있습니다.
</br>

위에서 살펴본 두 방법 모두 직렬화하려면 모든 인스턴스 필드를 일시적(transient)이라고 선언하고 `readResolve` 메서드를 다음과 같이 제공해야 합니다. 
``` java
private Object readResolve() {
    returm INSTANCE;
}
```
이렇게 하지 않으면 역직렬화 할때마다 새로운 인스턴스가 생성됩니다.</br></br>

### **열거 타입의 방식의 싱글턴**
``` java
public enum Elvis {
    INSTANCEl

    public void leaveTheBuilding() { ... }
}
```
직렬화 상황 그리고 리플렉션 공격에서도 싱글턴임을 보장할 수 있습니다. **대부분 상황에서는 원소가 하나뿐인 열거 타입이 싱글턴을 만드는 가장 좋은 방법입니다.**</br>
하지만 이 방법은 Enum 외의 클래스를 상속해야한다면 사용할 수 없습니다.(인터페이스를 구현할 수는 있습니다)
