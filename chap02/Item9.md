# [Item 9] try-finally보다는 try-with-resources를 사용하라
자바 라이브러리에는 `InputStream`, `OutputStream`, `java.sql.Connection`과 같이 직접 닫아(close)해줘야 하는 자원들 있습니다. 클라이언트는 실수로 자원을 닫아주지 않는 경우 예상치 못한 성능 문제로 이어질 수 있습니다.
``` java
  static String firstLineOfFile(String path) throws IOException {

    BufferedReader br = new BufferedReader(new FileReader(path));
    try {
      return br.readLine();
    } finally {
      br.close();
    }
  }
```
나쁘진 않지만 자원을 더 사용하게 되면 살수가 나올 가능성이 큽니다.
위 예제 같은 경우 try 블록과 finally 블록 모두에서 발생할 수 있는데, 예컨데 기기에 물리적 문제가 생긴다면 firstLineOfFile 메서드 안의 readLine 메서드가 예외를 던지고, 같은 이유로 close 메서드도 실패합니다. 이런 상황이라면 두 번째 예외가 첫 번째 예외를 완전히 집어삼키게 됩니다.(첫 번째 예외는 정보가 남지 않게 됩니다)
</br></br>
이러한 문제점들을 고안해서 자바 7부터는 `try-with-resources`으로 해결할 수 있습니다. 이 구조를 사용하려면 해당 자원이 AutoCloseable 인터페이스를 구현해야 합니다.</br>자바 라이브러리와 서드파티 라이브러리들의 수많은 클래스와 인터페이스가 이미 Autocloseable을 구현하거나 확정했습니다.</br>
try-with-resources를 사용한 코드
``` java
  static String firstLineOfFile(String path) throws IOException {
    try (BufferedReader br = new BufferedReader(new FileReader(path))) {
      return br.readLine();
    }
```
try-with-resources에서도 catch 절을 사용할 수 있습니다.
``` java
  static String firstLineOfFile(String path) {
    try (BufferedReader br = new BufferedReader(new FileReader(path))) {
      return br.readLine();
    } catch (IOException e) {
      return defaultVal;
    }
  }
```

