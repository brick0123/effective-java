# [Item 89] 인스턴스 수를 통제해야 한다면 readResolve 보단는 열거타입을 사용하라.

``` java
// 싱글턴
public class Elvis {
    public static final Elvis INSTANCE = new Elvis(); 
    private Elvis() { ... }
       
    public void leaveTheBuilding() { ... }
}
```
Serializable을 구현하는 순간 더이상 싱글턴이 아니다. 어떤 readObject를 사용하든 초기화될때 만들어진 인스턴스와는 별개인 인스턴스를 반환하게 된다.
</br>
</br>
readResolve 기능을 활용하면 readObject가 만들어낸 인스턴스를 다른 것으로 대체할 수 있다

``` java
// 인스턴스 통제를 위한 readResolve - 개선의 여지가 있다.
private Object readResolve() {

    // wlsWK Elvis를 반환하고, 가짜 Elvis는 가비지 컬렉터에 맡긴다.
    return INSTANCE;
}
```

readResolve를 인스턴스 통제 목적으로 사용한다면 객체 참조 타입 인스턴스 필드는 모두 transient로 선언해야 한다. 그렇지 않으면 역직렬화된 객체의 참조를 공격할 여지가 남는다.

</br>
</br>

싱글턴이 transient가 아닌 참조 필드를 갖고 있다면, 그 필드의 내용은 readResolve 메서드가 실행되기 전에 역직렬화된다. 그렇다면 잘 조작된 스트림을 써서 참조 필드의 내용이 역직렬화 되는 시점에 그 역직렬화된 인스턴스의 참조를 훔쳐올 수 있다.

``` java
// 잘못된 싱글턴 - transient가 아닌 참조 필드를 가지고 있다.
public class Elvis implements Serializable {
    public static final Elvis INSTANCE = new Elvis();
    private Elvis() { }

    private String[] favoriteSongs = { "Hound Dog", "Heartbreak Hotel" };

    public void printFavorites() {
        System.out.println(Arrays.toString(favoriteSongs));
    }
    private Object readResolve() {
           return INSTANCE;
    } 
}
```

``` java
// 도둑 클래스

public class ElvisStealer implements Serializable {
    static Elvis impersonator;
    private Elvis payload;

    private Object readResolve() {
        // resolve되기 전에 Elvis 인스턴스의 참조를 저장한ㄷ.
        impersonator = payload;

        // favoriteSongs 필드에 맞는 타입의 객체를 반환한다..
        return new String[] { "A Fool Such as I" };
       }

    private static final long serialVersionUID = 0;
   }
```

``` java
// 직렬화의 허점을 이용해 싱글턴 객체를 2개 생성한다
public class ElvisImpersonator {
     // 진짜 Elvis 인스턴스로는 만들어질 수 없는 바이트 스트림.
     private static final byte[] serializedForm = {
       (byte)0xac, (byte)0xed, 0x00, 0x05, 0x73, 0x72, 0x00, 0x05,
       0x45, 0x6c, 0x76, 0x69, 0x73, (byte)0x84, (byte)0xe6,
       (byte)0x93, 0x33, (byte)0xc3, (byte)0xf4, (byte)0x8b,
       0x32, 0x02, 0x00, 0x01, 0x4c, 0x00, 0x0d, 0x66, 0x61, 0x76,
       0x6f, 0x72, 0x69, 0x74, 0x65, 0x53, 0x6f, 0x6e, 0x67, 0x73,
       0x74, 0x00, 0x12, 0x4c, 0x6a, 0x61, 0x76, 0x61, 0x2f, 0x6c,
       0x61, 0x6e, 0x67, 0x2f, 0x4f, 0x62, 0x6a, 0x65, 0x63, 0x74,
       0x3b, 0x78, 0x70, 0x73, 0x72, 0x00, 0x0c, 0x45, 0x6c, 0x76,
       0x69, 0x73, 0x53, 0x74, 0x65, 0x61, 0x6c, 0x65, 0x72, 0x00,
       0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x02, 0x00, 0x01,
       0x4c, 0x00, 0x07, 0x70, 0x61, 0x79, 0x6c, 0x6f, 0x61, 0x64,
       0x74, 0x00, 0x07, 0x4c, 0x45, 0x6c, 0x76, 0x69, 0x73, 0x3b,
       0x78, 0x70, 0x71, 0x00, 0x7e, 0x00, 0x02
};
     
    public static void main(String[] args) {
        // ElvisStealer.impersonator를 초기화 한 다음, 진짜 Elvis(Elvis.INSTANCE)를 반환
        Elvis elvis = (Elvis) deserialize(serializedForm); 
        Elvis impersonator = ElvisStealer.impersonator;

        elvis.printFavorites();
        impersonator.printFavorites();
     }
}
```

출력 결과. 서로 다른 2개의 인스턴스를 생성할 수 있음을 증명했다.

>[Hound Dog, Heartbreak Hotel] </br>[A Fool Such as I]

``` java
// 열거 타임 싱글턴 - 전통적인 싱글턴보다 우수하다.
public enum Elvis {
    INSTANCE;
    private String[] favoriteSongs = { "Hound Dog", "Heartbreak Hotel" };

    public void printFavorites() {
    System.out.println(Arrays.toString(favoriteSongs));
    } 
}
```