# Item 17. 변경 가능성을 최소화하라

## 불변 클래스
**불변 클래스란 그 인스턴스의 내부 값을 수정할 수 없는 클래스이다.**

불변 클래스에 간직된 정보는 고정되어 객체가 파괴되는 순간까지 절대 달라지지 않는다.

불변 클래스는 가변 클래스보다 설계하고 구현하고 사용하기 쉬우며, 오류가 생길 여지도 적고 훨씬 안전하다.

<br>

## 불변 클래스의 규칙
- **객체의 상태를 변경하는 메서드(`setter`)를 제공하지 않는다.**
- **클래스를 확장할 수 없도록 한다.**
  - 하위 클래스에서 부주의하게 혹은 나쁜 의도로 객체의 상태를 변하게 만드는 사태를 막아준다.
- **모든 필드를 `final`로 선언한다.**
  - 시스템이 강제하는 수단을 이용해 설계자의 의도를 명확히 드러내는 방법이다.
- **모든 필드를 `private`으로 선언한다.**
  - 필드가 참조하는 가변 객체를 클라이언트에서 직접 접근해 수정하는 일을 막아준다.
- **자신 외에는 내부의 가변 컴포넌트에 접근할 수 없도록 한다.**
  - 클래스에 가변 객체를 참조하는 필드가 하나라도 있다면 클라이언트에서 그 객체의 참조를 얻을 수 없도록 해야 한다.
  - 이런 필드의 접근자 메서드는 방어적 복사를 수행하여 반환해야 한다.
  
<br>

## 불변 클래스 예시
```java
// 불변 복소수 클래스
public final class Complex {
    private final double re;
    private final double im;

    public Complex(double re, double im) {
      this.re = re;
      this.im = im;
    }

    public double realPart() { return re; }
    public double imaginaryPart() { return im; }

    public Complex plus(Complex c) {
      return new Complex(re + c.re, im + c.im);
    }
    
    ...
}
```
- **위 불변 클래스에서 사칙연산 메서드가 인스턴스 자신은 수정하지 않고, 새로운 `Complex` 인턴스를 만들어서 반환하는 모습에 주목하자.** 
> 이처럼 피연산자에 함수를 적용해 결과를 반환하지만, 피연산자 자체는 그대로인 프로그래밍 패턴을 함수형 프로그래밍이라고 한다.
> 이 방식으로 프로그래밍을 하면 코드에서 불변이 되는 영역의 비율이 높아지는 장점을 누릴 수 있다.
<br>

## 불변 클래스의 장점
- **불변 객체는 단순하다.**
  - 불변 객체는 생성된 시점의 상태를 파괴될 때까지 그대로 간직한다.
- **불변 객체는 근복적으로 `Thread-safe` 하므로 따로 동기화할 필요가 없다.**
  - 여러 스레드가 동시에 사용해도 절대 훼손되지 않는다.
  - `Thread-safe`한 클래스를 만드는 가장 쉬운 방법이기도 하다.
- **불변 객체는 안심하고 공유할 수 있다.**
  - 불변 객체에 대해서는 그 어떤 스레드로 다른 스레드에 영향을 줄 수 없다.
  - 따라서 불변 클래스라면 한번 만든 인스턴스를 최대한 재활용하기를 권한다.
  - 가장 쉬운 재활용 방법은 아래와 같이 자주 쓰이는 값들을 상수(`public static final`)로 제공하는 것이다.
```java
public static final Complex ZERO = new Complex(0, 0);
public static final Complex ONE = new Complex(1, 0);
public static final Complex I = new Complex(0, 1);
```
- **불변 객체는 자유롭게 공유할 수 있기 때문에 방어적 복사도 필요없다.**
  - 그러니 불변 클래스는 `clone` 메서드나 복사 생성자를 제공하지 않는게 좋다.
- **불변 객체는 자유롭게 공유할 수 있음은 물론, 불변 객체끼리는 내부 데이터를 공유할 수 있다.**
  - `BigInteger` 클래스는 내부에서 값의 부호와 크기를 따로 표현한다. 그리고 `negate` 메서드는 크기가 같고 부호만 반대인 새로운 `BigInteger`를 생성하는데, 이때 부호만 변경하고 크기는 원본 인스턴스와 공유해도 된다.
