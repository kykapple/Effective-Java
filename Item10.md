# Item10. `equals`는 일반 규약을 지켜 재정의하라
- `equals` 메서드는 재정의를 염두에 두고 설계된 것이라 재정의 시 지켜야 하는 일반 규약이 명확히 정의되어 있다.
- 따라서 `equals`를 재정하는 클래스들은 일반 규약에 맞게 재정의해야 하고, 잘못 구현할 시 `equals`를 기반으로 하는 클래스(`ArrayList`, `LinkedList`, `HashMap` 등)를 오동작하게 만들 수 있다.
<br>

## `equals`를 재정의하지 않는 것인 최선인 경우
- **각 인스턴스가 본질적으로 고유한 경우**
  - 값을 표현하는 것이 아닌 동작하는 개체를 표현하는 클래스가 여기에 해당한다. (Ex. `Thread`)
  - 인스턴스가 둘 이상 만들어지지 않음을 보장하는 인스턴스 통제 클래스의 경우도 여기에 해당한다. (Ex. `Boolean.ValueOf(boolean)`);
- **인스턴스의 `논치적 동치성(logical equality)`을 검사할 일이 없는 경우**
  - 쉽게 말해 `equals` 메서드로 인스턴스 간의 동등성을 비교할 일이 없는 경우
- **상위 클래스에서 재정의 `equals`가 하위 클래스에도 딱 들어맞는 경우**
  - `List` 구현체들은 AbstractList가 구현한 `equals`를 상속받아 사용한다.
- **클래스가 `private`이거나 `package-private(default)`이고, `equals` 메서드를 호출할 일이 없는 경우**
  - 실수로라도 `equals`가 호출되는 것을 막고 싶다면 다음과 같이 구현해두는 것도 방법이다.
```java
@Override
public boolean equals(Object o) {
  return new AssertionError();
}
```
<br>

## `equals`를 재정의해야 하는 경우
- 객체 식별성(`object identity`)가 아니라 논리적 동치성을 확인해야 하는데, 상위 클래스의 `equals`가 논리적 동치성을 비교하도록 재정의되지 않았을 때이다.
  - 즉, 상위 클래스의 `equals` 메서드가 동등성 비교를 위해 동일성(인스턴스의 참조 값) 비교를 하는 경우에 동등성 비교(값이 같은지)를 하도록 재정의하여야 한다.
<br>

## `equals` 메서드를 재정의할 때 따라야 하는 일반 규약
`equals` 메서드는 동치 관계를 구현하며, 다음을 만족한다.
- **반사성(reflexivity)** 
  - `null`이 아닌 모든 참조 값 x에 대해 `x.equals(x)`는 `true`이다.
  - 단순히 말하면 객체는 자기 자신과 같아야 한다는 뜻이다.
- **대칭성(symmetry)** 
  - `null`이 아닌 모든 참조 값 `x`, `y`에 대해 `x.equals(y)`가 `true`이면 `y.equals(x)`도 `true`이다.
  - 서로에 대한 동치 여부에 똑같이 답해야 한다는 뜻이다.
- **추이성(transitivity)**
  - `null`이 아닌 모든 참조 값 `x`, `y`, `z`에 대해 `x.equals(y)`가 `true`이고, `y.equals(z)`도 `true`이면 `x.equals(z)`도 `true`이다.
  - 구체 클래스를 확장해 새로운 값을 추가하면서 `equals` 규약을 만족시킬 방법은 존재하지 않는다.
