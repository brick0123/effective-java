# [Item 54] null이 아닌, 빈 컬렉션이나 배열을 반환하라.

``` java
// 컬렉션이 비어있으면 null을 반환 - 따라하지 말 것
private final List<Cheese> cheesesInStock = ...;

public List<Cheese> getCheeses() {
    return cheesesInStock.isEmpty() ? null : new ArrayList<>(cheesesInStock);
}
```

null을 반환할 경우 클라이어트는 이 null 상황을 처리하는 코드를 추가로 작성해야합니다.

``` java
List<Cheese> cheesesInStock = shop.getCheeses();
if (cheese != null && cheeses.contains(cheese.STILTON)) {
    system.out.println("좋았어, 바로 그거야!")
}
```

클라이언트에서 null을 처리하는 코드를 빼먹으면 오류가 발생할 수 있으며 코드가 장황해집니다. 때로는 빈 컨테이너를 할당하는 데 비용이 드니 null을 반환하는 게 낫다는 주장도 있습니다. 하지만 이 주장은 두가지 측면에서 틀립니다.</br>
첫 번째. 성능 분석 결과 이 할당이 성능 저하의 주범이라고 확인되지 않는 한, 이 정도의 성능 차이는 신경 쓸 수준이 못됩니다. </br>
두 번째 빈 컬렉션과 배열은 굳이 새로 할당하지 않고도 반환할 수 있습니다.

``` java
// 빈 컬렉션을 반환하는 올바른 예
public List<Cheese> getCheese() {
    return new ArrayList<>(cheeseInStock);
}
```

매번 똑같이 빈 불변 컬렉션을 반환하는 방식으로 성능 이슈도 해결할 수 있습니다. **Collections.emptyList**가 메서드가 대표적인 예입니다. 집합이 필요하면**Collections.emptySet**을, 맵이 필요하면 **Collections.emptyMap**을 사용하면 됩니다. 최적화가 필요하다고 판단되면 사용 전과 후를 측정해보고 사용할 것을 추천합니다.

``` java
public List<Cheese> getCheese() {
    return cheeseInStock.isEmpty() ? Collections.emptyList() : new ArrayList<>(cheeseInStock);
}
```

배열을 사용할 때도 마찬가지입니다. null을 반환 하지 말고 길이기 0인 배열을 반환하면 됩니다.

``` java
public Cheese[] getCheeses() {
    return cheesesInStock.toArray(new Cheese[0]);
}
```
이 방식에서 성능을 더 개선할 수 있는 방법이 있습니다. 길이가 0짜리 배열을 미리 선언해두고 반환하면 됩니다.
``` java
private static final Cheese[] EMPTY_CHEESE_ARRAY = new Cheese[0];
   public Cheese[] getCheeses() {
       return cheesesInStock.toArray(EMPTY_CHEESE_ARRAY);
}
```