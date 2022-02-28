# Item06. 불필요한 객체 생성을 피하라
- 똑같은 기능의 객체를 매번 생성하기보다는 객체 하나를 재사용하는 편이 적절할 때가 많다.
- 재사용은 빠르고 세련되며, 불변객체는 항상 재사용할 수 있다.

### 문자열 객체 생성
다음 코드는 하지 말아야 할 극단적인 예이다.
```java
String s = new String("apple");
```
이 코드는 실행될 때마다 String 인스턴스를 새로 만들게 되므로, 해당 문장이 빈번히 호출되는 메서드 안에 있다면 불필요한 String 인스턴스가 여러 개 만들어질 수도 있다.

이를 개선해보자면 아래와 같다.
```java
String s = "apple";
```
이 코드는 새로운 인스턴스를 매번 만드는 대신 하나의 String 인스턴스를 사용한다. 따라서 같은 자바 가상 머신 안에서 이와 동일한 문자열 리터럴을 사용하는 코드들은 모두 같은 객체를 재사용한다.

### 정적 팩터리 메서드
정적 팩터리 메서드를 제공하는 불변 클래스에서는 정적 팩터리 메서드를 사용해 불필요한 객체 생성을 피할 수 있다.

생성자는 호출할 때마다 새로운 객체를 생성하지만 팩터리 메서드는 그렇지 않다. 

예를 들어 `Boolean(String)` 생성자 대신 `Boolean.valueOf(String)` 팩터리 메서드를 사용할 수 있다.

### 비싼 객체
생성 비용이 비싼(메모리나 시간이 많이 드는) 객체가 반복해서 필요하다면 캐싱하여 재사용하는 것이 좋다.

정규표현식에 대한 예제를 살펴보도록 하자.
```java
static boolean isRomanNumeral(String s) {
    return s.matches("^(?=.)M*(C[MD]|D?C{0,3})(X[CL]|L?X{0,3})(I[XV]|V?I{0,3})$");
}
```
`String.matches` 메서드는 정규표현식으로 문자열 형태를 확인하는 가장 쉬운 방법이지만, 성능이 중요한 상황에서 반복적으로 사용하기엔 적합하지 않다.

이 메서드는 내부적으로 정규표현식용 `Pattern` 객체를 매번 만들어서 사용하는데, `Pattern`은 입력받은 정규표현식에 해당하는 유한 상태 머신을 만들기 때문에 객체 생성 비용이 높다.

성능을 개선하려면 필요한 정규표현식을 표현하는 `Pattern` 인스턴스를 만들어서 재사용하는 것이 좋다.
```java
public class RomanNumerals {

    private static final Pattern ROMAN = Pattern.compile("^(?=.)M*(C[MD]|D?C{0,3})(X[CL]|L?X{0,3})(I[XV]|V?I{0,3})$");
    
    static boolean isRomanNumeral(String s) {
        return ROMAN.matcher(s).matches();
    }
}
```

### 어댑터
객체가 불변이라면 재사용해도 안전하다는 것이 명백하다. 하지만 훨씬 덜 명확한 상황도 있는데 어댑터를 생각해보자.

어댑터는 실제 작업은 뒷단 객체에 위임하고, 자신은 제 2의 인터페이스 역할을 해주는 객체이므로 여러 개 만들 필요가 없다.
```java
class Item06 {

    public static void main(String[] args) {
        Map<String, String> fruits = new HashMap<>();
        fruits.put("apple", "red");
        fruits.put("grape", "purple");

        Set<String> keySet1 = fruits.keySet();
        Set<String> keySet2 = fruits.keySet();

        keySet1.remove("apple");
        System.out.println(keySet2.size()); // 1
        System.out.println(fruits.size());  // 1
    }
}
```
`Map` 인터페이스의 `keySet` 메서드는 `Map` 객체 안의 키 전부를 담은 `Set` 어댑터를 반환한다. `keySet`을 호출할 때마다 새로운 `Set` 인스턴스가 생성될 것 같지만 사실 같은 객체를 반환한다.

따라서 반환 받은 `Set` 객체 중 하나를 수정하면 다른 모든 객체가 따라서 바뀐다.

이로 인해 `Map` 인스턴스나 `keySet` 메서드로 반환 받은 `Set` 인스턴스를 여러 곳에서 사용한다고 했을 때 신뢰하고 사용할 수 없다. 

### 오토박싱
불필요한 객체를 만들어내는 또 다른 예로 오토박싱이 있다. 
> 오토박싱은 프로그래머가 기본 타입과 박싱된 기본 타입을 섞어 쓸 때 자동으로 상호 변환해주는 기술이다.

**오토박싱은 기본 타입과 그에 대응하는 박싱된 기본 타입의 구분을 흐려주지만 완전히 없애주는 것은 아니다.**

다음 예제를 보도록 하자.
```java
private static long sum() {
    Long sum = 0L;
    for (long i = 0; i <= Integer.MAX_VALUE; i++) {
        sum += i;    
    }
    return sum;
}
```
이 메서드를 보면 `sum` 변수의 타입을 `long` 이 아닌 `Long` 으로 선언하였기 때문에 `sum += i` 가 수행될 때마다 불필요한 `Long` 인스턴스가 2^31개나 만들어지게 되어 성능이 매우 느리다.

하지만 타입을 `long` 으로 바꿔주면 불필요한 객체 생성 과정이 사라지기 때문에 성능을 크게 향상시킬 수 있다.

**박싱된 기본 타입(Reference type)보다는 기본 타입(Primitive type)을 사용하고, 의도치 않은 오토박싱이 숨어들지 않도록 주의해야 한다.**

## 정리
- 기능적으로 동일한 객체를 새로 매번 새로 만들기보다는 재사용하는 것을 고려해보자.
- 그렇다고 "객체 생성은 비싸니 피해야 한다" 로 오해하지는 말자.
  - 프로그램의 명확성, 간결성, 기능을 위해 객체를 추가로 생성하는 것이라면 일반적을 좋은 일이다. 