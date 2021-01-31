# [Item 73] 추상화 수준에 맞는 예외를 던져라.

메서드가 저수준 예외를 처리하지 않고 바깥으로 전파해버릴 경우 예상치 못한 예외를 접하고 당황할 수가 있습니다. 이 문제를 피하려면 상위 계층에서 저수주 예외를 잡아 자신의 추상화 수준에 맞는 예외를 던져야 합니다. 이를 예외 번역이라 부릅니다.

``` java
try {
    ...
} catch (LowLevelException e) {
    // 추상화 수준에 맞게 번역.
    throw new HighLevelException(..);
}
```
`AbstractSequentialList`에서 수행하는 예외번역의 예시

``` java
    public E get(int index) {
        try {
            return listIterator(index).next();
        } catch (NoSuchElementException exc) {
            throw new IndexOutOfBoundsException("Index: "+index);
        }
    }
```

예외 번역을 할 때, 저수준 예외가 디버깅에 도움이 된다면 예외 연쇄룰 사용하는게 좋습니다. 예외 연쇄란 근본 원인인 저수준 예외를 고수준 예외에 실어 보내는 방식입니다. 그러면 별도의 접근자 메서드(Throwable의 getCause 메서드)를 통해 필요하면 저수준 예외를 꺼내 볼 수 도 있습니다.

``` java
try {
    ... // 저수준 추상화를 이용한다.

} catch (LowerLevelException cause) {
    throw new HigherLevelException(cause);
}
```
대부분 표준 예외는 예외 연쇄용 생성자를 갖추고 있습니다. 예외 연쇄는 문제의 원인을 프로그램에서 접근할 수 있게 해주며, 고수준 예외의 스택 추적 정보를 잘 통합해줍니다. 그렇다고 남용하지는 맙시다. 가능한 저수준 메서드가 반드시 성공하도록 하여 아래 계층에서 예외가 발생하지 않도록 하는 것이 베스트입니다.