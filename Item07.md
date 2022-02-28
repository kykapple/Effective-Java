# Item07. 다 쓴 객체 참조를 해제하라
- 자바처럼 가비지 컬렉터를 갖춘 언어는 다 쓴 객체를 알아서 회수해가니 메모리 관리에 더 이상 신경 쓰지 않아도 될거라고 오해할 수 있는데, 그렇지 않다.

### 메모리를 직접 관리하는 객체
```java
public class Stack {

    private Object[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;

    public Stack() {
        this.elements = new Object[DEFAULT_INITIAL_CAPACITY];
    }

    public void push(Object e) {
        this.ensureCapacity();
        this.elements[size++] = e;
    }

    public Object pop() {
        if (size == 0) {
            throw new EmptyStackException();
        }
        return this.elements[--size];
    }

    /**
     * 원소를 위한 공간을 적어도 하나 이상 확보한다.
     * 배열 크기를 늘려야 할 때마다 대략 두 배씩 늘린다.
     */
    private void ensureCapacity() {
        if (this.elements.length == size) {
            this.elements = Arrays.copyOf(elements, 2 * size + 1);
        }
    }
}
```
위에서 구현한 스택은 메모리 누수라는 문제점을 가지고 있는데, 어디서 메모리 누수가 일어날까?

이 코드에서는 스택에 데이터를 `push` 하다가 `pop` 한다고 하도 스택에서 꺼내진 객체들을 가비지 컬렉터가 회수하지 않는다.

따라서 스택이 차지하고 있는 메모리가 줄어들지 않는다.

이 이유는 스택이 `pop` 한 객체들의 참조를 여전히 가지고 있기 때문이다. 

스택에서 (`size`보다 작은 영역)활성 영역 밖(`size` 보다 큰 영역)의 참조들이 모두 불필요하게 메모리를 차지하고 있는 부분이다.

이러한 메모리 누수를 해결하기 위해서는 다 쓴 참조를 `null` 처리 해주는 것이다.

다음은 개선된 스택의 `pop` 메서드이다.
```java
public Object pop() {
    if (size == 0) {
        throw new EmptyStackException();
    }
  
    Object value = this.elements[--size];
    this.elements[size] = null; // 다 쓴 참조 해제
    return value;
}
```
이제는 스택에서 데이터를 `pop` 할 때 다 쓴 참조를 `null` 처리해주기 때문에 가비지 컬렉터가 메모리 공간을 회수해갈 수 있게 된다.

그렇다고 모든 객체를 다 쓰자마자 일일이 `null` 처리를 하지는 말자. 오히려 코드를 지저분하게 만들 뿐이다. **객체의 참조를 `null` 처리하는 일은 예외적인 경우여야 한다.** 

다 쓴 참조를 해제하는 가장 좋은 방법은 그 참조를 담은 변수를 유효 범위 밖으로 밀어내는 것이다. 변수의 범위를 최소가 되게 정의했다면 이는 자연스레 이뤄질 것이다. (예를 들어 지역 변수는 해당 지역 변수가 속해있는 메서드가 종료되면 GC에 의해 정리된다.)

그렇다면 `null` 처리는 언제 해야 하는가? **메모리를 직접 관리할 때이다.**

앞서 구현한 스택처럼 `elements` 라는 배열을 직접 관리하는 경우에 GC는 어떤 객체들이 더 이상 쓸모없는 객체인지 알 수가 없다.
프로그래머만이 비활성 영역의 객체들이 더 이상 쓸모없다는 것을 안다. 따라서 프로그래머는 비활성 영역이 되는 순간 `null` 처리를 통해 해당 객체를 더 이상 쓰지 않을 것임을 GC에 알려야 한다.

**결국 메모리를 직접 관리하는 클래스라면 프로그래머가 메모리 누수에 주의해야 한다.**

### 캐시
캐시를 사용할 때도 메모리 누수에 조심해야 한다. 객체 참조를 캐시에 넣어두고 캐시를 비우는 것을 잊기 쉽다.
여러 가지 해법이 있지만, 캐시의 키에 대한 참조가 외부에서 더 이상 필요 없어지면 자동으로 해당 엔트리를 제거해주는 `WeakHashMap` 을 사용할 수 있다.
> [WeakHashMap 이란](https://blog.breakingthat.com/2018/08/26/java-collection-map-weakhashmap/)

### 콜백
클라이언트가 콜백을 등록만 하고 명확히 해지하지 않는다면, 계속 쌓여만 갈 것이다.
이럴 때 콜백을 약한 참조로 저장하면 GC가 즉시 수거해간다. 예를 들어 `WeakHashMap` 에 키로 저장하면 된다.

## 정리
메모리 누수는 겉으로 잘 드러나지 않아 시스템에 수년간 잠복하는 사례도 있다.
이런 누수는 철저한 코드 리뷰가 힙 프로파일러 같은 디버깅 도구를 동원해야만 발견되기도 한다.
그래서 이런 문제의 예방법을 학습하여 메모리 누수를 미연에 방지하는 것이 좋다.
