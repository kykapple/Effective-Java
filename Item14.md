# Item14. `Comparable`을 구현할지 고려하라

## `Comparable` 인터페이스
```java
public interface Comparable<T> {
  public int compareTo(T o);
}
```
- `Comparable`은 `compareTo`라는 추상 메서드를 가지고 있는 인터페이스이다.
- **`compareTo` 메서드는 단순 동치성 비교에 더해 순서까지 비교할 수 있으며, 제네릭하다.**
- `Comparable`을 구현했다는 것은 그 클래스의 인스턴스들에는 자연적인 순서가 있음을 뜻한다.
<br>

## `compareTo` 메서드의 일반 규약
- 이 객체와 주어진 객체의 순서를 비교한다. 
  - 이 객체가 주어진 객체보다 작으면 음의 정수를, 같으면 0을, 크면 양의 정수를 반환한다.
  - 이 객체와 비교할 수 없는 타입의 객체가 주어지면 `ClassCastException`을 던진다.
- `Comparable`을 구현한 클래스는 모든 `x`, `y`에 대해 `sgn(x.compareTo(y)) == -sgn(y.compareTo(x))`여야 한다. (대칭성)
  - 따라서 `x.compareTo(y)`는 `y.compareTo(x)`가 예외를 던질 때에 한해 예외를 던져야 한다.
- `Comparable`을 구현한 클래스는 추이성을 보장해야 한다.
  - 즉, `x.compareTo(y) > 0 && y.compareTo(z) > 0`이면 `x.compareTo(z) > 0`이다.
- `Comparable`을 구현한 클래스는 모든 `z`에 대해 `x.compareTo(y) == 0`이면 `sgn(x.compareTo(z)) == sgn(y.compareTo(z))`이다.
- 이번 권고는 필수는 아니지만 꼭 지키는 것이 좋다. 
  - `(x.compareTo(y) == 0) == (x.equals(y))`여야 한다.
  - `compareTo` 메서드로 수행한 동치성 테스트의 결과가 `equals`와 같아야 한다는 것이다.
  - 예를 들어 `Collection`, `Set` 등의 컬렉션 인터페이스들은 `equals` 메서드 규약을 따르지만 정렬할 때는 `equals` 대신 `compareTo`를 사용한다. 따라서 일관성 있는 결과를 위해서는 이번 권고를 지키도록 하자.
<br>

## `compareTo` 메서드 작성 요령
### 컴파일 타임에 타입이 정해진다.
**`Comparable`은 타입을 인수로 받는 제네릭 인터페이스이므로 `compareTo` 메서드의 인수 타입은 컴파일타임에 정해진다.**

그러므로 입력 인수의 타입을 확인하거나 형변환할 필요가 없다.

<br>

### 순서를 비교한다.
**`compareTo` 메서드는 각 필드가 동치인지를 비교하는 게 아니라 그 순서를 비교한다.**
  - 객체 참조 필드를 비교하려면 `compareTo` 메서드를 재귀적으로 호출하면 된다.
  - `Comparable`을 구현하지 않은 필드라면 비교자(`Comparator`)를 대신 사용하도록 한다.
  - 다음은 자바가 제공하는 비교자를 사용하는 `compareTo` 메서드의 예시이다.
```java
public final class CaseInsensitiveString implements Comparable<CaseInsensitiveString> {
  public int compareTo(CaseInsensitiveString cis) {
    return String.CASE_INSENSITIVE_ORDER.compare(s, cis.s);
  }
  ...
}
```
위 코드에서 `CaseInsensitiveString`이 `Compare<CaseInsensitiveString>`을 구현한 것을 주목하자.

`CaseInsensitiveString`의 참조는 `CaseInsensitiveString` 참조와만 비교할 수 있다는 뜻으로, `Comparable`을 구현할 때 일반적으로 따르는 패턴이다.
<br>

### 핵심 필드의 비교 순서 지정
클래스에 핵심 필드가 여러 개라면 어느 것을 먼저 비교하느냐가 중요해진다.

**가장 핵심적인 필드부터 비교해나가자.**

비교 결과가 0이 아니라면, 즉 순서가 결정되면 거기서 곧장 결과를 반환하면 되고, 가장 핵심이 되는 필드가 똑같다면 똑같지 않은 필드를 찾을 때까지 그 다음으로 중요한 필드를 비교해나간다.
```java
public int compareTo(PhoneNumber pn) {
  int result = Short.compare(areaCode, pn.areaCode);   // 가장 중요한 필드
  if (result == 0) {
      result = Short.compare(prefix, pn.prefix);       // 두 번째로 중요한 필드
      if (result == 0) {
          result = Short.compare(lineNum, pn.lineNum); // 세 번째로 중요한 필드
      }
  }
  return result;
}
```
<br>

### Comparator 인터페이스 활용
- 자바 8에서는 `Comparator` 인터페이스가 일련의 비교자 생성 메서드와 팀을 꾸려 메서드 연쇄 방식으로 비교자를 생성할 수 있게 되었다.
- 그리고 이 비교자들은 `Comparable` 인터페이스가 원하는 `compareTo` 메서드를 구현하는 데 활용할 수 있다.
- 이 방식은 간결함을 가져다주지만, 약간의 성능 저하가 뒤따른다.
```java
private static final Comparator<PhoneNumber> COMPARTOR = 
    comparingInt((PhoneNumber pn) -> pn.areaCode)
        .thenComparingInt(pn -> pn.prefix)
        .thenComparingInt(pn -> pn.lineNum);

public int compareTo(PhoneNumber pn) {
    return COMPARATOR.compare(this, pn);
}
```
- 숫자 타입 외에도 객체 참조용 비교자 생성 메서드인 `compare`, `thenComparing`도 준비되어 있다.
- 추가적으로 이러한 비교자는 아래와 같이 값의 차를 기준으로 구현하면 안된다.
  - 정수 오버플로나 부동소수점 계산 방식에 따른 오류를 낼 수 있기 때문!
```java
static Comparator<Object> hashCodeOrder = new Comparator<>() {
  public int compare(Object o1, Object o2) {
      return o1.hashCode() - o2.hashCode();  // 값의 차를 이용하는 방법
  }
}
```
- 다음 두 방식 중 하나를 사용하도록 하자!
```java
// 정적 compare 메서드를 활용한 비교자
static Comparator<Object> hashCodeOrder = new Comparator<>() {
    public int compare(Object o1, Object o2) {
        return Integer.compare(o1.hashCode(), o2.hashCode());
    }
};
```
```java
// 비교자 생성 메서드를 활용한 비교자
static Comparator<Object> hashCodeOrder = Comparator.comparingInt(o -> o.hashCode());
```
<br>

## 정리
- 순서를 고려해야 하는 값 클래스를 작성한다면 꼭 `Comparable` 인터페이스를 구현하여, 그 인스턴스들을 쉽게 정렬하고, 검색하고, 비교 기능을 제공하는 컬렉션과 어우지도록 하자.
- `compareTo` 메서드에서 필드의 값을 비교할 때 박싱된 기본 타입 클래스가 제공하는 정적 `compare`메서드나 `Comparator` 인터페이스가 제공하는 비교자 생성 메서드를 사용하자.
  - `Integer.compare()`, `Double.compare()` 등
