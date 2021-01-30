# [Item 76] 가능한 한 실패 원자적으로 만들라.

여기서 말하는 실패 원자적이란 호출한 메서드가 실패하더라도 해당 객체는 호출 전 상태로 유지되는 것입니다. 가장 간단한 방법은 불변 객채로 설계하는 것입니다. 가변 객체일 경우 작업 수행 전에 유효성을 검사합는 것입니다.

``` java
public Object pop() {
    if (size == 0) {
        throw new EmptyStachException();
    }
    Object result = el[--size];
    el[size] = null; // 참조 해제
    return result;
}
```
유효성 검사하는 부분이 없어도 ArrayOutOfBoundsException을 던지지만 이는 추상화 수준에 상황에 어울리지 않습니다.