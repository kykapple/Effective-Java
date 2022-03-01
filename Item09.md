# Item09. `try-finally`보다는 `try-with-resource`를 사용하라
- 자바 라이브러리에는 `InputStream`, `OutputStream`, `java.sql.Connection` 등과 같이 `close` 메서드를 통해 직접 닫아주어야 하는 자원들이 많다.
- 그러나 이러한 자원들은 클라이언트가 놓치기 쉬워서 예측할 수 없는 성능 문제로 이어지기도 한다.
- 자원이 제대로 닫힘을 보장하는 `try-finally`와 `try-with-resource`에 대해 알아보도록 하자. 

### `try-finally`
- 전통적으로 자원이 제대로 닫힘을 보장하는 수단으로 사용되었다.
```java
public String firstLineOfFile(String path) throws IOException {
    BufferedReader br = new BufferedReader(new FileReader(path));
    try {
        return br.readLine();
    } finally {
        br.close();
    }
}

public void copy(String src, String dst) throws IOException {
    InputStream in = new FileInputStream(src);
    try {
        OutputStream out = new FileOutputStream(dst);
        try {
            byte[] buf = new byte[BUFFER_SIZE];
            int n;
            while ((n = in.read(buf)) >= 0)
                out.write(buf, 0, n);
        } finally {
            out.close();
        }
    } finally {
        in.close();
    }
    
}
```
- 위 2개의 메서드 모두 예외에 대한 결점이 있다.
- 예를 들어 기기에 물리적인 문제가 생긴하면 `firstLineOfFile`의 `br.readLine()`에서 예외가 발생하고, `br.close()`에서도 예외가 발생할 것이다.
- 이런 상황이라면 두 번째로 발생한 예외가 첫 번째로 발생한 예외를 집어삼켜 버리고, `stack trace`에서 첫 번째 예외에 대한 정보는 남지 않게 되어 문제를 디버깅하기 어렵게 한다.

### `try-with-resource`
```java
public String firstLineOfFile(String path) throws IOException {
    try (BufferedReader br = new BufferedReader(new FileReader(path))) {
        return br.readLine();
    } finally {
        br.close();
    }
}

public void copy(String src, String dst) throws IOException {
    try (InputStream in = new FileInputStream(src);
        OutputStream out = new FileOutputStream(dst)) {
        byte[] buf = new byte[16];
        int n;
        while ((n = in.read(buf)) >= 0)
            out.write(buf, 0, n);
    }
}
```
- `try-with-resource`는 `try-finally`에 비해 코드도 간결하고 가독성도 좋으며, 문제를 분석하기도 훨씬 좋다.
- 앞서 `try-finally`에서 다루었던 문제와 마찬가지로 `br.readLine()`과 `br.close()`에서 예외가 발생하면, `br.close()`에서 발생한 예외는 숨겨지고 `br.readLine()`에서 발생한 예외가 기록된다.
  - `try-finally`처럼 예외가 덮히는 것이 아니라 두 번째로 발생한 예외를 첫 번째로 발생한 예외 뒤에 쌓아두는(suppressed) 것이다.
  - 그리고 `Throwable`의 `getSuppressed` 메서드를 이용해 프로그램 코드에서 가져올 수 있다.
  - 이렇게 발생한 예외들이 쌓이기 때문에 문제를 진단하기에 더 좋다.
- `catch`블록은 `try-finally`와 동일하게 사용할 수 있다.

## 정리
- 반납해야 하는 자원을 다룰 때는 `try-finally`말고, `try-with-resource`를 사용하자.
- 코드는 더 짧고 분명해지고, 만들어지는 예외 정보도 훨씬 유용하다.