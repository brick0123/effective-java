# [Item 85] 자바 직렬화의 대안을 찾으라.

자바의 역직렬화에는 치명적인 단점이 있습니다. 신롸할 수 없는 스트림을 역직렬화하면 원격 코드 실행, 서비스 거부 등의 공격으로 이어질 수 있습니다.

</br>
</br>
역직렬화 폭탄이란 서비스 거부 공격을 유발하는 스트림입니다.

``` java
static byte[] bomb() {
    Set<Object> root = new HashSet<>();
    Set<Object> s1 = root;
    Set<Object> s2 = new HashSet<>();
    for (int i = 0; i < 100; i++) {
        Set<Object> t1 = new HashSet<>();
        Set<Object> t2 = new HashSet<>();
        t1.add("foo"); // t1을 t2와 다르게 만든다
        s1.add(t1); s1.add(t2);
        s2.add(t1); s2.add(t2);
        s1 = t1;
        s2 = t2;
    }
    return serialize(root); 
}
```
이 코드는 끝날 기미가 보이지 않는다. HashSet 인스턴스를 역직렬화 하려면 원소들의 해시코드를 계산해야 하는데 위 코드에서는 hashCode 메서드를 2^100번 넘게 호출해야합니다.
</br>
이러한 문제를 해결하는 가장 좋은 방법은 사용하지 않는 것입니다. 자바 직렬화보다 훨씬 간단하고 문제를 회피할 수 있는 방법으로 JSON과 프로토콜 버퍼가 대표적으로 있습니다.