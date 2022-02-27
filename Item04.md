# Item04. 인스턴스화를 막으려거든 private 생성자를 사용하라
- 단순히 정적 메서드와 정적 필드만을 담은 클래스를 만들고 싶을 때가 있을 것이다.
  - `java.lang.Math`와 `java.util.Arrays`, `java.util.Collections`가 그 예이다.
- 정적 멤버만 담은 유틸리티 클래스들은 인스턴스로 만들어 사용하려고 설계한 것이 아니다.
- 따라서 이러한 클래스들의 인스턴스화를 막기 위해 `private` 생성자를 추가해두어야 한다.
  - 만약 `private` 생성자를 추가하지 않으면 컴파일러가 자동으로 `public` 기본 생성자를 만들어주기 때문에 `private` 생성자를 선언해주어야 한다. 
```java
public class UtilityClass {
    
    // 기본 생성자가 만들어지는 것을 막는다. -> 인스턴스화 방지용
    private UtilityClass() {
        throw new AssertionError();
    }
    ...
}
```
- 위와 같이 `private` 생성자를 추가하여 클래스의 인스턴스화를 막을 수 있고, 실수로라도 해당 클래스 안에서 생성자를 호출하지 않도록 `private` 생성자 내부에 `AssertionError()`를 던질 수도 있다.
- 또한 이러한 방식은 상속을 불가능하게 하는 효과도 있다. 하위 클래스의 생성자는 명시적이든 묵시적이든 상위 클래스의 생성자를 호출하게 되는데, 이를 `private`으로 선언하면 하위 클래스가 상위 클래스의 생성자에 접근할 수 없게 된다.
