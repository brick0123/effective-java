# [Item 64] 객체는 인터페이스를 사용해 참조하라.

``` java
// 좋은 예
Set<Student> student = new HashSet<>();
```


``` java
// 나쁜 예. 클래스 참조
HashSet<Student> student = new HashSet<>();
```
인터페이스를 활용하면 유연함을 얻을 수 있습니다. 구현체를 교체하고 싶으면 구현 클래스만 바꾸면 됩니다.

``` java
// HashSet -> LinkedHashSet 교체
Set<Student> student = new LinkedHashSet<>();
```
하지만 구현체를 바꿀 경우 주의할 점이 있습니다. 기존 구현체만의 특별한 기능을 제공하는 게 있다면, 바꿀 구현체에도 있는지 혹은 사이드 이펙트도 충분히 고려해야 합니다. 물론 적합한 인터페이스가 없을 경우는 당연히 클래스를 참조해야한다. 대표적으로 `String` 등이 있습니다.
