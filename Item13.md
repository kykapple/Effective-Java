# Item13. `clone` 재정의는 주의해서 진행하라

## `Cloneable` 인터페이스
- `Cloneable`은 복제해도 되는 클래스임을 명시하는 용도의 믹스인 인터페이스이다.
- 하지만 문제는 `clone` 메서드가 선언된 곳이 `Cloneable`이 아닌 `Object`이고, 그마저도 `protected`라는 데 있다.
- 그래서 `Cloneable`을 구현하는 것만으로는 외부 객체에서 `clone` 메서드를 호출할 수 없다.
    - 실제 사용 시에는 `public` 접근 제어자로 오버라이딩해서 사용해야 한다.
> 믹스인이란 클래스가 구현할 수 있는 타입으로, 원래의 주된 타입 외에도 특정 선택적 행위를 제공한다고 선언하는 효과를 준다.

<br>

## `Cloneable` 인터페이스의 역할
```java
public interface Cloneable {
}
```
- 이 인터페이스는 `Object`의 `protected` 메서드인 `clone`의 동작 방식을 결정한다.
- `Cloneable`을 구현한 클래스의 인스턴스에서 `clone`을 호출하면 그 객체의 필드들을 하나하나 복사한 객체를 반환하며, 그렇지 않은 클래스의 인스턴스에서 호출하면 `CloneNotSupportedException`을 던진다.

<br>

## `clone` 메서드의 일반 규약
- 이 객체의 복사본을 생성해 반환한다.
    - `x.clone() != x`
- 관례상, 이 메서드가 반환하는 객체는 `super.clone`을 호출해서 얻어와야 한다. 이 클래스와 (Object를 제외한) 모든 상위 클래스가 이 관례를 따른다면 다음 식은 참이다.
    - `x.clone().getClass() == x.getClass()`
- 관례상, 반환된 객체와 원본 객체는 독립적이어야 한다. 이를 만족하려면 `super.clone`으로 얻은 객체의 필드 중 하나 이상을 반환 전에 수정해야 할 수도 있다.
    - 만약 필드로 객체를 가지고 있다면 깊은 복사를 위해 새로 생성해서 할당해주어야 한다.
  
<br>

## `clone` 메서드 재정의 (기본 타입과 불변 객체 필드)
- 만약 클래스에 정의된 모든 필드가 기본 타입이거나 불변 객체를 참조한다면 다음과 같이 구현해도 무방하다.
```java
public Class PhoneNumber implements Cloneable {
  ...

  @Override
  public PhoneNumber clone() {                  // 공변 반환 타이핑
      try {
          return (PhoneNumber) super.clone();
      } catch (CloneNotSupportedException e) {
          throw new AssertionError(); // 일어날 수 없는 일이다.
      }
  }

  ...
}
```
- 위 코드는 `super.clone()`을 통해 완벽한 복제본을 반환받을 수 있는 `clone`메서드이다.
- 공변 반환 타이핑을 사용하여 형변환을 신경쓰지 않아도 되게 해주었고, `try-catch` 블록은 `Object`의 `clone` 메서드가 `checked exception`인 `CloneNotSupportedException`을 던지도록 선언되어 있기 때문이다.
- 이렇게 클래스에 기본 타입이나 불변 객체만 존재한다면 괜찮은데, 가변 객체를 참조하는 순간 부가적인 작업을 더 해주어야 한다.

<br>

## `clone` 메서드 재정의 (가변 객체 필드)
- 다음 `Circle` 클래스를 예로 들어보자.
```java
class Circle {
  private Point point;
  private int radius;

  Circle(Point point, int radius) {
      this.point = point;
      this.radius = radius;
  }

  ...
}
```
- 이 클래스를 복제할 때 `clone` 메서드가 단순히 `super.clone`의 결과를 그대로 반환한다면 반환된 `Circle`인스턴스의 `radius` 필드는 올바른 값을 갖겠지만, `point` 필드는 원본 `Circle` 인스턴스와 똑같은 객체를 참조할 것이다. (얕은 복사)
  - 이로 인해 원본이나 복제본 중 하나를 수정하면 다른 하나도 영향을 받게 된다.
- **`clone`은 원본 객체에 아무런 해를 끼치지 않는 동시에 복제된 객체의 불변식을 보장해야 한다.** 
- 따라서 `Circle`의 `clone`메서드는 제대로 동작하려면 아래와 같이 가변 객체도 새로 복사를 해주어야 한다.
```java
@Override
public Circle clone() {
  Object obj = null;

  try {
      obj = super.clone();
  } catch(CloneNotSupportedException e) {}

  Circle c = (Circle) obj;
  c.point = this.point.clone();
  return c;
}
```
- 추가적으로 만약 복사하려는 클래스의 필드에 객체 배열이 존재하는 경우, `clone` 메서드에서 배열을 순회하며 객체 원소들을 모두 깊은 복사해주어야 한다.
<br>

## `Cloneable` 유의사항
- `clone`을 재정의한 `public` 메서드에서는 해당 메서드를 사용하기 편하도록 `throws` 절을 없애야 한다.
- `Cloneable`을 구현한 `Thread-safe` 클래스를 작성할 때는 `clone` 메서드 역시 적절히 동기화해주어야 한다.
<br>

## `Cloneable` 요약
- `Cloneable`을 구현하는 모든 클래스는 `clone`을 재정의해야 한다.
  - 이때 접근 제한자는 `public`으로, 반환 타입은 클래스 자신으로 변경한다.
- `clone` 메서드는 가장 먼저 `super.clone`을 호출한 후 필요한 필드를 전부 적절히 수정한다.
  - 객체의 내부 '깊은 구조'에 숨어있는 모든 가변 객체를 복사하고, 복제본이 가진 객체 참조 모두가 복사된 객체들을 가리키게 해야한다.
  - 만약 기본 타입 필드와 불변 객체 참조만 갖는 클래스라면 필드를 수정하지 않아도 되지만, 일련번호나 고유 ID는 기본 타입이나 불변일지라도 수정해줘야 한다.
<br>

## 꼭 `Cloneable`을 사용해야 하는가
- **복사 생성자와 복사 팩터리라는 더 나은 객체 복사 방식을 제공할 수 있다.**
- 복사 생성자란 단순히 자신과 같은 클래스의 인스턴스를 인수로 받는 생성자이다.
```java
public Yum(Yum yum) { ... };
```
- 복사 팩터리는 복사 생성자를 모방한 정적 팩터리이다.
```java
public static Yum newInstance(Yum yum) { ... };
```
- 복사 생성자와 복사 팩터리가 `Cloneable/clone` 방식보다 더 나은 이유는 다음과 같다.
  - 언어 모순적이고 위험천만한 객체 생성 매커니즘(생성자를 쓰지 않는 방식)을 사용하지 않는다.
  - 엉성하게 문서화된 규약에 기대지 않는다.
  - 정상적인 final 필드 용법과도 충돌하지 않는다.
  - 불필요한 검사 예외를 덤지지 않고, 형변환도 필요치 않다.
  - 해당 클래스가 구현한 `인터페이스` 타입의 인스턴스를 인수로 받아 클라이언트가 원본의 구현 타입에 얽매이지 않고 복제본의 타입을 직접 선택할 수 있다.
  ```java
  HashSet<Object> hashSet = new HashSet<>();
  TreeSet<Object> treeSet = new TreeSet<>(hashSet);
  ```
<br>

## 정리
- 복제 기능은 생성자와 팩터리를 이용하는 것이 좋다.
- 단, 배열만은 `clone` 메서드 방식이 가장 깔끔한, 이 규칙의 합당한 예외라 할 수 있다.
