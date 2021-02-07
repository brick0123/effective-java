# [Item 80] 스레드보다는 실행자, 태스크, 스트림을 애용하라.

`Excutor Framework`는 인터페이스 기반의 유연한 태스크 실행 기능을 담고 있다.

``` java
ExecutorService exec = Executors.newSingleThreadExecutor();
exec.excute(runnable); // 이 실행자에게 task를 넘김
exec.shutdown(); // graceful하게 종료 (이 작업이 실패하면 VM 자체가 종료되지 않을 것)
```
이 외에도 다양한 기능을 실행할 수 있다. 실행자 서비스를 사용하기에 까다로운 애플리케이션도 있는데, 가벼운 서버라면 Executors.newCachedThreadPool이 일반적으로 좋은 선택이지만 CachedThreadPool은 무거운 프로덕션 서버에는 좋지 못하다. </br>
CachedThreadPool에서는 요청받은 태스크들이 큐에 쌓이지 않고 즉시 스레드에 위임돼 실행한다 가용한 스레드가 없다면 새로 하나를 생성한다, 서버가 무거울 경우 CPU 이용률이 
100%로 치닫고, 새로운 태스크가 도착하는 족족 또 다른 스레드를 생성해 상황을 악화시킨다.
</br>
</br>
따라서 무거운 프로덕션 서버에서는 Executors.newFixedThreadPool이나 완전히통재할 수 있는 ThreadPoolExecutor를 직접 사용하는 편이 좋다.
