# Item03. private 생성자나 열거 타입으로 싱글턴임을 보증하라
- 싱글턴이란 인스턴스를 오직 하나만 생성할 수 있는 클래스를 말한다.
  - 싱글턴의 전형적인 예로는 함수와 같은 무상태(stateless) 객체나 설계상 유일해야 하는 시스템 컴포넌트가 있다.
- 클래스를 싱글턴으로 만들면 이를 사용하는 클라이언트 코드를 테스트하기 어려워진다.
  - 인터페이스를 구현해서 만든 싱글턴이 아니라면 `mock`으로 교체하는 것이 어렵기 때문이다.
- 싱글턴을 만드는 방법은 2가지가 있는데, 두 방법식 모두 생성자는 `private`으로 감춰두고, `public static` 멤버를 통해 유일한 인스턴스를 제공하는 방식이다.
<br>

### 방법1: `public static final` 필드
```java
public class Elvis {
  public static final Elvis INSTANCE = new Elvis();

  private Elvis() {
  }
}
```
- 첫 번째 방법은 `public static final` 필드로 싱글턴 객체를 제공하는 방식이다.
- `public static final` 필드의 객체는 `static` 이므로 클래스가 로딩되는 시점에 한 번만 만들어지고, 이후부터는 해당 인스턴스를 재사용하게 된다.
- 이 방법은 리플렉션 API를 사용해서 싱글턴이 깨질 수도 있지만 `private` 생성자를 수정하여 두 번째 객체가 생성되려 할 때 예외를 던지게 하면 된다.
  - `private` 생성자에서 `count`나 `flag`를 이용하여 예외를 던지게 할 수 있다.
- 이 방법의 장점은 해당 클래스가 싱글턴임이 API에 명확히 드러나고, 간결하다는 점이다.
 <br>
 
 ### 방법2: 정적 팩터리 메서드
 ```java
 public class Elvis {
  private static final Elvis INSTANCE = new Elvis();

  private Elvis() {
  }

  public static Elvis getInstance() {
      return INSTANCE;
  }
}
 ```
- 두 번째 방법은 정적 팩터리 메서드를 `public static` 멤버로 두고, 해당 정적 팩터리 메서드를 통해 싱글턴 객체를 제공하는 방식이다.
- `Elvis.getInstance()` 를 통해 항상 같은 인스턴스를 반환 받을 수 있지만, 리플렉션을 통한 예외는 첫 번째 방법과 동일하게 적용된다.
- 이 방법의 장점은 API를 변경하지 않고도 싱글턴이 아니게 변경할 수 있다는 점이다.
  - 위 예제 코드에서 `getInstance()` API에서 `return new Elvis()`를 해버리면 클라이언트 코드를 변경하지 않고도 싱글턴이 아니게 변경할 수 있다.
- 또한 정적 팩터리의 메서드 참조를 `supplier` 로 사용할 수 있다.
```java
Supplier<Elvis> supplier = Elvis::getInstance;
```
<br> 

### 직렬화
- 앞서 살펴본 2가지 방법 모두 직렬화에 사용한다면 역직렬화 할 때 같은 타입의 인스턴스가 여러 개 생길 수 있다.
- 이 문제를 해결하려면 모든 인스턴스 필드에 `transient`를 선언하고, `readResolve` 메서드를 제공해야 한다.
  - `transient`는 직렬화 하지 않겠다는 의미이고, `readResolve` 메서드를 구현하면 역직렬화 할 때 항상 호출된다.
```java
public class Elvis {
  private static final transient Elvis INSTANCE = new Elvis();

  private Elvis() {
  }

  public static Elvis getInstance() {
      return INSTANCE;
  }

  private Object readResolve() {
      return INSTANCE;
  }
}
```
<br>

### 방법3: Enum 타입
```java
public enum Elvis {
  INSTANCE;

  public void doSomething() {
      ...
  }
}
```
- 세 번째 방법은 원소가 하나인 열거 타입을 선언하는 것이다.
- 코드가 조금 부자연스러워 보일 수는 있으나 싱글턴을 만드는 가장 좋은 방법이다.
  - 직렬화나 리플렉션 공격에서도 제 2의 인스턴스가 생기는 일을 완벽히 막아준다.
- 단, 만들려는 싱글턴이 Enum 외의 클래스를 상속해야 한다면 이 방법은 사용할 수 없다.