```java
class Point {
    private final int x;
    private final int y;

    public Point(int x, int y) {
        this.x = x;
        this.y = y;
    }

    @Override
    public boolean equals(Object o) {
        if (!(o instanceof Point)) {
            return false;
        }
        Point p = (Point) o;
        return p.x == x && p.y == y;
    }
}

class ColorPoint extends Point {
    private final String color;

    public ColorPoint(int x, int y, String color) {
        super(x, y);
        this.color = color;
    }
    
    // 1. 대칭성 위배
    @Override
    public boolean equals(Object o) {
        if (!(o instanceof ColorPoint)) {
            return false;
        }
        return super.equals(o) && color.equals(((ColorPoint) o).color);
    }
}

public class Main {
  public static void main(String[] args) {
    Point p = new Point(1, 2);
    ColorPoint cp = new ColorPoint(1, 2, "red");
    System.out.println(p.equals(cp)); // true
    System.out.println(cp.equals(p)); // false
  }
}
```
- 위 코드에서 `ColorPoint`가 `equals` 메서드를 재정의하였는데, 대칭성에 위배된다. 이를 개선하기 위해 아래와 같이 `equals` 메서드를 변경할 수 있을 것이다.
```java
@Override
public boolean equals(Object o) {
    if (!(o instanceof Point)) {
        return false;
    }
    
    // o가 일반적인 Point면 색상을 무시하고 비교한다.
    if (!(o instanceof ColorPoint)) {
        return o.equals(this);
    }
    
    // o가 ColorPoint이면 색상까지 비교한다.
    return super.equals(o) && color.equals(((ColorPoint) o).color);
}
```
- 하지만 이는 추이성에 위배되는 코드이다. 다음 결과를 보도록 하자.
```java
public static void main(String[] args) {
  ColorPoint p1 = new ColorPoint(1, 2, "red");
  Point p2 = new Point(1, 2);
  ColorPoint p3 = new ColorPoint(1, 2, "blue");
  System.out.println(p1.equals(p2)); // true
  System.out.println(p2.equals(p3)); // true
  System.out.println(p1.equals(p3)); // false
}
```
- 결국 구체 클래스를 확장해 새로운 값을 추가하면서 `equals` 규약을 만족시킬 방법은 존재하지 않는다.
- 다음과 같이 `equals` 안의 `instanceof` 검사를 `getClass` 검사로 바꾸면 규약을 지킬 수 있을 것 같지만, 이는 리스코프 치환 원칙에 위배된다.
```java
@Override
public boolean equals(Object o) {
    if (o == null || o.getClass() != getClass()) {
        return false;
    }
    Point p = (Point) o;
    return p.x == x && p.y == y;
}
```
- 위와 같이 `Point` 클래스에서 `equals`를 구현하게 되면 `Point` 클래스를 상속한 하위 클래스가 `equals` 메서드에서 `Point` 로써 활용될 수 없다. `Point`의 하위 클래스는 정의상 여전히 `Point`이므로 어디서든 `Point`로써 활용될 수 있어야 하는데, 그렇지 못하므로 리스코프 치환 원칙에 위배된다.
- 대신 클래스 확장에 상속이 아닌 컴포지션을 사용하면 우회적으로 규약을 만족시킬 수는 있다.
```java
// 컴포지션 활용 
class ColorPoint {
    private final Point point;
    private final String color;

    public ColorPoint(Point point, String color) {
        this.point = point;
        this.color = color;
    }
    
    public Point asPoint() {
        return point;
    }

    @Override
    public boolean equals(Object o) {
        if (!(o instanceof ColorPoint)) {
            return false;
        }
        ColorPoint cp = (ColorPoint) o;
        return cp.point.equals(point) && cp.color.equals(color);
    }
}
```
- **일관성(consistency)** 
  - `null`이 아닌 모든 참조 값 `x`, `y`에 대해 `x.equals(y)`를 반복해서 호출하면 항상 `true`를 반환하거나 항상 `false`를 반환한다.
  - 클래스가 불변이든 가변이든 `equals`에 판단에 신뢰할 수 없는 자원이 끼어들게 해서는 안 된다.
- **null-아님** 
  - `null`이 아닌 모든 참조 값 `x`에 대해 `x.equals(null)`은 `false`이다.
  - 모든 객체가 `null`과 같이 않아야 한다는 뜻이다.
  - 이를 확인하기 위해서 다음과 같이 입력이 입력 매개변수가 올바른 타입인지 검사해줌으로써 묵시적으로 `null` 검사를 진행해주면 된다.
 ```java
@Override
public boolean equals(Object o) {
  if (!(o instanceof MyType)) {
    return false;
  }
  MyType myType = (MyType) o;
  ...
}
```
> 컬렉션 클래스들을 포함해 수많은 클래스들은 전달받은 객체가 `equals` 규약을 지킨다고 가정하고 동작하기 때문에 위 규약들을 어길 시 프로그램이 예상과는 다르게 동작하게 된다.
<br>

## `equals` 메서드 구현 방법
1. `==` 연산자를 사용해 입력이 자기 자신의 참조인지 확인한다.
2. `instanceof` 연산자로 입력이 올바른 타입인지 확인한다. 그렇지 않으면 `false`를 반환한다.
3. 입력을 올바른 타입으로 형변환한다.
4. 입력 객체와 자기 자신의 대응되는 핵심 필드들이 모두 일치하는지 하나씩 검사한다.
    - 모든 필드가 일치하면 `true`를, 하나라도 다르면 `false`를 반환한다.
    - Primitive type은 `==`연산자로 비교하고, Reference type은 `equals` 메서드로 비교한다.
 
**`equals`를 다 구현했다면 세 가지만 자문해보자. 대칭적인가? 추이성이 있는가? 일관적인가?**
<br>

## 전형적인 `equals` 메서드의 예
```java
@Override
public boolean equals(Object o) {
  if (o == this) {
    return true;
  }
  
  if (!(o instanceof PhoneNumber)) {
    return false;
  }
  PhoneNumber pn = new (PhoneNumber) o;
  return pn.lineNum == lineNum && pn.prefix == prefix;
}
```
<br>

## `equals` 구현 시 주의사항
- `equals`를 재정의할 땐 `hashCode`도 반드시 재정의하자 (Item12)
- 너무 복잡하게 해결하려 들지 말자. 필드들의 동치성만 검사해도 `equals` 규약을 어렵지 않게 지킬 수 있다.
- `Object`외의 타입을 매개변수로 받는 `equals` 메서드는 선언하지 말자.
```java
// 잘못된 예 - 입력 타입은 반드시 Object여야 한다.
public boolean equals(MyClass o) {
  ...
}
```
<br>

## 정리
- **꼭 필요한 경우가 아니라면 `equals`를 재정의하지 말자.**
- **재정의해야 한다면 그 클랙스의 핵심 필드 모두를 빠짐없이, 다섯 가지 규약을 확실히 지켜가면 비교해야 한다.**
