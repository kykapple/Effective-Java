# Item30. 이왕이면 제네릭 메서드로 만들라
- 클래스와 마찬가지로, 메서드도 제네릭으로 만들 수 있다.
- 매개변수화 타입을 받는 정적 유틸리티 메서드는 보통 제네릭이다.

<br>

## 제네릭 메서드를 이용해서 타입 안전성을 보장하자.
```java
public static Set union(Set s1, Set s2) {
    Set result = new HashSet(s1);
    result.addAll(s2);
    return result;
}
```
- 위 메서드는 raw 타입을 사용하고 있기 때문에 경고가 발생하고, 각 집합이 어떤 타입을 담고 있는지 모르기 때문에 런타임 에러가 발생할 수 있다.
- 따라서 제네릭을 사용하여 타입 안전성을 보장해주어야 한다.
```java
public static <E> Set<E> union(Set<E> s1, Set<E> s2) {
    Set<E> result = new HashSet<>();
    result.addAll(s2);
    return result;
}
```

<br>

## 제네릭 싱글턴 팩터리
- 때때로 불변 객체를 여러 타입으로 활용할 수 있게 만들어야 할 때가 있다.
- 제네릭은 런타임에 타입 정보가 소거되므로 하나의 객체를 어떤 타입으로든 매개변수화할 수 있는데, 이를 활용하여 제네릭 싱글턴 팩터리를 만들면 된다.
> 만약 제네릭이 없었다면 타입 별로 생성해주었어야 할 것이다.
```java
public static <T> Comparator<T> reverseOrder() {
    return (Comparator<T>) ReverseComparator.REVERSE_ORDER;
}
```

<br>

## 재귀적 타입 한정
- 자기 자신이 들어간 표현식을 사용하여 타입 매개변수의 허용 범위를 한정할 수 있다.
- 예를 들어 `<T extends Comparable<T>>`는 자신과 같은 타입의 원소만 허용한다는 것이고, 동시에 모든 타입 `T`는 자신과 비교할 수 있다는 의미이다.
```java
public static <E extends Comparable<E>> E max(Collection<E> c) {
    if (c.isEmpty())
        throw new IllegalArgumentException("컬렉션이 비어 있습니다.");
    
    E result = null;
    for (E e : c)
        if (result == null || e.compareTo(result) > 0)
            result Object.requireNonNull(e);
    
    return result;
}
```

<br>

## 정리
- 클라이언트에서 입력 매개변수와 반환값을 명시적으로 형변환해야 하는 메서드보다 제네릭 메서드가 더 안전하며 사용하기도 쉽다.
- 형변환 해줘야 하는 기존 메서드는 제네릭하게 만들자!
