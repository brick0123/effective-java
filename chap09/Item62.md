# [Item 62] 다른 타입이 적절하다면 문자열 사용을 피하라.

문자열은 다른 값 타입을 대신하기엔 적절하지 않습니다.

``` java
// 흔한 타입을 문자열로 처리한 부적절한 예
String compoundKey = className + "#" + i.next();
```
두 요소를 구분해주는 #이 두 요소 중 하나에서 쓰였다면 혼란을 초래할 수 있습니다. 각 요소를 개별로 접근하려면 문자열을 파싱해야 해서 느리고, 귀찮고, 오류 가능성도 커집니다. 이럴 경우 전용 클래스를 새로 만드는 편이 낫습니다. 보통 private 정적 멤버 클래스로 선언합니다.</br>

``` java
public final class ThreadLocal<T> {
    public ThreadLocal() {}
    public void set(T value);
    public T get();
}
```
