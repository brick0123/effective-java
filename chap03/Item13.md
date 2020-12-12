# [Item 13] clone 재정의는 주의해서 진행하라.

실무에서 `Cloneable`을 구현한 클래스는 clone 메서드를 public으로 제공하며, 사용자는 당연히 복제가 제대로 이뤄지리라 기대합니다.
하지만 clone 메서드의 일반 규약은 허술한 부분이 있습니다. 다음은 Object 명세에서 가져온 설명입니다.
</br></br>
이 객체의 복사본을 생성해 반환합니다. '복사'의 정확한 뜻은 그 객체를 구현한 클래스에 따라 다를 수 있지만 일반적은 의도는 다음과 같습니다. 어떤 객체 x에 대해 다음 식은 참입니다.
- x.clone() != x
- x.clone.getClass() = x.getClass()
- x.clone().equals(x)

clone을 사용하는 방법은 굉장히 쉽습니다. Cloneable인터페이스를 구현하고 super.clone을 호출하면 됩니다. 이렇게 얻은 객체는 원본의 모든 필드랑 똑같은 값을 가지게 됩니다.
``` java
@Override
public PhoneNumber clone() {
    try{
        return (PhoneNumber) super.class();
    } catch (CloneNotSupportedException e) {
        throw new AssertionsError();
    }
}
```
Obejct의 clone은 Object를 반환하지만 공변 반환 타입을 이용해서 PhoneNumber로 반환했습니다. 이 방식으로 사용하는 클라이언트는 형변환을 따로 해줄 필요가 없습니다.
</br>
간단했던 앞서의 구현이 클래스가 가변 객체를 참조하는 순간 문제점이 발생합니다.
``` java
public class Stack {
    private Object[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;
    
    public Stack() {
        this.elements = new Object[DEFAULT_INITIAL_CAPACITY];
    }

    ... 생략
}
```
clone 메서드가 단순히 super.clone의 결과를 그대로 반환하면 Stack인스턴스의 size 필드는 올바른 값을 갖겠지만, elements필드는 원본 Stack 인스턴스와 똑같은 배열을 참조하게 되는 상황이 발생합니다.</br>
clone메서드는 사실상 생성자와 같은 효과를 냅니다. 즉 clone은 원본 객체에 아무런 변화가 없는 동시에 복제된 객체의 **불변식을** 보장해야 합니다.
``` java
// 가변 상태를 참조하는 클래스용 clone 메서드
@Override
public PhoneNumber clone() {
    try{
        Stack result = (Stack) super.clone();
        result.elements = elements.clone();
        return result;
    } catch (CloneNotSupportedException e) {
        throw new AssertionsError();
    }
}
```


