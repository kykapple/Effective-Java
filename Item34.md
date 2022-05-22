# Item34. int 상수 대신 열거 타입을 사용하라
- 열거 타입은 일정 개수의 상수 값을 정의한 다음, 그 외의 값은 허용하지 않는 타입이다.
- 사계절, 태양계의 행성, 카드 게임의 카드 종류 등이 좋은 예이다.

<br>

## 열거 타입 지원 이전의 열거 패턴
- 열거 타입을 지원하기 전에는 다음과 같이 정수 열거 패턴을 사용하곤 했다.
```java
public static final int APPLE_FUJI = 0;
public static final int APPLE_PIPPIN = 1;
public static final int APPLE_GRANNY_SMITH = 2;

public static final int ORANGE_NAVEL = 0;
public static final int ORANGE_TEMPLE = 0;
public static final int ORANGE_BLOOD = 0;
```
- 이러한 정수 열거 패턴 기법은 타입 안전을 보장할 방법이 없고, 표현력도 좋지 않다.
  - 오렌지를 건네야 할 메서드에 사과를 보내고 동등 연산자로 비교하더라도 컴파일러는 아무런 경고 메시지를 출력하지 않는다!
  - 정수 상수값을 출력해도 단지 숫자이기 때문에 딱히 도움이 되지 않는다.

**다행히 이러한 열거 패턴의 단점을 말끔히 씻어주는 동시에 여러 장점을 안겨주는 `열거 타입(enum type)`이 등장했다.**

<br>

## 열거 타입
- 자바의 열거 타입은 완전한 형태의 클래스라서 단순한 정수값일 뿐인 다른 언어의 열거 타입보다 훨씬 강력하다.
```java
public enum Apple { FUJI, PIPPIN, GRANNY_SMITH }
public enum Orange { NAVEL, TEMPLE, BLOOD }
```
- 열거 타입의 아이디어는 단순하다. **열거 타입 자체는 클래스이며, 상수 하나당 자신의 인스턴스를 하나씩 만들어 `public static final` 필드로 공개한다.**
- 열거 타입은 밖에서 접근할 수 있는 생성자를 제공하지 않으므로 사실상 `final`이다. 따라서 클라이언트가 인스턴스를 직접 생성하거나 확장할 수 없으니 열거 타입 선언으로 만들어진 인스턴스들은 딱 하나씩만 존재함이 보장된다.
> 싱글턴은 원소가 하나뿐인 열거 타입이라 할 수 있고, 거꾸로 열거 타입은 싱글턴을 일반화한 형태라고 볼 수 있다.

<br>

### 열거 타입은 컴파일타임 타입 안전성을 제공한다.
- 열거 타입을 매개변수로 받는 메서드를 선언했다면, 매개변수에는 해당 열거 타입 중 하나임이 보장된다.
  - 다른 타입의 값을 넘길 경우 컴파일 오류가 발생한다.
```java
public static void main(String[] args) {
    foo(Orange.NAVEL);   // 컴파일 오류
}

static void foo(Apple apple) {
    // ...
}
```
 <br>
 
### 데이터와 메서드를 갖는 열거 타입 예시
```java
public enum Planet {
    MERCURY(3.302e+23, 2.439e6),
    VENUS(4.869e+24, 6.052e6),
    EARTH(5.975e+24, 6.378e6);
  
    private final double mass;
    private final double radius;
    private final double surfaceGravity;
  
    private static final double G = 6.67300E-11;
  
    Planet(double mass, double radius) {
        this.mass = mass;
        this.radius = radius;
        this.surfaceGravity = G * mass / (radius * radius);
    }
  
    public double surfaceWeight(double mass) {
         return mass * surfaceGravity;
    }
}

```
- 열거 타입 상수 각각을 특정 데이터와 연결지으려면 생성자에서 데이터를 받아 인스턴스 필드에 저장하면 된다.
- 열거 타입을 사용하면 열거 타입 상수 중 하나가 제거될 경우, 다시 컴파일하지 않더라도 런타임에 예외가 발생하게 될 것이다.
  - 정수 열거 패턴에서는 다시 컴파일하지 않는 한 상수가 제거된 것을 발견하지 못하므로, 기대할 수 없는 시나리오이다.

<br>

