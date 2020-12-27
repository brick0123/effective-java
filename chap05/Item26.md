# [Item 26] Raw 타입은 사용하지 마라.

`raw type`이란 제네릭 타입에서에서 타입 파라미터를 전혀 사용하지 않았을 때를 말합니다.
``` java
// raw type
List list = new ArrayList();
```
로 타입은 타입 선언에서 제네릭 타입 정보가 전부 지워진 것처럼 동작하는데, 제네릭이 도래하기 전 코드와 호환성을 위해서 주로 존재합니다.</br>
``` java
// 문자열을 저장하는 컬렉션
private final List names = new ArrayList();
```
위 코드를 사용하면 String대신 다른 타입을 넣어도 오류없이 실행됩니다. (경고 메세지를 보여주긴 합니다.)
``` java
naems.add(1); // "Unchecked call" 경고를 보여줍니다.
```
``` java
String str = (String) names.get(0); // ClassCastException
```
컴파일 오류를 체크하지 못하고 런타임할 때 예외가 발생하게 됩니다. 가장 이상적인 오류는 컴파일할 때 발견하는 것이 좋습니다. 이렇게 되면 런타임에 문제를 겪는 코드와 원인을 제공한 코드가 물리적으로 상당히 떨어져 있을 가능성이 커집니다.
</br>
제네릭을 활용하면 이 정보가 주석이 아닌 타입 선언 자체에 녹아듭니다.
``` java
// 매개변수화된 컬렉션 타입 - 타입 안전성 확보
private final List<String> names = new ArrayList<>();
```

이렇게 선언하면 컴파일러는 names에는 String만 넣어야 함을 컴파일러가 인지하게 됩니다. 이제 names에 엉뚱한 타입의 인스턴스를 넣으려고하면 컴파일 오류가 발생합니다.
``` java
names.add(1); // 컴파일 오류
```
앞에서도 얘기했듯, 로 타입(타입 매개변수가 없는 제네릭 타입)은 절대로 사용해서는 안 됩니다. **로 타입을 쓰면 제네릭이 안겨주는 안전성과 표현력을 모두 잃게 됩니다.** 로 타입을 만들어놓은 이유는 호환성 때문입니다. 자바가 제네릭을 받아들이기까지 거의 10년이 걸린 탓에 제네릭 없이 짠 코드를 모두 수용하면서 제네릭을 사용하는 새로운 코드와도 맞물려 돌아가게 해야만 했습니다.</br>

### List vs List\<Objects>
로 타입 List는 제네릭 타입에서 완전히 발을 뺀 것이고, List<Object>는 모든 타입을 허용한다는 의사를 컴파일러에게 전달한 것입니다. 매개변수로 List를 받는 메서드에 List<String>을 넘길 수 있지만, List<Object>를 받는 메서드에는 넘길 수 없습니다. 이는 제네릭의 하위 타입 규칙 때문입니다.</br>
즉, List<Integer>은 로 타입인 List의 하위타입 이지만, List<Object>의 하위타입은 아닙니다. 그 결과 List<Object>같은 매개변수화 타입을 사용할 때와 달리 List같은 로 타입을 사용하면 타입 안전성을 잃게 됩니다.
</br>

예제를 위한 코드

``` java
// Interger는 Number의 서브 타입입니다.
List<Integer> intList = new ArrayList<>();
List<Number> numberList = intList; // 컴파일 에러
```

``` java
List<Integer> intList = new ArrayList<>();
List rawType = intList; // 오류가 발생하지 않지만, 안전하지 않습니다.
```
이쯤 되면 원소의 타입을 몰라도 되는 로 타입을 쓰고 싶어질 수 있습니다. 그럴 땐 비한정적 와일드 타입(unbounded wildcard types)를 사용하는 대안이 있습니다. 이는 타입 파리머티에 `?`을 작성하면 됩니다.

``` java
// unbounded wildcard type - 타입 세이프하고 유연합니다,
static int numElementsInCommon(Set<?> s1, Set<?> s2) { ... }
```

그렇다면 비한정적 와일드카드 타입인 Set<?>와 로타입인 Set의 차이는 무엇일까? 물음펴가 무언가 멋진 일은 해주는 것일까?</br>
와일드 카드는 type safe 하고 로 타입은 그렇지 않다는 점입니다. 로 타입 원소에는 아무 원소나 넣을 수 있으니 타입 붋변식을 훼손하기 쉽습니다. </br>
반면 Collection<?>에는 (null 이외는) 어떤 원소도 넣을 수 없습니다. 다른 원소를 넣으려 하면 컴파일 할 때 오류 메세지를 보여줍니다. 이러한 제약을 받아들일 수 없다면 제네릭 메서드나 한정적 와일드카드 타입을 사용하면 됩니다.</br>
로 타입을 쓰지 말라는 규칙에도 소소한 예외가 몇 개 있습니다. **class 리터럴에는 로 타입을 써야 합니다. 자바 명세는 class 리터럴에 매개변수화 타입을 사용하지 못하게 했습니다.**(배열과 기보 타입은 허용합니다)</br>
예를 들어 List.class.String[].class, int.class는 허용하고 List<String>.class와 List<?>.class는 허용하지 않습니다.
</br>
두 번째 예외는 instanceof 연산자와 관련이 있습니다. 런타임에는 제네릭 타입 정보가 지워지므로 instanceof 정보가 지워지므로 instanceof 연산자는 비한정적 와일드카드 타입 이외의 매개변수화 타입에는 적용할 수 없습니다.
</br>
### 핵심 정리
- 로 타입을 사용하면 런타임 예외가 일어날 수 있으니 사용하면 안 됩니다.
- 로 타입은 제네릭이 도입되기 이전 코드와의 호환성을 위해 제공될 뿐입니다.
- Set<Object>는 어떤 타입의 객체도 저장할 수 있는 매개변수화 타입이고, Set<?>는 모든 종의 타입 객체만 저장할 수 있는 와일드카드 타입입니다.
  
