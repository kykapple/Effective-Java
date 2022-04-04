# Item22. 인터페이스는 타입을 정의하는 용도로만 사용하라

**인터페이스는 자신을 구현한 클래스의 인스턴스를 참조할 수 있는 타입 역할을 한다.**

달리 말해, 클래스가 어떤 인터페이스를 구현한다는 것은 자신의 인스턴스로 무엇을 할 수 있을지를 클라이언트에게 알려주는 것이다.

인터페이스는 오직 이 용도로만 사용해야 한다.

<br>

## 안티패턴 - 상수 인터페이스
위 지침에 맞지 않는 예로 상수 인터페이스가 있다.
```java
public interface PhysicalConstants {
    // 아보가드로 (1/몰)
    static final double AVOGADROS_NUMBER = 6.022_140_857e23;

    // 볼츠만 상수 (J/K)
    static final double BOLTZMANN_CONSTANT = 1.380_648_52e-23;
    
    // 전자 질량 (kg)
    static final double ELECTRON_MASS = 9.109_383_56e-31;
}
```

상수 인터페이스는 메서드 없이, 상수를 뜻하는 `static final` 필드로만 가득 찬 인터페이스를 말한다.

이와 같은 상수 인터페이스의 상수는 외부 인터페이스가 아니라 내부 구현에 해당한다. **따라서 상수 인터페이스를 구현하는 것은 이 내부 구현을 클래스의 API로 노출하는 행위이다.**

이는 사용자에게 혼란을 주기도 하며, 더 심하게는 클라이언트 코드가 내부 구현에 해당하는 이 상수들에 종속될 수도 있다.

<br>

## 상수 공개에 대한 적절한 선택지
상수를 공개할 목적이라면 다음과 같은 방법을 사용하도록 하자.
1. 특정 클래스나 인터페이스와 강하게 연관된 상수라면 클래스나 인터페이스 자체에 추가하자.
2. 열거 타입(`Enum`)으로 나타내기 적합한 상수라면 열거 타입(`Enum`)을 만들어 공개하자.
3. 인스턴스화할 수 없는 유틸리티 클래스에 담아 공개하자.
```java
public class PhysicalConstants {
    private PhysicalConstants() {}  // 인스턴스화 방지

    // 아보가드로 (1/몰)
    static final double AVOGADROS_NUMBER = 6.022_140_857e23;

    // 볼츠만 상수 (J/K)
    static final double BOLTZMANN_CONSTANT = 1.380_648_52e-23;
    
    // 전자 질량 (kg)
    static final double ELECTRON_MASS = 9.109_383_56e-31;
}
```

<br>

## 정리
**인터페이스는 타입을 정의하는 용도로만 사용해야 한다. 상수 공개용 수단으로 사용하지 말자.**
