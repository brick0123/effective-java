# [Item 90] 직렬화된 인스턴스 대신 직렬화 프록시 사용을 검토하라.

직렬화 프록시 패턴을 이용하면 앞서 애기했던 단점들을 줄일 수 있다.먼저 바깥 클래스의 논리적인 상태를 표현하는 중첩 클래스를 private static으로 생성한다. 이 중첩 클래스가 바깥 클래스의 직렬화 프록시다. 이 클래스는 단순히 인스로 넘어온 인스턴스의 데이터만 복사하고 바깥 클래스와 모두 Serializable을 구현해야한다.

</br>
</br>

``` java
// Period 클래스용 직렬화 프록시
private static class SerializationProxy implements Serializable { 
    private final Date start;
    private final Date end;
    
    SerializationProxy(Period p) {
        this.start = p.start;
        this.end = p.end;
    }
    private static final long serialVersionUID = 234098243823485285L; // do (Item 87)
}
```

다음은 바깥 클래스에 다음의 writeReplace 메서드를 추가한다. 직렬화 프록시를 사용하는 모든 클래스에서 그대로 사용하면 된다.

``` java
// 직려롸 프록시 패턴용 writeReplace 메서드
private Object writeReplace() {
    return new SerializationProxy(this);
}
```
이 메서드는 직렬화가 이뤄지기 전에 바깥 클래스의 인스턴스를 직렬화 프록시로 변환해준다. writeReplace 결코 바깥 클래스의 직렬화된 인스턴스를 생성해낼 수 없다.
</br>

``` java
// readObject를 바깥 클래스에 추가. ( 불변 훼손 공격 방어)
private void readObject(ObjectInputStream stream) throws InvalidObjectException {
    throw new InvalidObjectException("Proxy required");
}
```

마지막으로 바깥 클래스와 논리적으로 동일한 인스턴스를 반환하는 readResolve 메서드를 SerializationProxy 클래스에 추가한다. 이 메서드는 역직렬화 시에 직렬화 시스템이 직렬화 프록시를 다시 바깥 클래스의 인스턴스로 변환하게 해준다.

</br>
</br>
readObject 메서드는 공개된 API만을 사용해 바깥 클래스의 인스턴서를 생성하는데 장점은 다음과 같다.
</br>
- 직렬화는 생성자를 이용하지 않고도 인스턴스를 생성하는 기능을 제공하는데, 이 패턴은 직려화의 이런 특성을 상당 부분 제거한다. (인스턴스를 만들 때와 똑같은 생성자 등을 사용해 역직렬화된 인스턴스를 생성함)

따라서 불변식을 만족하는 검사를 하지 않아도 된다.

``` java
// Period.SerializationProxy용 readObject메서드
   private Object readResolve() {
       return new Period(start, end);  // public constructor 사용.
}

```
프록시 수준에서 내부 필드 탈취 공격을 차단해준다. 또한 필드에 final을 선언해도 돼서 Period 클래스를 진정한 불변으로 만들 수 있다. 또한 직렬화 프록시 패턴은 역직렬화한 인스턴스와 원래의 직렬화된 인스턴스의 클래스가 달라도 정상 작동한다.
</br>
</br>

프록시 패턴의 한계
- 클라이언트가 멋대로 확장할 수 있는 클래스에는 적용할 수 없다.
- 객체 그래프에서 순환이 있는 클래스에도 적용할 수 없다.
- 성능이 느리다. 


