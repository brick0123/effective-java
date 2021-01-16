# [Item 52] 다중정의는 신중히 사용하라.

다음은 컬렉션을 집합, 리스트, 그 외로 구분하고자 만든 프로그램입니다.

``` java
public class CollectionClassifier {
    public static String classify(Set<?> s) {
        return "Set";
    }
    public static String classify(List<?> lst) {
        return "List";
    }
    public static String classify(Collection<?> c) {
        return "Unknown Collection";
    }
    public static void main(String[] args) {
        Collection<?>[] collections = {
            new HashSet<String>(),
            new ArrayList<BigInteger>(),
            new HashMap<String, String>().values()
        };
        for (Collection<?> c : collections)
            System.out.println(classify(c));
    } 
}
```
"Set", "List", "Unknown Collection" 차례대로 출력될 거 같지만, 실제로 수행하면 "Unknown Collection"만 연달아 세 번 출력됩니다. 이유는 오버로딩된 세 classify 증 어느 메서드를 호출할 지가 컴파일 타임에 정해지기 때문입니다. 컴파일 타임에서 for문 안에 c타입은 항상 `Collection<?>`타입 입니다. 런타임 시에는 달라지지만, 호출할 메서드를 선택하는 데는 영향을 주지 못합니다.
</br>
</br>
이처럼 직관에 어긋나는 이유는 **재정의한 메서드는 동적으로 선택되고, 다중정의한 메서드는 정적으로 선택되기 때문입니다.** 메서드를 재정의 한 다음 하위클래스의 인스턴스에서 그 메서드를 호출하면 재정의한 메서드가 실행됩니다. 컴파일 타임에 그 인스턴스 타입이 무엇인지는 상관 없습니다. </br>
다음 코드는 이러한 상황을 구체적으로 보여주는 예시입니다.

``` java
class Wine {
  String name() {
    return "wine";
  }
}

class SparklingWine extends Wine {
  @Override
  String name() {
    return "sparkling wine";
  }
}

class Champagne extends SparklingWine {
  @Override
  String name() {
    return "champagne";
  }
}

class Overriding {
  public static void main(String[] args) {
    List<Wine> wineList = Arrays.asList(
        new Wine(), new SparklingWine(), new Champagne());

    for (Wine wine : wineList) {
      System.out.println(wine.name());
    }
  }
}
``` 
`Wine`클래스에서 정의한 name메서드를 각각 서브타입에서 재정의 했습니다. 이 코드를 실행하면 아까와는 달리 "wine","sparkling wine", "champagne"이 차례대로 출력됩니다. 컴파일 타임 타입이 모두 Wine인 것에 무관하게 항상 가장 하위에서 재정의한 메서드가 실행됩니다.</br>
</br>
한편 다중정의된 메서드 사이에서는 객체의 런타임 타입은 전혀 중요하지 않습니다. 선택은 컴파일 타임에, 오직 매개변수의 컴파일 타임 타입에 의해 이뤄집니다. 앞서 의도대로 동작하지 않았던 classify 메서드는 하나로 합친 후 instanceof를 이용하면 원하는 데로 동작하게 변경할 수 있습니다.
</br>
다중정의를 안전하고 보수적으로 사용하려면 매개변수 수가 같은 다중정의는 만들지 않는 게 좋습니다. 다중정의 하는 대신 메서드 이름을 다르게 지어주는 경우도 고려해볼 수 있습니다. 대표적인 예로 `ObjectOutputStream`클래스의 write 메서드가 그러합니다.

</br></br>
자바 4까지는 기본 타입이 모든 참조 타입이랑 근본적으로 달랐지만, 자바 5에서 부터 오토박싱이 도입되면서 문제가 생긴 부분이 있습니다. 대표적으로 `List` 인터페이스의 `remove`메서드가 그러합니다. remove메서드는 int를 받는 메서드와 Object를 매개변수로 받는 메서드로 다중정의 되어 있습니다. 물론 자바 4까지는 문제가 없었지만 오토박싱이 생기면서 혼란을 안겨주고 List인터페이스가 취약해졌습니다.
