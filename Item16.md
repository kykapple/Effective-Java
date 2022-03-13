# Item 16. `public` 클래스에서는 `public` 필드가 아닌 접근자 메서드를 사용하라

## 부적절한 구현
```java
public class Point {
    public double x;
    public double y;
}
```
위와 같은 클래스는 데이터 필드에 직접 접근할 수 있으니 캡슐화의 이점을 제공하지 못한다.
- API를 수정하지 않고는 내부 표현을 바꿀 수 없다.
  - 외부에서 필드를 직접적으로 사용할 수 있기 때문에 내부 표현을 자유롭게 바꿀 수 없다.
- 불변식을 보장할 수 없다.
  - 외부에서 쉽게 접근하여 값을 변경할 수 있으므로 불변식을 보장할 수 없고, `Thread-safe`하지도 않다.
- 외부에서 필드에 접근할 때 부수 작업을 수행할 수도 없다.

<br>

## 접근자와 변경자를 이용한 구현
```java
public class Point {
    private double x;
    private double y;

    public Point(double x, double y) {
      this.x = x;
      this.y = y;
    }

    public double getX() { return x; }
    public double getY() { return y; }

    public void setX(double x) { this.x = x; }
    public void setY(double y) { this.y = y; }
}
```
- 앞서 `public`으로 필드를 선언한 구현의 문제점을 해결하기 위해 필드들을 모두 `private`으로 선언하고, `public` 접근자(`getter`)와 변경자(`setter`)를 제공하는 방법이 있다.
- 외부에서 접근할 수 있는 클래스라면 접근자를 제공함으로써 클래스 내부 표현 방식을 언제든지 바꿀 수 있는 유연성을 얻을 수 있다.

<br>

## 정리
- `public` 클래스는 절대 가변 필드를 직접 노출해서는 안 된다.
- 불변 필드라면 노출해도 덜 위험하지만 완전히 안심할 수는 없다.
  - API를 변경하지 않고는 표현 방식을 바꿀 수 없고, 필드를 읽을 때 부수 작업을 수행할 수 없다는 단점이 있다.