- **객체를 만들 때 다른 불변 객체들을 구성요소로 사용하면 이점이 많다**
  - 값이 바뀌지 않는 구성요소들로 이뤄진 객체라면, 그 구조가 아무리 복잡하더라도 불변식을 유지하기 훨씬 수월하다.
- **불변 객체는 그 자체로 실패 원자성을 제공한다**
  - 상태가 절대 변하지 않으니 잠깐이라도 불일치 상태에 빠질 가능성이 없다.
> 실패 원자성 - 메서드에서 예외가 발생한 후에도 그 객체는 여전히 (메서드 호출 전과 똑같은)유효한 상태여야 한다는 것

<br>

## 불변 클래스의 단점
- **값이 다르면 반드시 독립된 객체로 만들어야 한다.**
  - 값의 가짓수가 많다면 이들을 모두 만드는 데 큰 비용을 치러야 한다.
- 이러한 문제는 가변 동반 클래스를 통해 해결할 수 있다.
  - 예를 들어, 불변인 `String` 클래스의 가변 동반 클래스로 `StringBuilder`가 있고, `BigInteger` 클래스의 가변 동반 클래스로 `MutableBigInteger`, `BitSieve` 등이 있다.
  
<br>

## 불변 클래스의 또 다른 설계 방식
클래스가 불변임을 보장하는 가장 쉬운 방법은 `final` 클래스로 선언하는 것이다.

하지만 모든 생성자를 `private` 혹은 `package-private`으로 만들고, `public` 정적 팩터리를 제공하는 더 유연한 방법도 있다.

다음 코드는 앞서 `final` 클래스로 선언한 `Complex`를 정적 팩터리 방식으로 구현한 코드이다.
```java
// 생성자 대신 정적 팩토리를 사용한 불변 클래스
public class Complex {
    private final double re;
    private final double im;

    private Complex(double re, double im) {
      this.re = re;
      this.im = im;
    }

    public static Complex valueOf(double re, double im) {
      return new Complex(re, im);
    }

    ...
}
```
- 정적 팩터리 방식은 다수의 구현 클래스를 활용한 유연성을 제공하고, 이에 더해 다음 릴리스에서 객체 캐싱 기능을 추가해 성능을 끌어올릴 수도 있다.
  - 싱글톤이나, 앞서 살펴본 `Zero`, `One`, `I` 등을 캐싱해둘 수 있을 것 같다.

<br>

## 불변 클래스의 규칙 완화
불변 클래스의 규칙 목록에 따르면 모든 필드는 `final`이어야 하고, 어떤 메서드도 그 객체를 수정할 수 없다고 했다. 사실 이 규칙은 조금 과한 감이 있어서 다음처럼 완화할 수 있다.

**어떤 메서드도 객체의 상태 중 외부에 비치는 값을 변경할 수 없다.**

어떤 불변 클래스는 계산 비용이 큰 값을 나중에(처음 쓰일 때) 계산하여 `final`이 아닌 필드에 캐시해놓기도 한다.
  - `String` 클래스의 `hashCode`는 처음 쓰일 때 계산하여 `final`이 아닌 `hash` 필드에 캐시해놓고 사용한다.
  - 이는 그 객체가 불변이므로 몇 번을 계산해도 항상 같은 결과가 만들어짐이 보장되기 때문이다.
  
<br>

## 정리
- `getter`가 있다고 해서 무조건 `setter`를 만들지는 말자.
- **클래스는 꼭 필요한 경우가 아니라면 불변이어야 한다.**
  - 불변 클래스는 장점이 많으며, 단점이라곤 특정 상황에서의 잠재적 성능 저하 뿐이다. 
- **불변으로 만들 수 없는 클래스라면, 변경할 수 있는 부분을 최소한으로 줄이자.**
  - 꼭 변경해야 할 필드를 뺀 나머지 모두를 `final`로 선언하자.
  - [아이템15](https://github.com/kykapple/Effective-Java/blob/main/Item15.md)의 내용과 종합하면, **다른 합당한 이유가 없다면 모든 필드는 `private final`이어야 한다.**
- 확실한 이유가 없다면 생성자와 정적 팩터리 외에는 그 어떤 초기화 메서드도 `public`으로 제공해서는 안 된다.
