# [Item 42] 익명 클래스보다는 람다를 사용하라.

자바 8부터는 함수형 프로그래밍을 지원합니다. 추상 메서드가 하나인(Single Abstract Method)인터페이스를 함수형 인터페이스라고 부르며 람다식을 사용해 만들 수 있습니다. 람다는 익명클래스에 비해 코드가 간결하고 가독성이 좋다는 장점이 있습니다.

``` java
// 익명 클래스 방식
Collections.sort(words, new Comparator<String>() {
    @Override
    public int compare(String s1, String s2) {
        return Interget.compare(s1.length(), s2.length());
    }
});
```
``` java
// 람다 활용
Collections.sort(words, 
    (s1, s2) -> Integer.compare(s1.length(), s2.length()));
```
컴파일러가 타입을 추론해주기 때문에 타입을 명시해야 코드가 더 명확해질 때만 제외하고는, 람다의 매개변수 타입은 생략해줍시다. 혹시 컴파일러가 타입을 추론하지 못할경우에 명시해주면 됩니다.</br></br>
람다 자리에 메서드 참조를 활용하면 코드를 더 간결하게 만들어 줄 수 있습니다.
> 문법:</br>
> 클래스이름::메소드이름</br>
> 또는</br>
> 참조변수이름::메소드이름

``` java
Collections.sort(words, comparing(String::length));
```

더 나아가 자바 8 때 List 인터페이스에 추가된 sort 메서드를 이용하면 더욱 짧아집니다.

``` java
words.sort(comparing(String::length));
```

람다를 사용해서 코드 자체로 동작이 명확히 설명되지 않거나, 코드가 더 장황해질 경우 람다를 사용하지 말아야 합니다. 람다는 세 줄안에 끝내는 게 좋습니다.람다가 길거나 읽기 불편하면 리팩터링을 시도하길 추천합니다.
</br>
람다는 함수형 인터페이스에서만 쓰입니다. 예컨대 인터페이스의 추상메서드가 한 개가 넘어가면 람다를 사용할 수 없습니다. 또 람다는 자신을 참조할 수 없습니다. 람다에서 `this` 키워드는 바깥 인스턴스를 가리킵니다.
