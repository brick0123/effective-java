# [Item 7] 다 쓴 객체 참조를 해제하라
**메모리 누수가 일어나는 예제 코드**
``` java
public class Stack {
    private Object[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;

    public Stack() {
        elements = new Object[DEFAULT_INITIAL_CAPACITY];
    }

    public void push(Object e) {
        ensureCapacity();
        elements[size++] = e; 
    }

    public Object pop() {
        if (size == 0)
            throw new EmptyStackException();
        return elements[--size]; // 주의
    }

    private void ensureCapacity() {
        if (elements.length == size)
            elements = Arrays.copyOf(elements, 2 * size);
    }
}
```
스택에서 데이터를 `pop`을 실행해도 메모리는 줄어들지 않습니다. 왜냐면 `size`를 감소 시키지만 배열에서 여전히 레퍼런스는 그대로 갖고있기 때문입니다.
</br>
다음과 같이 코드를 수정할 수 있습니다.
``` java
    public Object pop() {
        if (size == 0) {
            throw new EmptyStackException();
        }

        Object value = this.elements[--size];
        this.elements[size] = null;
        return value;
    }
```

size를 감소시킬 뿐만 아니라 실제로 해당 참조를 `null`로 처리한 방법입니다.
</br>
하지만 객체 처리를 null로 처리하는 건 예외적인 경우여야 합니다. 다 쓴 참조를 해제하는 가장 좋은 방법은 다 쓴 참조를 담은 변수를 특정한 스코프 안에서만 사용하는 것입니다.
</br></br>
캐시를 사용할 때도 메모리 누수에 유의해야 합니다. 
- 외부에서 키를 참조하는 동안만 엔트리가 살아있는 캐시가 필요하다면 `WeakHashMap`을 사용합니다.
- 시간이 지날 수록 엔티리의 가치를 떨어뜨리는 방식.


