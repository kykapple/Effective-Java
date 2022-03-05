# Item11. `equals`를 재정의하려거든 `hashCode`도 재정의하라
- `equals`를 재정의한 클래스 모두에서 `hashCode`도 재정의해야 한다.
- 그렇지 않으면 `hashCode` 일반 규약을 어기게 되어 해당 클래스의 인스턴스를 `HashMap`이나 `HashSet`같은 컬렉션의 원소로 사용할 때 문제를 일으키게 된다.
<br>

## `equals`와 `hashCode`에 대한 Object 명세
- `equals` 비교에 사용되는 정보가 변경되지 않았다면, 애플리케이션이 실행되는 동안 그 객체의 `hashCode` 메서드는 몇 번을 호출해도 일관되게 항상 같은 값을 반환해야 한다. 단, 애플리케이션을 다시 실행한다면 이 값이 달라져도 상관없다.
- `equals(Object)`가 두 객체를 같다고 판단했다면, 두 객체의 `hashCode`는 똑같은 값을 반환해야 한다.
- `equals(Object)`가 두 객체를 다르게 판단했더라도, 두 객체의 `hashCode`가 서로 다른 값을 반환할 필요는 없다. 단, 다른 객체에 대해서는 다른 값을 반환해야 해시테이블의 성능이 좋아진다.

**`hashCode` 재정의를 잘못했을 때 크게 문제가 되는 조항은 두 번째이다. 즉, 논리적으로 같은 객체는 같은 해시코드를 반환해야 한다.**
```java
public static void main(String[] args) {
    Map<Point, Boolean> map = new HashMap<>();
    map.put(new Point(1, 1), true);
    System.out.println(map.get(new Point(1, 1))); // null
}
```
- 위 코드에서 사용된 2개의 `Point` 객체가 논리적으로 동일하므로 `map.get()` 을 했을 때 `true`가 반환될 것 같지만 `null`이 반환된다.
- 이는 `Point` 클래스가 `hashCode`를 재정의하지 않았기 때문에 논리적 동치인 두 객체가 서로 다른 해시코드를 반환하기 때문이다.
- 따라서 `Point` 클래스에 적절한 `hashCode` 메서드만 재정의해주면 된다.
```java
@Override
public int hashCode() {
    return Objects.hash(x, y);
}
```
<br>

## 최악의 `hashCode` 구현
```java
@Override
public int hashCode() {
  return 42;
}
```
- 이는 모든 객체에 대해서 동일한 해시코드 값을 반환하므로 모든 객체가 해시테이블의 버킷 하나에 담겨 마치 연결 리스트처럼 동작한다. 
- 그 결과 평균 수행 시간이 `O(1)`에서 `O(n)`으로 느려져서, 객체가 많아지면 도저히 쓸 수 없게 된다.
<br>

## 좋은 `hashCode` 구현
- 이상적인 해시 함수는 주어진 인스턴스들을 32비트 정수 범위에 균일하게 분배해야 한다.
- 다음은 좋은 `hashCode`를 작성하는 간단한 요령이다.
  1. `int` 변수 `result` 선언 후 값 `c`로 초기화한다.
     - 이때 `c`는 해당 객체이 첫 번째 핵심 필드(`equals` 비교에 사용되는 필드)를 단계 `2.a` 방식으로 계산한 해시 코드이다.
  2. 해당 객체의 나머지 핵심 필드 `f` 각각에 대해 다음 작업을 수행한다.
      - 해당 필드의 해시코드 `c`를 계산한다.
          - 기본 타입 필드라면 `Type.hashCode(f)`를 수행한다. 여기서 `Type`은 해당 기본 타입의 박싱 클래스이다.
          - 참조 타입 필드면서 이 클래스의 `equals` 메서드가 이 필드의 `equals`를 재귀적으로 호출해 비교한다면, 이 필드의 `hashCode`를 재귀적으로 호출한다. (필드의 값이 `null`일 경우 0을 사용)
          - 필드가 배열이라면, 핵심 원소 각각을 별도 필드처럼 다루어 각 핵심 원소의 해시 코드를 계산한 다음, 단계 `2.b` 방식을 갱신한다.
            - 배열에 핵심 원소가 하나도 없다면 단순히 상수를 사용하고, 모든 원소가 핵심 원소라면 `Arrays.hashCode`를 사용한다.
      - 단계 `2.a`에서 계산한 해시코드 `c`로 `result`를 갱신한다. 
          - `result = 31 * result + c;`
  3. `result`를 반환한다.
<br>

## 전형적인 `hashCode` 메서드
```java
@Override
public int hashCode() {
  int result = Short.hashCode(areaCode);
  result = 31 * result + Short.hashCode(prefix);
  result = 31 * result + Short.hashCode(lineNum);
  return result;
}
```
**`hashCode`를 다 구현했다면 이 메서드가 동치인 인스턴스에 대해 동일한 해시코드를 반환하는지 테스트해보자.**

**`hashCode`를 구현할 때 `equals` 비교에 사용되지 않는 필드는 반드시 제외해야 한다.**

<br>

## `Objects` 클래스가 제공하는 해시코드 메서드
- `Object` 클래스는 임의의 개수만큼 객체를 받아 해시코드를 계산해주는 정적 메서드인 `hash`를 제공한다.
- 이 메서드를 활용하면 앞서 구현한 `hashCode` 메서드와 비슷한 수준의 메서드를 한 줄로 작성할 수 있다.
- 하지만 입력 인수를 담기 위한 배열이 만들어지고, 입력 중 기본 타입이 있다면 박싱과 언박싱도 거쳐야 하기 때문에 더 느리다.
- 따라서 성능에 민감하지 않은 상황에서만 사용하자.
```java
@Override
public int hashCode() {
  return Objects.hash(lineNum, prefix, areaCode);
}
```
<br>

## `hashCode` 메서드 유의점
- 클래스가 불변이고 해시코드를 계산하는 비용이 크다면, 매번 새로 계산하기보다는 캐싱하는 방식을 고려해야 한다.
  - 이 타입의 객체가 주로 해시의 키로 사용될 것 같다면 인스턴스가 만들어질 때 해시코드를 계산해둬야 한다.
  - 반대로 해시의 키로 사용되지 않는 경우라면 `hashCode`가 처음 불릴 때 계산하는 지연 초기화 전략을 사용하자.
```java
private int hashCode; // 자동으로 0으로 초기화

@Override
public int hashCode() {
    int result = hashCode;
    if (result == 0) {                            // 지연 초기화 전략
        int result = Integer.hashCode(areaCode);
        result = 31 * result + Integer.hashCode(prefix);
        result = 31 * result + Integer.hashCode(lineNum);
        return result;
    }
    return result;
}
```
- 성능을 높인답시고 해시코드를 계산할 때 핵심 필드를 생략해서는 안 된다.
  - 속도야 빨라지겠지만, 해시 품질이 나빠져 해시테이블의 성능을 심각하게 떨어뜨릴 수도 있다.
<br>

## 정리
- **`equals`를 재정의할 때는 `hashCode`도 반드시 재정의하자.**
- **재정의한 `hashCode`는 `Object`의 API 문서에 기술된 일반 규약을 따라야 하며, 서로 다른 인스턴스라면 되도록 해시코드도 서로 다르게 구현해야 한다.**
