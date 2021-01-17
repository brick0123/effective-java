# [Item 47] 반환 타입으로는 스트림보다 컬렉션이 낫다.

사실 `Stream` 인터페이스 `Iterable` 인터페이스가 정의한 추상 메서드를 전부 포함하고 Iterable 인터페이스가 정의한 방식대로 동작합니다. 하지만 for-each로 스트림을 반복할 수 없는 이유는 Stream이 Iterable을 확장하지 않아서입니다.
</br>
`Collection`인터페이스는 Iterable의 하위 타입이고 stream의 메서드도 제공하니 반복과 스트림을 동시에 지원합니다. 따라서 원소 시퀀수를 공개하는 공개 API의 반환타입에는 Collection이나 그 하위 타입을 사용하는게 일반적으로 좋습니다.

### 정리
- 원소 시퀀스를 반환하는 메서드를 작성할 때는, 이를 스트림으로 처리하길 원하는 사용자와 반복으로 처리하길 원하는 사용자가 모두 있을 수 있음을 고려하라
- 컬렉션을 반환할 수 있다면 그렇게 하고 반환 전부터 이미 원소들을 컬렉션에 관리하고 있거나 컬렉션을 하나 더 만들어도 될 정도로 원소 개수가 적다면 ArrayList같은 표준 컬렉션에 담아 반환하라.
