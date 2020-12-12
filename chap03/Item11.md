# [Item 11] equals를 재정의하려거든 hashCode도 재정의하라.

equals와 hasoCode를 재정의 하지 않으면 `HashMap`이나 `HashSet`에서 같은 원소를 사용할 때 문제가 발생합니다.
</br>
  1. equals 비교에 사용되는 정보가 변경되지 않는다면, 애플리케이션이 실행되는 동안 객체의 hashCode메서드는 여러번 호출해도 일관된 값을 반환해야 합니다.
  2. equals가 두 객체를 같다고 판단하면 hashCode 또한 같은 값을 반환해야 합니다.
  3. equals가 두 객체를 다르다고 판단해도, 두 객체의 hashCode가 서로 다른 값을 반환할 필요는 없습니다. 단, 다른 객체에 대해서는 다른 값을 반환해야 해쉬테이블의 성능이 좋아집니다.

서로 다른 인스턴스에 대해서 모두 다른 해시코드를 반환하면 좋겠지만 hashCode는 int형이므로 2^32만큼의 경우의 수로 제한되어 있기 때문에 비둘기 집의 원리로 예외가 생길 수 있습니다.</br>
다음은 hashCode를 작성하는 요령입니다.

1. int 변수 result를 선언한 후 값 c로 초기화 합니다. 이때 c는 해당 객체의 첫번째 핵시 필드를 2-a 방식으로 계산한 해시코드입니다.
2. 해당 개게의 나머지 필드 f 각각에 대해서 다음 작업을 수행합니다.</br>
   a. 해당 필드의 해시코드를 c를 계산합니다.</br>
   - 기본 타입 필드라면, Type.hashCode(f)를 수행합니다. 여기서 Type은 해당 기본 타입의 박싱 클래스입니다.
   - 참조 타입 필드면서 이 클래스의 equals 메서드가 이 필드의 equals를 재귀적으로 호출해 비교한다면, 이 필드의 hashCode를 재귀적으로 호출합니다.
   - 필드가 배열이라면, 원소 각각을 별도 필드처럼 다룹니다. 이상의 규칙을 재귀적으로 적용해 각 핵심 원소의 해시코드를 게산한 다음, 2.b방식으로 갱신합니다. 배열에 핵심 원소가 하나도 없다면 단순 상수를 사용하고 모든 원소가 핵심 원소라면 Arrays.hashCode를 사용합니다.
   - 
   </br>
     b. 단계 2.a에서 계산한 해시코드 c로 result를 갱신합니다.
     result = 31 * result + c;
</br>
3. result를 반환합니다.

</br>

`equals` 비교에서 사용되지 않는 필든 '반드시'제외해야 합니다. 그렇지 않으면 맨 처음에 언급했던 두 번째 규약을 어기게됩니다.</br>
숫자를 곱하는 이유는 만약 곱셉 없는 hahCode를 구현하게 되면 모든 아나그램의 해시코드가 같아집니다. 그리고 31을 선택한 이유는 홀수이면서 소수이기 때문입니다.</br>
``` java
전형적인 hashCode 메서드

@Override
public int hashCode() {
    int result = Short.hashCode(areaCode);
    result = 31 * result + Short.hashCode(prefix);
    result = 31 * result + Short.hashCode(lineNum);
    return result;
}
```
``` java
한 줄짜리 hasoCode 메서드 - 성능이 살짝 아쉽습니다.
@Override 
public int hashCode(){
    return Objects.hash(lineNum, prefix, areaCode);
}
```
### 정리
`equals`를 재정의 할 때는 `hashCdoe`도 재정의해야 합니다. 그렇지 않으면 프로글매이 제대로 동작하지 않을 수 있습니다. 서로 다른 인스턴스라면 되도록 해시코드도 서로 다르게 구현해야 합니다.