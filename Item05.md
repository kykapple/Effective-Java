# Item05. 자원을 직접 명시하지 말고 의존 객체 주입을 사용하라
- 많은 클래스가 하나 이상의 자원에 의존한다.
  - 예를 들어 아래와 같이 `SpellChecker` 클래스는 `Lexicon` 타입의 자원을 사용한다.
<br>

### 부적절한 구현1: 정적 유틸리티 
```java
interface Lexicon {}

class KoreanDictionary implements Lexicon {}

public class SpellChecker {
    
    private static final Lexicon dictionary = new KoreanDictionary();
    
    private SpellChecker{   // 객체 생성 방지
    }
    
    public static boolean isValid(String word) { ... }
    
    public static List<String> suggestions(String typo) { ... }
}
```
### 부적절한 구현2: 싱글턴
```java
public class SpellChecker {

    private final Lexicon dictionary = new KoreanDictionary();
    
    private SpellChecker() {
    }
    
    public static final SpellChecker INSTANCE = new SpellChecker();
    
    public boolean isValid(String word) { ... }

    public List<String> suggestions(String typo) { ... }
}
```
- 위 두 가지 구현 방식 모두 단 하나의 `KoreanDictionary` 를 사용한다는 점에서 좋지 않은 구현 방식이다.
  - 언어별로 사전이 따로 있기 때문에 `KoreanDictionary`로 모든 경우를 대응하기는 힘들다.
- 필드에서 `final` 한정자를 제거하고 `setter` 메서드를 추가하여 다른 `Dictionary`로 교체할 수도 있지만 이는 `Thread-safe` 하지 못하다.
- 따라서 사용하는 자원에 따라 동작을 달리해야 하는 경우에는 정적 유틸리티 클래스나 싱글턴 방식이 적합하지 않다.
 
**`SpellChecker` 클래스는 여러 자원 인스턴스를 지원해야 하며, 클라이언트가 원하는 자원(`Dictionary`)을 사용해야 한다.**

이 조건을 만족하는 간단한 패턴으로 인스턴스를 생성할 때 생성자에 필요한 자원을 넘겨주는 방법이 있다.

### 적절한 구현: 의존성 주입
```java
public class SpellChecker {
    
    private final Lexicon dictionary;
    
    public SpellChecker(Lexicon dictionary) {
        this.dictionary = Objects.requireNonNull(dictionary);
    }
    
    public boolean isValid(String word) { ... }

    public List<String> suggestions(String typo) { ... }
}
```
- 의존성 주입 방법은 자원에 대한 의존성을 쉽게 바꿔 끼울 수 있도록 해주어 유연함을 가져다 준다.
- 또한 불변을 보장하여 같은 자원을 사용하려는 여러 클라이언트가 의존 객체들을 안심하고 공유할 수 있기도 하다.
  - 생성자를 통해 구현체를 주입 받으므로 불변이 보장되고, `Thread-safe` 하게 동작한다.
- 이러한 의존성 주입은 생성자 뿐만 아니라 정적 팩터리, 빌더에도 적용할 수 있다.

추가적으로 이 패턴의 변형으로, 생성자에 자원 팩터리를 넘겨주는 방식이 있다.
- 팩터리란 호출할 때마다 특정 타입의 인스턴스를 반복해서 만들어주는 객체를 말하며, 자바 8에서 등장한 `Supplier<T>` 인터페이스가 팩터리를 표현한 완벽한 예이다.
```java
public class SpellChecker {

    private final Lexicon dictionary;

    public SpellChecker(Supplier<? extends Lexicon> supplier) {
        this.dictionary = Objects.requireNonNull(supplier.get());
    }

    public static void main(String[] args) {
        SpellChecker spellChecker = new SpellChecker(() -> new KoreanDictionary());
    }
}
```

## 정리
- 클래스가 의존하는 자원에 따라 동작을 달리한다면 싱글턴과 정적 유틸리티 클래스는 사용하지 않는 것이 좋다.
- 대신 필요한 자원을 생성자(혹은 정적 팩터리나 빌더)에 전달하는 의존성 주입을 사용하여 클래스의 유연성, 재사용성, 테스트 용이성을 향상 시키도록 하자.
