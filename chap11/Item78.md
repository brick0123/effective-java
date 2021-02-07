# [Item 78] 공유중인 가변 데이터는 동기화해 사용하라.

동기화란 특정 메서드나 블럭에 한 쓰레드가 접근했을 때, 해당 객체에 락을 걸고 다른 쓰레드가 접근하지 못하도록 하는 것이다.

``` java
public class StopThread {

  private static boolean stopRequested;

  public static void main(String[] args) throws InterruptedException {
    Thread backgroundThread = new Thread(() -> {
      int i = 0;
      while (!stopRequested)
        i++;
    });
    backgroundThread.start();

    TimeUnit.SECONDS.sleep(1);
    stopRequested = true;
  }
}

```
메인 스레드가 1초 후 stopRequested를 true로 설정하고 반복문을 빠져나올 거락 생각할 수 있지만, 끝나지 않고 계속 실행됩니다.</br>
원인은 동기화에 있습니다. 동기화 하지 않으면 메인 스레드가 수정한 값을 백그라운드 스레드가 언제쯤에나 보게 될지 보증할 수 없습니다.
</br>
stopRequested 필드를 동기화해 접근하면 이 문제를 해결할 수 있습니다.

``` java
// 정상적으로 종료한다.
public class StopThread {

  private static boolean stopRequested;

  private static synchronized void requestStop() {
    stopRequested = true;
  }

  private static synchronized boolean stopRequested() {
    return stopRequested;
  }

  public static void main(String[] args) throws InterruptedException {
    Thread backgroundThread = new Thread(() -> {
      int i = 0;
      while (!stopRequested())
        i++;
    });
    backgroundThread.start();

    TimeUnit.SECONDS.sleep(1);
    requestStop();
  }
}
```
쓰기와 읽기 모두가 동기화 되지 않으면 동작을 보장하지 않습니다. 
</br>
</br>
volatile으로 필드를 선언하면 동기화를 생략해도 되고, 속도가 더 빠릅니다. volatile 한정자는 배타적 수행과는 상관없자먼 항상 최근에 기록된 값을 읽게 됨을 보장합니다.

``` java
public class StopThread {

  private static volatile boolean stopRequested;

  public static void main(String[] args) throws InterruptedException {
    Thread backgroundThread = new Thread(() -> {
      int i = 0;
      while (!stopRequested)
        i++;
    });
    backgroundThread.start();

    TimeUnit.SECONDS.sleep(1);
    stopRequested = true;
  }
}
```

하지만 volatile를 사용할 때도 주의할 점이 있습니다.
``` java
public static volatile int num = 0;

public static int gernerateNum() {
    return num++;
}
```
num필드는 원자적으로 접근할 수 있고 어떤 값이든 허용한다. 문제는 ++연산자인데 numgernerateNum는 필드에 두 번 접근한다. 먼저 값을 읽고, 그 다음에 1증가한 값을 저장하는 것이다. 이 두 접근 사이에 다른 스레드가 들어와 값을 읽을 경우 첫번째 스레드와 똑같은 값을 돌려받게 되다. 프로그램이 잘못된 결과를 계사해내는 이런 오류를 safety failure라고 한다.
</br>
</br>

이러한 문제점들에 벗어나느 가장 좋은 방법은 가변 데이터를 공유하지 않는 것이다. 한 스레드가 데이터를 다 수정한 후 다른 스레드에 공유할 때는 해당 객체에서 공유하는 부분만 동기화해도 된다. 그럼 그 객체는 다시 수정할 일이 생기기 전까지 다른 스레드들은 동기화 없이 자유롭게 값을 이어갈 수 있다. 이러한 객체를 effective immutable이라 하며, 다른 스레드에 이런 객체를 건내는 행위는 safe publication이라 한다. 
</br>
객체를 안전하게 발행하는 방법은 많다. 클래스 초기화 과정에서 객체를 정적 필드, volatile 필드, final 필드, 혹은 보통릐 락을 통해 접근하는 필드에 저장해도 되고 동시성 컬렉션에 저장하는 방법도 있다.