### 상수마다 동작이 달라지는 열거 타입
- 사칙연산과 같이 상수마다 동작이 달라져야 하는 상황도 있을 것이다.
```java
public enum Operation {
    PLUS, MINUS, TIMES, DIVIDE;
  
    public double apply(double x, double y) {
        switch(this) {
          case PLUS:   return x + y;
          case MINUS:  return x - y;
          case TIMES:  return x * y;
          case DIVIDE: return x / y;
        }
        throw new AssertionError("알 수 없는 연산: " + this);
    }
}
```
- 위 예제에서는 `switch`문을 이용해 상수의 값에 따라 분기하는 방법을 사용하였다.
- 동작은 하지만, 깨지기 쉬운 코드이다.
- 새로운 상수가 추가되면 해당 `case`문도 추가해주어야 하고, 깜빡할 경우 새로 추가한 연산을 수행하려 할 때 `AssertionError`가 발생하게 된다.
- 다행히 열거 타입은 상수별로 다르게 동작하는 코드를 구현하는 더 나은 수단을 제공한다!
- **열거 타입에 추상 메서드를 선언하고, 각 상수별 클래스 몸체에서 자신에 맞게 재정의하는 방법이다. 이를 상수별 메서드 구현이라고 한다.**
```java
public enum Operation {
    PLUS {
        public double apply(double x, double y) { return x + y; } 
    },
    MINUS {
        public double apply(double x, double y) { return x - y; } 
    },
    TIMES {
        public double apply(double x, double y) { return x * y; } 
    },
    DIVIDE {
        public double apply(double x, double y) { return x / y; } 
    };
  
    public abstract double apply(double x, double y);
}
```
- 보다시피 `apply` 메서드가 추상 메서드이므로, 재정의하는 것을 강제할 수 있게 된다.
  - 만약 재정의하지 않으면 컴파일 오류가 발생한다.

<br>

### 전략 열거 타입
- 상수별 메서드 구현에는 열거 타입 상수끼리 코드를 공유하기 어렵다는 단점이 있다.
```java
enum PayrollDay {
    MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY, SATURDAY, SUNDAY;
  
    private static final int MINS_PER_SHIFT = 8 * 60;
    
    int pay(int minutesWorked, int payRate) {
        int basePay = minutesWorked * payRate;
      
        int overtimePay;
        switch(this) {
          case SATURDAY: case SUNDAY:
            overtimePay = basePay / 2;
            break;
          default:
            overtimePay = minutesWorked <= MINS_PER_SHIFT ? 0 : (minutesWorked - MINS_PER_SHIFT) * payRate / 2;
        }
      
        return basePay + overtimePay;
    }
    
}
```
- 위 예제는 요일별 일당을 계산해주는 메서드를 가진 열거 타입이다.
- 분명 간결하지만, 휴가와 같은 새로운 값을 열거 타입에 추가할 경우, 그 값을 처리하는 `case` 문을 넣어줘야 한다. 
  - 만약 실수로 넣지 않으면 잘못된 방향으로 동작할 것이다.
- 이를 해결하기 위해 상수별 메서드 구현을 도입할 수 있는데, 모든 상수에 잔업 수당을 계산하는 코드를 넣게 되면 중복이 발생한다.
- 또한 상수별 메서드 구현이 아니라 계산 코드 자체를 평일용과 주말용으로 나눠 작성할 수도 있겠지만 코드가 장황해져 가독성이 떨어진다.
- **이를 해결할 수 있는 방법은 새로운 상수를 추가했을 때 잔업수당 `전략`을 선택하도록 하는 것이다.**
```java
enum PayrollDay {
    MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY, SATURDAY, SUNDAY;
  
    private final PayType payType;
  
    PayrollDay(PayType payType) { this.payType = payType; }
  
    int pay(int minutesWorked, int payRate) {
        return payType.pay(minutesWorked, payRate); 
    }
  
    enum PayType {
        WEEKDAY {
            int overtimePay(int minutesWorked, int payRate) {
               return minutesWorked <= MINS_PER_SHIFT ? 0 : (minutesWorked - MINS_PER_SHIFT) * payRate / 2;
            }
        },
        WEEKEND {
            int overtimePay(int minutesWorked, int payRate) {
                return minutesWorked * payRate / 2;
            }
        };
      
        abstract int overtimePay(int mins, int payRate);
        private static final int MINS_PER_SHIFT = 8 * 60;
      
        int pay(int minsWorked, int payRate) {
            int basePay = minutesWorked * payRate;
            return basePay + overtimePay(minsWorked, payRate);
        }
    }
}
```
- 잔업수당 계산을 `private` 중첩 열거 타입으로 옮기고, `PayrollDay` 열거 타입의 생성자에서 이 중 적당한 것을 선택한다.
- 그러면 `PayrollDay` 열거 타입은 잔업수당 계산을 그 전략 열거 타입에 위임하여, `switch`문이나 상수별 메서드 구현이 필요 없게 된다.
- 이 패턴은 `switch`문보다 복잡하지만 더 안전하고 유연하다.

<br>

## 그래서 열거 타입을 언제 쓰란건가?
- **필요한 원소를 컴파일타임에 다 알 수 있는 상수 집합이라면 항상 열거 타입을 사용하자.**

<br>

## 정리
- 열거 타입은 정수 상수보다 읽기 쉽고, 안전하고, 강력하다.
- 열거 타입을 사용하면서 각 상수마다 다르게 동작해야 할 때는 상수별 메서드 구현을 사용하자.
- 만약 열거 타입 상수 일부가 같은 동작을 공유해야 한다면 전략 열거 타입 패턴을 사용하자.
