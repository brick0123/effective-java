# [Item 88] readObject 메서드는 방어적으로 작성하라.

``` java
// 방어적 복사를 사용하는 불변 클ㄹ스

public final class Period {

    private final Date start;
    private final Date end;

    /**
    * @param start 시작 시간
    * @param end 종료 시각. 시작 시간보다 뒤어야 한다
    * @throws IllegalArgumentException 시작 시간이 종료 시간보다 늦을 때 발생한다
    * @throws NullPointerException start나 end가 null일시 발생
    */
    public Period(Date start, Date end) {
        this.start = new Date(start.getTime());
        this.end   = new Date(end.getTime());
            if (this.start.compareTo(this.end) > 0)
                throw new IllegalArgumentException(tart + " after " + end);
    }

    public Date start () { return new Date(start.getTime()); }
    public Date end () { return new Date(end.getTime()); }
    public String toString() { return start + " - " + end; }
    ...
   }
```

Period에 기본 직렬화 방법을 이용하면 주요한 불변식을 더는 보장하지 못하게 된다. readObject 메서드가 실질적으로 또 다른 public 생성자이기 때문이다. readObject는 매개변수로 바이트 스틤을 받는 생성자라고 할 수 있다. 불변식을 깨디를 의도로 임의 생성한 바이트 스트림을 건네면 정상적인 생성자로는 만들어낼 수 없는 객체를 생성해 낼 수 있기 떄문이다.

</br></br>
 이러한 문제를 해결하려면 Period의 readObject 메서드가 defaultReadObject를 호출한 다음 역직렬화된 객체가 유효한지 검사해야한다. 유효성 검사에 실패하면 InvalidObjectException을 던져서 역직렬화를 막을 수 있다.

 ``` java
 // 유효성 검사를 수행하는 readObject method - 아직 부족하다
private void readObject(ObjectInputStream s) throws IOException, ClassNotFoundException {
    s.defaultReadObject();

    // 불변식을 만족하는지 검사한다
    if (start.compareTo(end) > 0)
        throw new InvalidObjectException(start +" after "+ end);
}
 ```

 아직 맹점이 남아 있다. 정상 Period 인스턴스에서 시작된 바이트 스트림 끝에 private Date 필드로의 참조를 추가하면 가변 Period 인스턴스를 만들어낼 수 있다. ObjectInpuStream에서 Period 인스턴스를 읽은 후 스트림 끝에 추가된 악의적인 참조를 읽어 Period객체의 내부 정보를 얻을 수 있다.
 </br>
 이제 이 참조로 얻은 Date 인스턴스들은 수정할 수 있어서 Period는 더는 불변이 아니게 된다.

 ``` java
 // 가변 공격의 예
 public class MutablePeriod {

  public final Period period;
  public final Date start;
  public final Date end;

  public MutablePeriod() {
    try {
      ByteArrayOutputStream bos = new ByteArrayOutputStream();
      ObjectOutputStream out = new ObjectOutputStream(bos);

      // 유효한 Period 인스턴스를 직렬화한다.
      out.writeObject(new Period(new Date(), new Date()));

      /*
       * 악의적인 '이진 객체 참조', 즉 내부 Date 피드로의 참조를 추가한다.
       */
      byte[] ref = {0x71, 0, 0x7e, 0, 5}; // 참조 #5
      bos.write(ref); // 시작(start) 필드
      ref[4] = 4; // 참조 #4
      out.write(ref); // 종료(end) 필드
      
      // Period 역직렬화 후 Date 참조룰 '훔친다'
      ObjectInputStream in = new ObjectInputStream(new ByteArrayInputStream(bos.toByteArray()));
      period = (Period) in.readObject();
      start = (Date) in.readObject();
      end = (Date) in.readObject();
      
    } catch (ClassNotFoundException | IOException e) {
      throw new AssertionError(e);
    }
  }
}
 ```

실제 공격이 이루어지는 코드

``` java
public static void main(String[] args) {
    MutablePeriod mp = new MutablePeriod();
    Period p = mp.period;
    Date pEnd = mp.end;

    // 시간을 되돌리자
    pEnd.setYear(78);
    System.out.println(p);

    // 60년대로 회귀
    pEnd.setYear(69);
    System.out.println(p);
}
```

심각한 보안 문제로 이어질 수 있다. 실제로 보안 문제를 String이 불변이라는 사실에 기댄 클래스들이 존재하기 많기 때문에 극단적인 예제라고 볼 수 없다. 객체를 역직렬화할 때는 클라이언트가 소유해서는 안 되는 객체 참조를 갖는 필드를 모두 반드시 방어적으로 복사해야한다. 띠리사 readObject에서는 불변 클래스 안의 모든 private 가변 요소를 방어적으로 복사해야한다.

``` java
// 방어적 복사와 유효성 검사를 수행하는 readObject 메서드
private void readObject(ObjectInputStream s) throws IOException, ClassNotFoundException {
    s.defaultReadObject();

    // 가변 요소를 방어적으로 복사한다.
    start = new Date(start.getTime());
    end   = new Date(end.getTime());

    // 불변식을 만족하는지 검사
    if (start.compareTo(end) > 0)
        throw new InvalidObjectException(start +" after "+ end);
}
```
final 필드는 방어적 복사가 불가능하다. 따라서 start와 end 필드에 final을 제거해야한다.

</br>
</br>

기본 readObject 메서드를 사용해도 괜찮을 때
- transient 필드를 제외한 모든 필드의 값을 매개변수로 받아 유효성 검사 없이 필드에 대입하는 public 생성자를 추가해도 괜찮을 때
