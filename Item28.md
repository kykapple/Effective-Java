# Item28. 배열보다는 리스트를 사용하라

## 배열과 제네릭 타입의 차이 1. 공변 vs 불공변
- 배열은 공변이지만, 제네릭 타입은 불공변이다.
- 따라서 배열은 `Sub`가 `Super`의 하위 타입이라면 배열 `Sub[]`는 배열 `Super[]`의 하위 타입이 된다.
- 반면, 제네릭 타입에서는 `List<Sub>`와 `List<Super>`가 아무런 관계가 없다.
> 공변은 함께 변한다는 뜻이다.

### 공변 vs 불공변 예시
```java
Object[] arr = new Long[1];                // 컴파일 성공
arr[0] = "타입이 달라 넣을 수 없다.";        // 런타임 때 ArrayStoreException 발생

List<Object> list = new ArrayList<Long>(); // 컴파일 실패
list.add("타입이 달라 넣을 수 없다.");
```
- 배열의 경우, 공변이기 때문에 잘못된 자료형의 데이터를 넣었더라도 컴파일 타임 때 발견하지 못한다.
- 하지만 리스트의 경우, 불공변이기 때문에 `list`를 선언할 때 자료형이 다르다면 컴파일되지 않는다.
- **즉, 배열을 사용하면 런타임에 예외가 발생하지만 리스트를 사용하면 컴파일 타임에 예외를 방지할 수 있다는 것이다.**

<br>

## 배열과 제네릭 타입의 차이 2. 실제화 vs 타입 정보 소거
- 배열은 런타임에도 자신이 담기로 한 원소의 타입을 인지하고 확인한다.
  - 타입 안전성을 보장하기 힘들다.
- 반면, 제네릭은 원소의 타입을 컴파일 타임에만 검사하며 런타임에는 알 수 없다.
  - 즉, 컴파일 타임에 타입을 체크하고, 런타임 때는 타입을 제거한다는 것이다.
  ```java
  // 컴파일 타임
  class Tiger<T extends Animal> {
      void foo(T animal) {
          // ... 
      }
  }

  // 런타임
  class Tiger {
      void foo(Animal animal) {
          // ... 
      }
  }
  ```
  ```java
  // 컴파일 타임
  public class Temp<T> {
      void foo(T test) {
          // ...
      }
  }
  // 런타임
  public class Temp {
      void foo(Object test) {
          // ...
      }
  }
  ```
> 타입 정보 소거는 [여기서](https://www.baeldung.com/java-type-erasure) 더 자세히 알아볼 수 있다.
<br>

## 배열과 제네릭은 잘 어우러지지 못한다.
- 이상의 주요 차이로 배열과 제네릭은 잘 어우러지지 못한다.
- 예컨대 배열은 제네릭 타입, 매개변수화 타입, 타입 매개변수로 사용할 수 없다.
```java
// 컴파일 오류 발생
new List<E>[];      // 제네릭 타입
new List<String>[]; // 매개변수화 타입
new E[];            // 타입 매개변수
```

### 제네릭 배열을 만들지 못하게 막은 이유는 무엇일까?
- 타입 안전하지 않기 때문이다.
- 이를 허용한다면 컴파일러가 자동 생성한 형변환 코드에서 런타임에 `ClassCastException`이 발생할 수 있다.
- **이는 런타임에 `ClassCastException`이 발생하는 일을 막아주는 제네릭 타입 시스템의 취지에 어긋나는 것이다.**
```java
List<String>[] stringLists = new List<String>[10]; // (1)
List<Integer> intList = List.of(45);               // (2)
Object[] objects = stringLists;                    // (3)
objects[0] = intList;                              // (4)
String s = stringLists[0].get(0);                  // (5)
```
- 위 코드는 제네릭 배열을 사용하는 코드로, `ClassCastException`을 유발한다.
- 해당 코드를 그림으로 나타내면 아래와 같다.

![image](https://user-images.githubusercontent.com/76088639/167461765-517c3d9a-bf5d-4ab5-bb73-fa836a17dd47.png)

<br>

## 배열에서 제네릭 타입으로 넘어가기
### 배열
- 생성자에서 컬렉션을 받는 `Chooser` 클래스를 예로 살펴보자.
```java
public class Chooser {
    private final Object[] choiceArray;

    public Chooser(Collection choices) {
        this.choiceArray = choices.toArray();
    }
  
    public Object choose() {
        Random rnd = ThreadLocalRandom.current();
        return choiceArray[rnd.nextInt(choiceArray.length)];
    }
}
```
- 위 클래스를 사용하면, `choose` 메서드를 호출할 때마다 반환된 `Object`를 원하는 타입을 형변환해야 한다.
- 혹시나 다른 타입의 원소가 들어 있었다면 런타임에 형변환 오류가 날 수도 있다.

### 제네릭 도입
- 따라서 타입 안전성을 위해 제네릭으로 만들어보도록 하자.
```java
public class Chooser<T> {
    private final T[] choiceArray;

    public Chooser(Collection<T> choices) {
        this.choiceArray = (T[]) choices.toArray();
    }
  
    // ...
}
```
- 생성자로 들어오는 컬렉션과 배열을 제네릭 타입으로 선언했다.
- 하지만 위 코드에서도 타입 정보 소거로 인해 런타임에 무슨 타입인지 알 수 없어 경고가 발생하기는 하지만, 이는 코드를 작성하는 사람이 안전하다고 판단한다면 주석을 남기고 애너테이션을 달아 경고를 숨겨도 된다.

### 리스트
- 비검사 형변환 경고를 완전히 제거하고 싶다면 배열 대신 리스트를 쓰면 된다.
```java
public class Chooser<T> {
    private final List<T> choiceList;
    
    public Chooser(Collection<T> choices) {
        choiceList = new ArrayList<>(choices);
    }

    public T choose() {
        Random rnd = ThreadLocalRandom.current();
        return choiceList.get(rnd.nextInt(choiceList.size()));
    }
}
```
- 코드양은 조금 늘었지만, 런타임에 `ClassCastException`을 만날 일은 없으니 그만한 가치가 있다.

<br>

## 정리
- 배열과 제네릭에는 매우 다른 타입 규칙이 적용된다.
- 배열은 공변이고 실체화되는 반면, 제네릭은 불공변이고 타입 정보가 소거된다.
- 그 결과 배열은 런타임에 타입 안전하지만 컴파일 타임에는 그렇지 않다. 제네릭은 그 반대이다.
