# [Item 77] 예외를 무시하지 마라.

발생할 수 있는 실수
``` java

try {
    ...
} catch (IllegalArgumentException e){
}
```
초보 프로그래머가 할 수 있는 실수입니다. catch 블록에서 아무것도 하지 않으면 catch가 존재할 이유가 없어지는 것과 마찬가지입니다. 예외가 발생허더라도 계속 지나칠 수 있습니다. 이는 심각한 결함으로 이어질 수 있습니다. 만약 예외를 무시하기로 결정한 부분이라면 IllegalArgumentException ignored로 바꿔놓는 것이 좋습니다.