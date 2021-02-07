# [Item 81] wait와 notify보다는 동시성 유틸리티를 애용하라.

wait와 notify는 올바르게 사용하기가 아주 까다로우니 고수준 동시성 유틸리티를 사용하라.
</br>
java.util.concurrent의 고수준 유틸리티는 세 범주로 나눌 수 있다.

- 실헹자 프레임워크
- 동시성 컬렉션
- 동기화 장치

동시성 컬렉션은 높은 동시성에 도달하기 위해 동기화를 각자의 내부에서 구현한다. 따라서 컬렉션에서 동기성을 무력화하는 건 불가능하며 외부에서 락을 추가로 사용하면 속도가 느려진다.
</br>
동시성을 무력화하지 못하므로 여러 메서드를 원자적으로 묶어 호출하는 건 불가능하다. 그레서 여러 기본 동작을 하나의 원자적 동작으로 묶는 '상태 의존적 수정' 메서드들이 추가되었다. 이 메서드들은 유용하여 자바 8에서는 일반 컬렉션 인터페이스에더 디폴트 메서드로 추가되었다.
</br>
</br>

자주 쓰이는 동기화 장치는 `CountDownLatch`, `Semaphore`, `Phaser`가 있습니다.

``` java
// 동시에 실행 시간을 재는 간단한 프레임워크 (wait, notify만으로 구현하려면 난해해진다)
  public static long time(Executor executor, int concurrency, Runnable action) throws InterruptedException {
    CountDownLatch ready = new CountDownLatch(concurrency);
    CountDownLatch start = new CountDownLatch(1);
    CountDownLatch done = new CountDownLatch(concurrency);

    for (int i = 0; i < concurrency; i++) {
      executor.execute(() -> {
        // 타이머에 준비를 마쳤음을 알린다.
        ready.countDown();
        try {
          // 모든 작업자 스레드가 준비될 때까지 기다린다.
          start.await();
          action.run();
        } catch (InterruptedException e) {
          Thread.currentThread().interrupt();
        } finally {
          // 타이머에게 작업을 마쳤음을 알린다.
          done.countDown();
        }
      });
    }

    ready.await(); // 모든 작업자가 준비될 때까지 기다린다.
    long startNanos = System.nanoTime();
    start.countDown(); // 작업자들을 깨운다.
    done.await(); // 모든 작업자가 일을 끝마치기를 기다린다.
    return System.nanoTime() - startNanos;
  }
```
time 메서드에 넘긴 실행자는 concurrency 매개변수로 지정한 동시성 수준만큼 스레드를 생성할 수 있어야 한다. 그렇지 못하면 이 메서드는 결코 끝나지 않을 것이고 이럴 상태를 스레드 기아 교착상태(thread starvation deadlock)라 한다.