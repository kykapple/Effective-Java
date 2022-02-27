# Item02. 생성자에 매개변수가 많다면 빌더를 고려하라
- [Item01](https://github.com/kykapple/Effective-Java/blob/main/Item01.md)에서 알아본 정적 팩터리와 생성자는 선택적 매개변수가 많을 때 적절히 대응하기 어렵다는 제약이 있다.
- 그렇다면 선택적 매개변수가 많을 때 어떠한 패턴을 사용할 수 있을지 알아보도록 하자.
<br>

### 점층적 생성자 패턴
- 필수 매개변수만 받는 생성자, 필수 매개변수와 선택 매개변수 1개를 받는 생성자, 선택 매개변수를 2개까지 받는 생성자 , ..., 형태로 선택 매개변수를 전부 다 받는 생성자까지 늘려가는 방식이다.
- 다음은 점층적 생성자 패턴의 예시이다.
```java
public class NutritionFacts {

  private final int servingSize;
  private final int servings;
  private final int calories;
  private final int fat;
  private final int sodium;
  private final int carbohydrate;

  public NutritionFacts(int servingSize, int servings) {
      this(servingSize, servings, 0);
  }

  public NutritionFacts(int servingSize, int servings, int calories) {
      this(servingSize, servings, calories, 0);
  }

  public NutritionFacts(int servingSize, int servings, int calories, int fat) {
      this(servingSize, servings, calories, fat, 0);
  }

  public NutritionFacts(int servingSize, int servings, int calories, int fat, int sodium) {
      this(servingSize, servings, calories, fat, sodium, 0);
  }

  public NutritionFacts(int servingSize, int servings, int calories, int fat, int sodium, int carbohydrate) {
      this.servingSize = servingSize;
      this.servings = servings;
      this.calories = calories;
      this.fat = fat;
      this.sodium = sodium;
      this.carbohydrate = carbohydrate;
  }
  
  public static void main(String[] args) {
        NutritionFacts cocaCola = new NutritionFacts(240, 8, 100, 0, 35, 27);
  }
    
}
```
- 이러한 점층적 생성자 패턴은 매개변수가 많아질수록 **클라이언트 코드를 작성하거나 읽기 어려워진다.**
  - 코드를 읽을 때 각 값의 의미가 무엇인지 헷갈리고, 매개변수가 몇 개인지도 주의해서 세어 보아야 한다.
  - 결국 휴먼 에러가 발생할 가능성이 많아지고, 이로 인해 런타임에 엉뚱한 동작을 할 수도 있다.
<br>

### 자바빈즈 패턴
- 매개변수가 없는 생성자로 객체를 만든 후, 세터(`setter`) 메서드를 호출해 원하는 매개변수의 값을 설정하는 방식이다.
- 점층적 생성자 패턴에 비해 인스턴스를 만들기 쉽고, `setter`를 사용하기 때문에 코드도 읽기 쉬워진다.
```java
public class NutritionFacts {

  private int servingSize = -1;
  private int servings = -1;
  private int calories = 0;
  private int fat = 0;
  private int sodium = 0;
  private int carbohydrate = 0;

  public NutritionFacts() {
  }

  public void setServingSize(int servingSize) { this.servingSize = servingSize; }

  public void setServings(int servings) { this.servings = servings; }

  public void setCalories(int calories) { this.calories = calories; }

  public void setFat(int fat) { this.fat = fat; }

  public void setSodium(int sodium) { this.sodium = sodium; }

  public void setCarbohydrate(int carbohydrate) { this.carbohydrate = carbohydrate; }

  public static void main(String[] args) {
      NutritionFacts cocaCola = new NutritionFacts();
      cocaCola.setServingSize(240);
      cocaCola.setServings(8);
      cocaCola.setCalories(100);
      cocaCola.setSodium(35);
      cocaCola.setCarbohydrate(27);
  }

}

```
- 하지만 이러한 자바빈즈 패턴은 객체 하나를 만드려면 메서드를 여러 개 호출해야 하고, 객체가 완전히 생성되기 전까지는 일관성이 무너진 상태에 놓이게 된다.
  - 위 코드에서 `cocaCola.setServings()` 까지만 설정하고 사용될 수도 있다. 
  - 안정적이지 않은 상태로 사용될 여지가 있다는 것이다.
- 또한 `setter`가 있기 때문에 불변 클래스로 만들지 못한다는 단점이 있다.
- 더불어 쓰레드 간 공유 가능한 상태가 있고, `setter`를 이용해서 이를 변경할 수 있기 때문에 `Thread-safe` 하지 않다. 따라서 멀티 쓰레드 환경에서 사용할 때는 추가적인 `locking` 작업이 필요하다.
<br>

### 빌더 패턴
- 점층적 생성자 패턴의 안전성과 자바빈즈 패턴의 가독성을 겸비한 패턴이다.
- 클라이언트는 필요한 객체를 직접 만드는 대신, 필수 매개변수를 가지고 `Builder`를 호출해 `Builder` 객체를 얻고, `Builder` 객체가 제공하는 일종의 `setter` 메서드들로 원하는 필드들을 설정한 다음, 마지막으로 매개변수가 없는 `build` 메서드를 호출해 객체를 생성한다.
```java
public class NutritionFacts {

  private final int servingSize;
  private final int servings;
  private final int calories;
  private final int fat;
  private final int sodium;
  private final int carbohydrate;

  public static class Builder {
      // 필수 매개변수
      private final int servingSize;
      private final int servings;

      // 선택 매개변수 - 기본값으로 초기화한다.
      private int calories = 0;
      private int fat = 0;
      private int sodium = 0;
      private int carbohydrate = 0;

      public Builder(int servingSize, int servings) {
          this.servingSize = servingSize;
          this.servings = servings;
      }

      public Builder calories(int calories) {
          this.calories = calories;
          return this;
      }

      public Builder fat(int fat) {
          this.fat = fat;
          return this;
      }

      public Builder sodium(int sodium) {
          this.sodium = sodium;
          return this;
      }

      public Builder carbohydrate(int carbohydrate) {
          this.carbohydrate = carbohydrate;
          return this;
      }

      public NutritionFacts build() {
          return new NutritionFacts(this);
      }

  }

  public NutritionFacts(Builder builder) {
      this.servingSize = builder.servingSize;
      this.servings = builder.servings;
      this.calories = builder.calories;
      this.fat = builder.fat;
      this.sodium = builder.sodium;
      this.carbohydrate = builder.carbohydrate;
  }

  public static void main(String[] args) {
      NutritionFacts cocaCola = new NutritionFacts.Builder(240, 8)
              .calories(100)
              .sodium(35)
              .carbohydrate(27)
              .build();
  }

}

```
- 이러한 빌더 패턴은 빌더의 생성자와 메서드에서 입력 매개변수의 유효성을 검사하여 검증에 실패할 경우 적절한 예외와 메시지로 어떤 매개변수가 잘못되었는지 알려줄 수 있다.
- 또한 빌더 패턴은 계층적으로 설계된 클래스와 함께 쓰기에 좋다.
  - 추상 클래스는 추상 빌더를 가지고 있고, 구체 클래스와 구체 빌더는 각각 추상 클래스와 추상 빌더를 상속 받아 만들 수 있다.
  - 이에 대한 예제 코드는 아래와 같다.
```java
public abstract class Pizza {

  public enum Topping {
      HAM, MUSHROOM, ONION, PEEPER, SAUSAGE
  }

  final Set<Topping> toppings;

  abstract static class Builder<T extends  Builder<T>> {  // 재귀적인 타입 매개변수
      EnumSet<Topping> toppings = EnumSet.noneOf(Topping.class);

      public T addTopping(Topping topping) {
          toppings.add(Objects.requireNonNull(topping));
          return self();
      }

      abstract Pizza build();

      protected abstract T self(); // 하위 클래스에서 형변환하지 않고도 메서드 체이닝을 할 수 있도록 self-type 추가
  }

  Pizza(Builder<?> builder) {
      toppings = builder.toppings.clone();
  }
}
```
```java
public class NyPizza extends Pizza {

  public enum Size {
      SMALL, MEDIUM, LARGE
  }

  private final Size size;

  public static class Builder extends Pizza.Builder<Builder> {
      private final Size size;

      public Builder(Size size) {
          this.size = Objects.requireNonNull(size);
      }


      @Override
      public NyPizza build() {
          return new NyPizza(this);
      }

      @Override
      protected Builder self() {
          return this;
      }
  }

  private NyPizza(Builder builder) {
      super(builder);
      size = builder.size;
  }
}
```
``` java
public class Calzone extends Pizza {

  private final boolean sauceInside;

  public static class Builder extends Pizza.Builder<Builder> {
      private boolean sauseInside = false;

      public Builder sauceInde() {
          sauseInside = true;
          return this;
      }

      @Override
      public Calzone build() {
          return new Calzone(this);
      }

      @Override
      protected Builder self() {
          return this;
      }
  }

  private Calzone(Builder builder) {
      super(builder);
      sauceInside = builder.sauseInside;
  }
}
```
- 여기서 주목할 만한 점은 추상 빌더가 `self` 메서드를 이용해 `self-type` 개념을 사용할 수 있도록 했다는 점과,
- 하위 클래스의 `build` 메서드가 상위 클래스의 `build` 메서드가 정의한 반환 타입이 아닌 그 하위 타입을 반환하는 공변 반환 타이핑을 사용하여 클라이언트가 형변환에 신경 쓰지 않고도 빌더를 사용할 수 있도록 했다는 점이다.
```java
public static void main(String[] args) {
  NyPizza nyPizza = new NyPizza.Builder(NyPizza.Size.SMALL)
          .addTopping(Pizza.Topping.SAUSAGE)
          .addTopping(Pizza.Topping.ONION)
          .build();

  Calzone calzone = new Calzone.Builder()
          .addTopping(Pizza.Topping.HAM)
          .sauceInde()
          .build();
}
```
- 앞서 살펴본 빌더 패턴은 상당히 유연하다. 빌더에 넘기는 매개변수에 따라서 다른 객체를 만들 수도 있고, 객체마다 부여되는 일련번호와 같은 특정 필드는 빌더가 알아서 채우도록 할 수도 있다.
- 반대로 이러한 빌더 패턴의 단점으로는 객체를 만들려면, 그에 앞서 빌더부터 만들어야 하기 때문에 빌더 생성 비용이 크진 않지만 성능에 민감한 상황에서는 문제가 될 수 있다.
- 또한 점층적 생성자 패턴보다 코드가 더 장황하다. 따라서 빌더 패턴은 매개변수가 4개 이상은 되어야 값어치를 한다.
<br>

## 정리
- 빌더는 점층적 생성자 패턴보다 클라이언트 코드를 읽고 쓰기가 훨씬 간결하고, 자바빈즈 패턴보다 훨씬 안전하다.
- 생성자나 정적 팩터리가 처리해야 할 매개변수가 많다면 빌더 패턴을 선택하는 것이 더 낫다.
- 하지만 성능에 민감한 상황이라면 빌더 생성 비용을 고려해볼 필요가 있다.
