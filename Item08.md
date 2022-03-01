# Item08. `finalizer`와 `cleaner` 사용을 피하라
- 자바는 `finalizer`와 `cleaner` 두 가지 객체 소멸자를 제공한다.
  - 그 중 `finalzier`는 예측 불가능하고, 대부분 불필요하다. 오동작, 낮은 성능, 이식성 문제의 원이 되기도 하기 때문에 기본적으로 쓰지 말아야 한다.
  - 따라서 자바 9에서는 `deprecated`되었고, 대안으로 `cleaner`가 등장했다.
  - `cleaner`는 `finalizer` 보다는 덜 위험하지만 여전히 예측할 수 없고, 느리고, 일반적으로 불필요하다.

### 단점 1
- `finalizer`와 `cleaner`는 즉시 수행된다는 보장이 없다,
- 객체에 접근을 할 수 없게 된 후 `finalizer`와 `cleaner`가 언제 수행될지 모른다는 것이다.
- **따라서 제때 실행되어야 하는 작업을 절대 `finalizer`와 `cleaner`에서 하면 안된다.**
- 예를 들어, `파일 닫기`를 `finalizer`나 `cleaner`에 맡기면 해당 작업이 언제 실행될지 모르고, `파일 닫기`를 하지 않는다면 더 이상 새로운 파일을 열지 못해 프로그램이 실패할 수도 있다.

### 단점 2
- `finalizer`와 `cleaner`는 아예 수행되지 않을 수도 있다.
- 접근할 수 없는 일부 객체에 딸린 종료 작업을 전혀 수행하지 못한 채 프로그램이 중단될 수도 있다는 얘기이다.
- 따라서 **상태를 영구적으로 수정하는 작업은 절대 `finalizer`나 `cleaner`에 맡기면 안된다.**
- 예를 들어 DB 같은 공유 자원의 락을 해제하는 작업을 `finalizer`나 `cleaner`로 한다면 분산 시스템 전체가 서서히 멈출 것이다.
- `System.gc` 나 `System.runFinalization`도 `finalizer`나 `cleaner`가 실행될 가능성을 높여줄 뿐, 보장해주지 않으므로 현혹되지 말자.

### 단점 3
- `finalizer`와 `cleaner`는 심각한 성능 문제도 동반한다.
- `AutoCloseable` 객체를 생성하고 `try-with-resource`로 자원을 반납하는 것에 비해 `finalizer`나 `cleaner`를 통해 자원을 반납하는 것은 매우 느리다.

### 단점 4
- `finalizer` 공격에 노출되어 심각한 보안 문제를 일으킬 수 있다.
- 생성자나 직렬화 과정에서 예외가 발생하면, 이 생성되다만 객체의 `finalizer`가 수행될 수 있게 된다. 그리고 `finalizer`는 해당 객체의 참조를 정적 필드에 기록하여 GC가 회수하지 못하게 막을 수 있다. 이후 이 인스턴스를 이용해 허용되지 않은 작업을 수행할 수도 있다.
  - 생성자에서 예외가 발생했기 때문에 존재하질 않았어야 하는 인스턴스인데, `finalizer` 때문에 살아 남은 것이다.
- `final` 클래스들은 상속이 안되기 때문에 이 공격에서 안전하지만 `final`이 아닌 클래스들은 `finalize` 메서를 `final`로 선언해야 한다.

### 그렇다면 자원 반납은 어떻게 해야하는가?
- **그저 자원 반납이 필요한 클래스에 `AutoCloseable` 인터페이스를 구현해주고, 클라이언트에서 인스턴스를 다 사용하고 나면 `try-with-resource`나 `close` 메서드를 호출하여 반납하는 것이 좋다.**

### `finalizer`와 `cleaner`는 어디에 쓰는가?
- 자원의 소유자가 `close` 메서드를 호출하지 않는 것에 대비한 안전망 역할을 할 수 있다.
  - finalizer와 cleaner가 즉시 호출되리라는 보장은 없지만, 클라이언트가 하지 않은 자원 회수를 늦게라도 해줄 수 있다.
- 실제 `FileInputStream`, `FileOutputStream`, `ThreadPoolExecutor`에는 안전망으로 동작하는 `finalizer`가 존재한다. 
