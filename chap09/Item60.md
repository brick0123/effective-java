# [Item 60] 정확한 답이 필요하다면 float와 double은 피하라.

float와 dobule은 과학과 공학 계산용으로 설계 되었습니다. 이진 부동소수점 연산에 쓰이며, 넓은 범위의 수를 빠르게 정밀한 '근사치'로 계산하도록 설계되었습니다. 따라서 정확한 계산 결과가 필요할 때는 사용하면 안 됩니다. 특 히 금용 관련 계산과 맞지 않습니다. 0.1 혹은 10의 음의 거듭 제곱수(10^-1, 10-^2)를 표현할 수 없기 때문입니다.</br>
예를 들어 1.03달러에서 42센트를 사용하고 남은 돈을 계산한다고 가정해봅시다.
``` java
System.out.println(1.03  - 0.42);
```
이 코드는 0.6100000000000001를 출력합니다. 원하는 결과가 나오지 않음을 알 수 있습니다.</br>
다음 예제는 주머니에는 1달러가 있고 10,20 .. 1달러 짜리의 사탕이 있습니다. 10센트짜리부터 하나씩 얼마나 살 수 있고, 잔돈을 확인하는 어설픈 코드입니다.

``` java
double funds = 1.00;
int itemsBought = 0;
for (double price = 0.10; funds >= price; price += 0.10) {
    funds -= price;
    itemsBought++;
}
System.out.println(itemsBought + " items bought.");
System.out.println("Change: $" + funds);  
```
실행 결과 3개를 구매했고, 잔돈은 0.3999999999999999 달러가 출력되는 걸 확인할 수 있습니다. 이러한 금융 계산 문제에는 `BigDecimal`, `int` 혹은 `long`을 사용해야 합니다.

다음 코드는 BigDecimal로 교체한 코드입니다. 

``` java
final BigDecimal TEN_CENTS = new BigDecimal(".10");

int itemsBought = 0;
BigDecimal funds = new BigDecimal("1.00"); 

for (BigDecimal price = TEN_CENTS;
        funds.compareTo(price) >= 0;
        price = price.add(TEN_CENTS)) {
    funds = funds.subtract(price);
    itemsBought++;
    }
System.out.println(itemsBought + " items bought.");
System.out.println("Money left over: $" + funds);
```
4개를 구매했고 잔돈은 0이 나온 걸 확인할 수 있습니다. 하지만 Dedecimal에는 두 가지 단점이 존재합니다. 기본타입보다 쓰기 불편하고 훨씬 느립니다. Bigdecimal의 대안으로 int, long를 사용할 수도 있습니다. 그럴경우 소숫점 관리랑 값의 크기는 따로 관리해줘야합니다.