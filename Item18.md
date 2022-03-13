# Item 18. 상속보다는 컴포지션을 사용하라

## 메서드 호출과 달리 상속은 캡슐화를 깨뜨린다.
상속을 통해 하위 클래스가 상위 클래스의 메서드를 오버라이딩하고 해당 메서드에서 상위 클래스의 동작을 사용한다면, 상위 클래스가 어떻게 구현되느냐에 따라 하위 클래스의 동작에 이상이 생길 수 있다.

**이는 상위 클래스의 구현이 하위 클래스에게 노출되기 때문에 캡슐화에 위배된다.**

이러한 이유로 상위 클래스 설계자가 확장을 충분히 고려하고 문서화도 제대로 해두지 않으면 하위 클래스는 상위 클래스의 변화에 발맞춰 수정되어야만 한다.
```java
public class InstrumentedHashSet<E> extends HashSet<E> {
  // 추가된 원소의 수
  private int addCount = 0; 
  
  public InstrumentedHashSet() {}
  
  ...
    
  @Override
  public boolean add(E e) {
    addCount++;
    return super.add(e);
  }
  
  @Override
  public boolean addAll(Collection<? extends E> c) {
    addCount += c.size();
    return super.addAll();
  }
  
  public int getAddCount() {
    return addCount;
  }
}
```
```java
InstrumentedHashSet<String> s = new InstrumentedHashSet<>();
s.addAll(List.of("틱", "탁탁", "펑"));
System.out.println(s.getAddCount());    // 6
```
위 클래스는 `HashSet`을 상속한 `InstrumentedHashSet`과 이를 사용하는 코드이다. `s.getAddCount()`를 수행하면 3이라는 값이 반환될 것 같지만 6이 반환된다.

그 이유는 `InstrumentedHashSet`이 상속하고 있는 `HashSet`의 `addAll`메서드가 `add`메서드를 사용하여 구현된 데 있다.

이러한 문제로 하위 클래스는 상위 클래스에 맞춰 수정되어야만 한다. 즉, `InstrumentedHashSet`은 깨지기 쉬운 클래스가 된다.

문제가 발생하는 원인이 메서드 오버라이딩이므로, 클래스를 확장할 때 메서드 오버라이딩 대신 새로운 메서드를 추가하면 괜찮을 것 같지만 이는 추후 상위 클래스에 시그니처가 같은 메서드가 생기게 
되면 또 깨지게 된다. 

<br>

## 컴포지션의 도입
이러한 문제를 피해갈 수 있는 방법은 기존 클래스를 확장하는 대신, 새로운 클래스를 만들고 `private` 필드로 기존 클래스의 인스턴스를 참조하게 하면 된다.

**기존 클래스가 새로운 클래스의 구성요소로 쓰인다는 뜻에서 이러한 설계를 컴포지션이라고 한다.**

새로운 클래스의 인스턴스 메서드들은 기존 클래스의 대응하는 메서드를 호출해 그 결과를 반환한다. 이 방식을 전달(forwarding)이라고 하며, 새 클래스의 메서드들을 전달 메서드라 부른다.

**그 결과 새로운 클래스는 기존 클래스의 내부 구현 방식의 영향에서 벗어나며, 심지어 기존 클래스에 새로운 메서드가 추가되더라도 전혀 영향받지 않는다.**

앞서 구현한 `InstrumentedHashSet`을 컴포지션과 전달 방식으로 구현해보도록 하자.
```java
public class InstrumentedSet<E> extends ForwardingSet<E> {
  private int addCount = 0;
  
  public InstrumentedSet(Set<E> s) {
    super(s);
  }
   
  @Override
  public boolean add(E e) {
    addCount++;
    return super.add(e);
  }
  
  @Override
  public boolean addAll(Collection<? extends E> c) {
    addCount += c.size();
    return super.addAll();
  }
  
  public int getAddCount() {
    return addCount;
  }
}
```
```java
public class ForwardingSet<E> implements Set<E> {
  private final Set<E> s;
  public ForwardingSet(Set<E> s) { this.s = s}
  
  public boolean add(E e) { return set.add(e); }
  public boolean addAll(Collection<? extends E> c) { return set.addAll(c); }
  
  ...
}
```
`InstrumentedSet`에서 임의의 `Set`에 계측 기능을 덧씌워 새로운 `Set`으로 만들었다.

