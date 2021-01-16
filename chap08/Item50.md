# [Item50] 적시에 방어적 복사본을 만들라

이번 아이템에서는 지난 Item17에 다루었던 불변에 대한 주제가 포함되어있습니다. 어떤 객체든 객체의 허락 없이는 외부에서 함부로 내부를 수정하게 하는 일이 없아야 합니다. 하지만 주의를 기울이지 않으면 자신도 모르게 내부를 수정하도록 코드를 짜는 경우가 생길 수 있습니다.
</br>
다음은 부주의로 일어날 수 있는 상황을 예시로 든 코드입니다.

``` java
public final class Period {
    private final Date date;
    private final Date end;

    public Period(Date start, Date end) {
        if (start.compareTo(end) > 0) {
            throw new IllegalArgumentException(start + "가" + end + "보다 늦다.");
        }
        this.start = start;
        this.end = end;
    }

    public Date start() {
        return start;
    }

    public Date end() {
        return end;
    }
}
```

불변을 의도하고 설계한 코드입니다. 하지만 `Date`가 mutable 하다는 사실을 이용하여 이 불변을 쉽게 깨뜨릴 수 있습니다.

``` java
Date start = new Date();
Date end = new Date();
Period p = new Period(start, end);
end.setYear(100); // p의 내부를 수정했다
```
이러한 문제점을 해결하기 위해 자바 8에서는 `java.time` 패키지를 지원합니다. 하지만 기존 레거시 프로젝트에 `Date`는 여전히 많이 남아있습니다. 이번에 소개해드릴 방법은 이러한 레거시 코드들을 대처하기 위함입니다. 외부로부터 `Period` 인스턴스의 내부를 보호하려면 생성자에게 받은 가변 매개변수 각각을 방어적으로 복사해야합니다. 그런 다음 Period 인스턴스 안에서는 원본이 아닌 복사본을 사용해야 합니다.

``` java
// 수정한 생성자 - 매개변수의 방어적 복사본을 만듦
public Period(Date start, Date end) {
    this.start = new Date(start.getTime());
    this.end = new Date(end.getTime());

    if (this.start.compareTo(this.end) > 0) {
        throw new IllegalArgumentException(start + "가" + end + "보다 늦다.");
    }
}
```
이 방법을 이용하면 방금의 방식으로 값을 바꾸려고하는 걸 막을 수 있습니다. 매개변수의 유효성을 검하기 전에 방어적 복사본을 만들고, 이 복사본으로 유혀성을 검사한다는 점에 주목해야합니다. 하지만 아직도 Period 객체는 변경 가능합니다. 접근자 메서드가 내부의 가변 정보를 직접 드러내기 떄문입니다.

``` java
Date start = new Date();
Date end = new Date();
Period p = new Period(start, end);
p.end().setYear(100); // p의 내부를 변경했다.
```

이 공격을 막아내려면 단순히 접근자가 가변 필드의 방어적 본사본을 반환하거나 `setter`를 제공하지 않으면 됩니다.

``` java
public Date start() {
    return new Date(start.getTime());
}

public Date end() {
    return new Date(end.getTime());
}
```

이제 Period는 완벽한 불변으로 거듭납니다. 메서드든 생성자든 클라이언트가 제공한 객체의 참조를 내부의 자료구조에 보관해야 할 때면 항시 그 객체가 잠재적으로 변경될 수 있는지를 생각해야합니다. 방어적 복사에는 성능 저하가 따른다는 것도 명심해야하고 되도록 불변 객체들을 조합해 객체를 구성해야 방어적 복사를 할 일이 줄어듭니다.