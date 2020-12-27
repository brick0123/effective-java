# [Item 33] 타입 안전 이종 컨테이너를 고려하라
타입 안전 이종 컨테이너 패턴이란 키를 매개변수화한 다음, 컨테이너에 값을 넣거나 뺄 때 매개변수화한 키를 함께 제공하는 방식입니다.
``` java
// 타입 안전 이종 컨테이너 패턴 - API
public class Favorites {
    public <T> void putFavorite(Class<T> type, T instance);
    public <T> getFavorite(Class<T> type)
}
```
다음은 Favorite 클래스를 사용하는 예시입니다.
``` java
// 타입 안전 이종 컨태이너 패턴 - 클라이언트
    Favorites f = new Favorites();
    
    f.putFavorite(String.class, "JAVA");
    f.putFavorite(Integer.class, 0xcafebabe);
    f.putFavorite(Class.class, Favorite.class);

    String favoriteString = f.getFavorite(String.class);
    Integer favoriteInteger = f.getFavorite(Integer.class);
    Class<?> favoriteClass = f.getFavorite(Class.class);
```
favorite 인스턴스는 type safe합니다. String을 요청했는데 Integer를 반환할 일이 없습니다.
``` java
// 타입 안전 이종 컨태이너 패턴 - 구현
public class Favorites {
    private Map<Class<?>, Object> favorite = new HashMap<>();
    
    public <T> void putFavorite(Class<T> type, T instance) {
        favorite.put(Objects.requireNonNull(type), instance);
    }

    public <T> T getFavorite(Class<T> type) {
        return type.cast(favorites.get(type));
    }
}

```
 Map<Class<?>, Object>에서 비한정적 와일드카드 타입을 사용해서 값을 아무것도 넣을 수 없을 거라고 생각할 수 있지만, 맵이 아니라 키가 와일드카드 타입이라서 값을 넣을 수 있습니다.</br>
 지금 만든 `Favorites` 클래스에 주의점이 두 가지가 있습니다.</br>
 첫 번째는 Class 객체를 raw type으로 넘기면 Favorites 인스턴스의 타입 안전성이 쉽게 깨집니다. Favorites가 타입 불변식이 어기는 일이 없도록 보장하려면 다음과 같이 수정할 수 있습니다.

 ``` java
public <T> void putFavorite(Class<T> type, T instance) {
    favorites.put(Objects.requireNonNull(type, type.cast(instance)));
}
 ```
 java.util.Collections에는 `checkedSet`, `checkedList`, `checkedMap` 가 대표적으로 이 방식을 적용한 메서드입니다.
 </br>
 두 번째는 실체화 불가 타입에는 사용할 수 없다는 것입니다. String이나 String[]는 저장할 수 있어도 List<String>은 저장할 수 없습니다. List<String>이나 List<Integer>는 List.class라는 같은 Class 객체를 공유하기 때문입니다.
 ### 정리
 - 컬렉션 API로 대표되는 일반적인 제네릭 형태에서는 한 컨테이너가 다룰 수 있는 타입 매개변수의 수가 고정되어 있습니다.
 - 컨테이너 자체가 아닌 키를 타입 매개변수로 바꾸면 이런 제약이 없는 타입 안전 이종 컨테이너를 만들 수 있습니다.
 - 타입 안전 이종 컨테이너는 Class를 키로 쓰며, 이런 식으로 쓰이는 Class 객체를 타입 토큰이라 합니다.