**`InstrumentedSet`는 `Set`의 메서드들을 오버라이딩하여 호출하는 것이 아니라 상위 전달 클래스(`ForwardingSet`)가 필드로 가지고 있는 `Set`의 메서드를 호출하므로 상속에서 발생한 문제가 
발생하지 않고, 캡슐화 또한 깨지지 않는다.**

> 다른 `Set` 인스턴스를 감싸고 있다는 뜻에서 `InstrumentedSet` 클래스를 래퍼 클래스라고 한다.

<br>

## 래퍼 클래스의 단점
래퍼 클래스는 단점이 거의 없다.

한 가지, 래퍼 클래스가 콜백 프레임워크와는 잘 어울리지 않는다.

이는 SELF 문제 때문인데, 콜백 프레임워크에서는 자기 자신의 참조를 다른 객체에 넘겨서 다음 호출 때 사용하도록 한다. 내부 객체는 자신을 감싸고 있는 래퍼의 존재를 모르니 대신 자신(`this`)의 
참조를 넘기고, 콜백 때는 래퍼가 아닌 내부 객체를 호출하게 된다.

> [콜백에 대한 참고자료](https://stackoverflow.com/questions/28254116/wrapper-classes-are-not-suited-for-callback-frameworks)

> 전달 메서드가 성능에 주는 영향이나 래퍼 객체가 메모리 사용량에 주는 영향을 걱정할 수도 있지만, 이는 별다른 영향을 주지 않는다.

<br>

## 상속은 언제 사용해야 하는가?
**상속 반드시 하위 클래스가 상위 클래스의 '진짜' 하위 타입인 상황에서만 사용해야 한다.**

다르게 말하면, 클래스 B가 클래스 A와 `is-a` 관계일 때만 클래스 A를 상속해야 한다.

만약 "B가 정말 A인가?" 라는 질문에 대한 대답이 "아니다" 라면 A를 `private` 인스턴스로 두고, A와는 다른 API를 제공해야 한다.
<br>

**컴포지션을 써야 할 상황에서 상속을 사용하는 건 내부 구현을 불필요하게 노출하는 꼴이다. -> 캡슐화 위반**

그 결과 `API`가 내부 구현에 묶이고 그 클래스의 성능도 영원히 제한된다.
<br>

컴포지션 대신 상속을 사용하기로 결정하기 전에 마지막으로 "확장하려는 클래스의 `API`에 아무런 결함이 없는가?" "결함이 있다면, 이 결함이 새로운 클래스의 `API`까지 전파돼도 괜찮은가?"를 자문해야 한다.

컴포지션으로는 이러한 결함을 숨기는 새로운 `API`를 설계할 수 있지만, 상속은 상위 클래스의 `API`를 그 결함까지도 그대로 계승한다.

<br>

## 정리
- 상속은 강력하지만 캡슐화를 해친다는 문제가 있다.
- 상속은 상위 클래스와 하위 클래스가 순수한 `is-a` 관계일 때만 써야 한다.
  - `is-a` 관계일 때도 안심할 수만은 없는 게, 하위 클래스의 패키지가 상위 클래스와 다르고, 상위 클래스가 확장을 고려해 설계되지 않았다면 여전히 문제가 될 수 있다.
- 상속의 취약점을 피하려면 상속 대신 컴포지션과 전달을 사용하자.
  - 특히 래퍼 클래스로 구현할 적당한 인터페이스가 있다면 더욱 그렇다. 래퍼 클래스는 하위 클래스보다 견고하고 강력하다.
