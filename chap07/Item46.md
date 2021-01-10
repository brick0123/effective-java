# [Item 46] 스트림에서는 부작용이 없는 함수를 사용하라.

스트림 패러다임의 핵심은 계산을 일련의 변환으로 재구성하는 부분입니다. 이때 각 변환 단계는 가능한 한 이전 단계의 결과를 받아 처리하는 순수 함수여야 하는데, 순수 함수란 입력만이 결과에 영향을 주는 함수를 말합니다. 즉 다른 가변 상태를 참조하지 않고, 함수 스스로도 다른 상태를 변경하지 않습니다.</br>
다음은 텍스트 파일에서 단어별 수를 세어 빈도표를 만드는 코드입니다.

``` java
// 스트림을 이해하지 못한 코드 - 따라 하지 말 것
Map<String, Long> freq = new HashMap<>();
try (Stream<String> words = new Scanner(file).tokens()) {
    words.forEach(word -> {
        freq.merge(word.toLowerCase(), 1L, Long::sum)
    });
}
```
스트림 API의 이점을 살리지 못한 코드입니다. 같은 기능의 반복 코드보다 읽기 어렵고, 유지 보수도 어렵습니다. 심지어 `forEach`에서는 람다가 상태를 수정하기도 합니다.

``` java
// 스트림을 제대로 할용한 예
Map<String, Long> freq = new HashMap<>();
try (Stream<String> words = new Scanner(file).tokens()) {
   freq = words
        .collect(groupingBy(String::toLowerCase, counting()));
}
```
`forEach` 연산은 스트림 결과를 보고할 때만 사용하고, 계산할 때는 사용하지 맙시다.
</br>
다음 코드는 빈도표에서 가장 흔한 단어 10개를 뽑아내는 파이프라인 입니다.

``` java
// Collectors 정적 임포트를 활용해 코드가 짧아지고 가독성을 향상시켰습니다.
List<String> toTen = freq.keySet().stream()
    .sorted(comparing(freq::get).reversed())
    .limit(10)
    .collect(toList());
```
이번엔 `Collectors`의 메서드 기능을 알아보겠습니다. 가장 간단한 맵 수집기로 toMap(keyMapper, valueMapper)가 있습니다.

``` java
    Map<String, String> listToMap = books.stream()
        .collect(toMap(Book::getIsbn, Book::getName));
```
다수의 키가 존재한다면 IllegalStateException을 던지며 종료합니다.</br>
인수를 3개 받는 toMap은 어떤 키와 그 키에 연관된 연관된 원소들 중 하나를 골라 연관짓는 맵을 만들때 유용합니다. 다음 코드는 음악가와 그 음악가의 베스트 앨범을 연관짓고 싶은 일을 수행합니다.

``` java
Map<Artist, Album> toHits = albums.collect(
    toMap(Album::artist, artist -> artist, maxBy(Comparing(Album::sales)));
)
```

인수 3개를 받는 toMap도 있습니다. 어떤 키와 그 키에 연관된 원소들 중 하나를 골라 연관짓는 맵을 만들때 유용합니다. 다음 코드는 음악가의 베스트 앨범을 연관짓는 Map을 만드는 코드입니다.

``` java
Map<Artist, Album> toHits = albums.collect(
    toMap(Album::airtst, a -> a, maxBy(comparing(Album::sales)));
)
```

`groupingBy` 메서드는 입력으로 분류 함수(classifier)를 받고 출력으로는 원소들을 카테고리별로 모아놓은 맵을 담은 수집기를 반환합니다. 분류 함수는 입력받은 원소가 속하는 카테고리를 반환합니다. groupingBy 함수의 가장 간단한 것은 분류 함수 하나를 인수로 받아 맵으로 반환하는 것입니다.
``` java
// ISBN을 key값으로 갖는 맵의 형태입니다.
Map<String, List<Book>> collect = books.stream()
    .collect(groupingBy(Book::getIsbn));
```
인수를 두개 갖는 `groupingBy`함수는 다운스트림 수집기도 함께 명시해야 합니다. 다운 스트림 수집기의 역할은 해당 카테고리의 모든 원소를 담은 스트림으로 부터 값을 생성하는 것입니다. 그 다음 인수를 3개 받는 메서드도 있는데 맵 팩터리도 지정할 수 있습니다.