# [Item 58] 전통적인 for 문보다는 for-each를 사용하라.

for 문은 코드가 장황해질 수 있고, 요소 종류가 늘어날 수록 오류가 생길 가능성이 있습니다. for-each (정식명칭 향상된 for문)은 이러한 단점들을 해결해줄 수 있습니다.  

``` java
for (Element e : elements) {
    .... // e로 무언가를 한다.
}
```
컬렉션을 중첩해서 사용하면 for-each 문의 이점은 더욱 커집니다.

``` java
// 버그를 찾아보자
enum Suit { CLUB, DIAMOND, HEART, SPADE }
enum Rank { ACE, DEUCE, THREE, FOUR, FIVE, SIX, SEVEN, EIGHT,
            NINE, TEN, JACK, QUEEN, KING }
...

static Collection<Suit> suits = Arrays.asList(Suit.values());
static Collection<Rank> ranks = Arrays.asList(Rank.values());

List<Card> deck = new ArrayList<>();
for (Iterator<Suit> i = suits.iterator(); i.hasNext(); )
    for (Iterator<Rank> j = ranks.iterator(); j.hasNext(); )
        deck.add(new Card(i.next(), j.next()));
```

마지막 줄에 i에서 문제가 발생합니다. i(무늬)가 j(숫자)당 하나씩만 부여되야하는데, i랑 j과 next로 함께 순회하고 있는 것입니다. 이 케이스에너느 결국  `NoSuchElementException`이 발생하게 됩니다.
</br>
이러한 문제점들은 for-each문을 이용해서 간단하게 해결할 수 있습니다.

``` java
    for (Suit suit : suits)
      for (Rank rank : ranks) {
        deck.add(new Card(suit, rank));
      }
  }
```

하지만 몬든 상황에서 for-each 문을 사용할 수 있는 건 아닙니다. 다음은 for-each 문을 사용할 수 없는 상황입니다.
- 파괴적인 필터링: 컬렉션을 순회하면서 선택된 원소를 제거하려면 반복자의 remove 메서드를 호출해야 합니다. 자바 8부터는 Collection의 removeIf 메서드를 사용해 컬렉션을 명시적으로 순회하는 일을 피할수 있습니다.
- 변형: 리시트나 배열을 순회하면서 그 원소의 값 일부 홋은 전체를 교체해야 한다면 반복자나 배열의 인덱스를 사용해야 합니다.
- 병렬 반복: 여러 컬렉션을 병렬로 순회해야 한다면 각각의 반복자와 인덱스 변수를 사용해 엄격하고 명시적으로 제어해야 합니다.

