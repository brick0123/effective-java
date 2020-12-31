# [Item 36] 비트 필드 대신 EnumSet을 사용하라.

열거한 값들이 집합으로 사용될 경우, 예전에는 상수에 서로 다른 2의 거듭제곱 값을 할당한 정수 열거 패턴을 사용해왔습니다.

``` java
// 비트 필드 열거 상수 - 구닥다리 기법
   public class Text {
       public static final int STYLE_BOLD = 1 << 0; // 1
       public static final int STYLE_ITALIC = 1 << 1; // 2
       public static final int STYLE_UNDERLINE = 1 << 2; // 4
       public static final int STYLE_STRIKETHROUGH = 1 << 3;  // 8

       // 매개변수 styles는 0개 이상의 STYLE_ 상수를 비트별 OR한 값입니다.
       public void applyStyles(int styles) { ... }
   }
```
다음과 같은 식으로 비트별 OR를 사용해 여러 상수를 하나의 집합으로 모을 수 있으며, 이렇게 만들어진 집합을 비트 필드라고 합니다.
``` java
text.applyStyles(Text.STYLE_BOLD | Text.STYLE_ITALIC);
```
비트 필드를 사용하면 비트별 연산을 사용해 집합 연산을 효율적으로 사용할 수 있지만, 정수 열거 상수와 같은 단점을 지니고 있으며 추가로 다음과 같은 단점들이 있습니다.
</br>
1. 비트 필드 값이 그대로 출력되면 정수 열거 상수를 출력할 때보다 해석하기가 훨씬 어렵습니다.
2. 비트 필드 하나에 녹아 있는 모든 원소를 순회하기도 까다롭습니다.
3. 최대 몇 비트가 필요한지를 API 작성 시 미리 예측하여 적절한 타입을 선택해야 합니다. API를 수정하지 않고는 비트 수를 더 늘릴 수 없기 때문입니다.
</br>
`EnumSet`을 이용하면 이러한 단점을 보완하여 사용할 수 있습니다. EnumSet의 내부는 비트 벡터로 구현되어있고, 원소가 총 64개 이하라면 EnumSet 전체를 long 변수 하나로 표현하여 비트 필드에 비견되는 성능을 보여줍니다.

``` java
public class Text {
    public enum Style { BOLD, ITALIC, UNDERLINE, STRIKETHROUGH }

       // 어떤 Set을 넘겨도 됩지만 EnumSet이 가장 좋습니다.
       public void applyStyles(Set<Style> styles) { ... }
}
```
EnumSet을 이용한 클라이언트 코드
``` java
text.applyStyles(EnumSet.of(Style.BOLD, Style.ITALIC));
```
### 정리
- 열거할 수 있는 타입을 모아 집합 형태로 사용한다고 해도 비트 필드를 사용할 이유는 없습니다.
- 비트 필드의 단점이 보완되는 EnumSet을 활용합시다.