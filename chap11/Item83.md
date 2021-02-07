# [Item 83] 지연 초기화는 신중히 사용하라.

지연 초기화는 필드의 초기화 시점을 그 값이 처음 필요햔 시점까지 늦추는 것이다. 지연 초기화는 인스턴스 생성시 초기화 비용을 줄일 수 있지만, 지연 초기화하는 필드에 접근하는 비용은 커진다. 지연 초기화를 잘못사용하면 실제로 성능이 더 느려질 수도 있다.
</br>
</br>
멀티 스레드 환경에서 지연 초기화를 하기에는 까다롭다. 지연 초기화하는 필드를 둘 이상 스레드가 공유한다면 동기화해야 한다. 일반적으로 일반적인 초기화가 지연 초기화보다 낫다.

``` java
// 인스턴스 필드를 초기화하는 일반적인 방법
private final FieldType field = computerFieldValue();
```

``` java
// 지연 초기화 - sychronized 방식
private FieldType field;

private sychronized FieldType getField() {
    if (field == null) {
        field = computerFieldValue();
    }
    return field;
}
```

``` java
// 정적 필드용 지연 초기화 홀더 클래스 관용구
private static class FieldHolder {
    static final FieldType field = computerFieldValue();
}

private static FieldType getField() {
    return FieldHolder.field();
}
```
getField가 호출되는 순간 FieldHolder.field가 처음 읽히면서 FieldHolder 클래스 초기화를 촉발한다. 일반적인 VM은 클래스를 초기화 할때만 필드 접근을 동기화할 것이다. 클래스 초기화가 끝난 후에는 VM이 동기화 코드를 제거하여, 그 다음부터는 아무런 검사나 동기화 없이 필드에 접근하게 된다.

</br></br>
성능 때문에 인스턴스 필드를 지연 초기화 한다면 이중검사 관용구를 사용하라. 한 번은 동기화 없이 검사하고 두 번째는 동기화하여 검사한다. 두 번째 검사에서 필드가 초기화되지 않았을 때만 초기화한다.

``` java
  private volatile FieldType fieldType;

  private FieldType getFieldType() {
    FieldType result = fieldType;
    if (result != null) { // 첫 번째 검사 (락 사용 x)
      return result;
    }
    
    synchronized (this) {
      if (fieldType == null) { // 두 번째 검사 (락 사용)
        fieldType = computerFieldValue();
      }
      return fieldType;
    }
  }
```
