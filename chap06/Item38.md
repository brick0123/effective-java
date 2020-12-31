# [Item 37] ordinal 인덱싱 대신 EnumMap을 사용하라.

``` java
public class Plant {
  enum LifeCycle { ANNUAL, PERENNIAL, BIENNIAL }

  final String name;
  final LifeCycle lifeCycle;

  Plant(String name, LifeCycle lifeCycle) {
    this.name = name;
    this.lifeCycle = lifeCycle;
  }

  @Override
  public String toString() {
    return name;
  }
}
```
식물을 간단히 나타낸 코드입니다. 이들을 생애주기 별로 묶어봅시다.생애주기별로 총 3개의 집합을 만들고 정원을 한 바퀴 돌며 식물을 해당 집합에 집어넣습니다.
``` java
   Map<Plant.LifeCycle, Set<Plant>> plantsByLifeCycle =
       new EnumMap<>(Plant.LifeCycle.class);
   for (Plant.LifeCycle lc : Plant.LifeCycle.values())
       plantsByLifeCycle.put(lc, new HashSet<>());
   for (Plant p : garden)
       plantsByLifeCycle.get(p.lifeCycle).add(p);
   System.out.println(plantsByLifeCycle);
```
`EnumMap`은 열거타입 키를 사용하도록 설계한 아주 빠른 Map 구현체 입니다. EnumMap의 생성자가 받는 키 타입의 Class 객체는 한정적 타입 토큰으로 런타임 제네릭 타입 정보를 제공합니다. </br>
스트림을 사용해 맵을 관리하면 코드를 더 줄일 수 있습니다.
``` java
System.out.println(Arrays.stream(garden)
           .collect(groupingBy(p -> p.lifeCycle)));
```
이 코드는 EnumMap이 아닌 고유한 맵 구현체를 사용했기 때문에 EnumMap을 써서 얻은 공간과 성능 이점이 사라진다는 단점이 있습니다. 좀 더 자세히 살펴보면
 매개변수 3개짜리 Collectors.groupingBy 메서드는 mapFactory 매개변수에 원하는 맵 구현체를 명시해 호출할 수 있습니다.
``` java 
// EnumMap을 이용해 데이터 열거 타입과 매핑
 System.out.println(Arrays.stream(garden)
        .collect(groupingBy(p -> p.lifeCycle,
        () -> new EnumMap<>(LifeCycle.class), toSet())));
```
맵을 빈번히 사용하는 프로그램에는 도움이 될 것입니다.</br>
스트림을 사용하면 EnumMap만 사용했을 때와는 다른 점이 있습니다. EnumMap 버전은 언제나 식물의 생애주기당 하나씩의 중첩 맵을 만들지만, 스트림 버전은 해당 생애주기에 속하는 식물이 있을 떄만 만듭니다.
