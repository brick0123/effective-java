# [Item 57] 지역변수의 범위를 최소화하라.

지역변수의 범위는 가능한 좁히는 게 좋습니다. 가장 좋은 방법은 선언과 동사에 초기화 해주는 것입니다. 초기화에 필요한 정보가 없다면 정보가 주어질 때까지 선언을 미루는 것입니다.. 물론 `try-catch`문은 이 규칙에서 예외입니다. try 블록 안에서 초기화해야하고 밖에서도 쓰일 경우 try 블록 앞에서 선언해야 합니다.
</br>
이러한 지역변수 초기화는 대표적으로 for (for-each), while로 비교할 수 있습니다.

``` java
// 컬렉션이나 베열을 순회하는 권장 관용구
for (Element e : c) {
    ...
}
```

``` java 
//  빈복자가 필요한 때의 관용구
for (Iterator<Element> i = c.iterator(); i.hasNext;) {
    Elements e = i.next();
    ...
}
```

``` java
Iterator<Element> i = c.iterator();
while (i.hasNext()) { // 버그
    doSomething(i.next());
}
// while문 실수로 버그 발생
Iterator<Element> i2 = c2.iterator();
while (i.hasNext()) { // 버그
    doSomething(i2.next());
}
```
복사 붙여넣기를 하다가 실수로 i를 그대로 사용한 것입니다. i의 유효범위가 살아있어서 발생하는 일입니다. 이는 컴파일 오류도 잡지 못하고 런타임시 엉뚱한 결과를 초래할 수 있습니다. 이러한 일이 발생하지 않으려면 지역변수의 범위를 최소화 해야합니다.