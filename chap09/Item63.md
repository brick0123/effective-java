# # [Item 63] 문자열 연결은 느리니 주의하라.

문자열 연결 연산자로 문자열을 n개를 잇는 시간은 n^2에 비례합니다. 문자열은 immutable이라 두 문자열을 연결할 경우 양쪽의 내용을 모두 복사해야 하므로 성능 저하는 피할 수 없습니다.

``` java
public String statement() {
    String result = "";
    for (int i = 0; i < numItems(); i++) {
        result += lineForItem(); // 문자열연결
    }
    return result;
}
```
`StringBuilder`를 활용하면 성능을 크게 개선할 수 있습니다.

``` java
// StringBuilder를 활용한 성능 개g선
public String statement() {
    StringBuilder sb = new SringBuilder(numItems() * LINE_WIDTH);
    for (int i = 0; i < numItems(); i++) {
        b.append(lineForItem(i));
    }
    return b.toString();
}
```
`