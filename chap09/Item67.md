# [Item 67] 최적화는 신중히 하라.

최적화는 좋은 결과보다 해로운 결과로 이어지기 쉽고, 자칫하면 빠르지도 않고 제대로 동작하지 않는 소프트웨어를 탄생시키는 것과 같습니다. 그러므로 최적화 할때는 득과실을 잘 생각해봐야 합니다.

### 성능을 제한하는 설계를 피하라
완성 후 변경하기가 가장 어려운 설계 요소는 바로 컴포넌트끼리 혹은 외부 시스템과의 소통 방식입니다. 대표적으로 API, 네트워크 프로토콜 등이 있습ㄴ디ㅏ. 이런 설계 요소들은 완성 후에는 변경하기 어려우며 동시에 시스템 성능을 심각하게 제한할 수 있습니다.

### API 설계할 때 성능에 주는 영향을 고려하라.
public 타입을 가변으로 만들면 불필요한 방어적 복사를 유발할 수 있습니다. 또한 상속 방식으로 설계한 pulblic 클래스는 상위 클래스에 영원히 종석되며 성능 제약까지 물려받습니다.
