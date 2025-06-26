# [아이템 34] int 상수 대신 열거 타입을 사용하라

### 정수 열거 패턴의 한계

자바에서 열거 타입 지원하기 이전 코드 (정수 묶음)

```java
public static int APPLE_FUJI = 0;
public static int APPLE_PIPPIN = 1;

public static int ORANGE_NAVEL = 0;
public static int ORANGE_TEMPLE = 1;
```

**문제점**

- 타입 안전을 보장할 수 없고 표현력도 좋지 않다.
    - `ORANGE_NAVEL == APPLE_FUJI`
- 단순 상수 나열이라 데이터 변경 시 다시 컴파일 해야 한다.
- 문자열로 출력하기 까다롭다.

문자열 열거 패턴은 더 나쁨 → 하드코딩하기 쉬워지기 때문

### 열거 타입

```java
public enum Apple { FUJI, PIPPIN }
public enum ORAGNE { NAVEL, BLOOD }
```

**자바 열거 타입의 특징**

- 열거 타입 자체는 하나의 클래스이며, 상수 하나당 하나의 인스턴스 `public static final`
    - 생성자가 없으므로 인스턴스 통제, 싱글턴
- 컴파일타임 타입 안전성 제공
- 각자의 이름공간 존재 → 새로운 상수 추가 또는 순서 변경해도 재컴파일 필요 없음
- toString에 적합한 문자열 값 제공
- 임의의 **메서드, 필드 추가 및 인터페이스 구현** 가능

**데이터와 메서드를 갖는 열거 타입**

```java
public enum Planet {
  MERCURY(3.3, 2.4),
  VENUS(4.8, 6.0),
  NEPTUNE(1.0, 2.4);
  
  private final double mass;  // 질량
  private final double radius; // 반지름
  private final double surfaceGravity; // 표면중력
  
  private static final double G = 6.67; // 중력 상수
  
  public double surfaceWeight(double mass) {
    return mass * surfaceGravity; // F = ma;
  }
```

다음과 같이 행성별 지구의 무게를 계산하는 코드를 손쉽게 작성 가능

```java
double earthWeight = Double.parseDouble(args[0]);
double mass = earthWeight / Planet.Earth.surfaceGravity();
for (Planet p : Planet.values())
  sout("%s에서의 무게 %f", p, p.surfaceWeight(mass));
```

- 자신 안에 정의된 상수들의 값을 반환하는 `values()` 제공
- 상수 이름을 문자열로 반환하는 `toString()`

**타입에서 상수 하나를 제거하면?**

클라이언트에서는 컴파일 오류를 반환하므로 → 디버깅 쉬움 (정수 열거 패턴에서는 어려움)

**외부에 노출할 필요 없는 기능은 private, p-p로 구현하라 (아이템 15)**

**널리 쓰이는 열거 타입은 톱레벨 클래스, 특정 클래스에서만 쓰인다면 멤버 클래스로 만든다. (아이템 24)**

### 상수마다 동작이 달라져야 하는 경우

단순히 데이터와 연결되는 걸 넘어 상수마다 동작이 달라져야 하는 경우

```java
public enum Operation {
  PLUS, MINUS, TIMES, DIVIDE;
  
  public double apply(double x, double y) {
    switch(this) {
      case PLUS: return x + y;
      //...
    }
  }
```

간단하지만 새로운 상수 추가되면 코드도 변경되어야 함 → OCP x

→ 추상 메서드를 선언하고 각 상수에서 직접 재정의하도록 하자! **상수별 메서드 구현** 이라고 함

```java
public enum Operation {
  PLUS("+") {
    public double apply(double x, double y) { return x + y; }
  }, 
  MINUX("-") { ... }
  // ...
  
  private final String symbol;
  Operation(String symbol) { this.symbol = symbol; }
  
  @Override
  public String toString() { return symbol;}
  public abstract double apply(double x, double y);
}
```

열거 타입에는 `상수 이름 → 열거 상수` 를 해주는 `valueOf(String)` 메서드가 자동 생성

열거타입 toString을 재정의하려면, fromString (toString의 반대) 메서드도 함께 제공하는걸 고려해보자.

```java
private static final Map<String, Operation> strToEnum
  = Stream.of(values()).collect(toMap(Object::toString, e -> e));
  
public static Optinal<Operation> fromString(String symbol) {
  return Optional.ofNullable(strToEnum.get(symbol));
}
```

- 열거 타입 상수는 생성자에서 자신의 인스턴스를 맵에 추가할 수 없다. (컴파일 오류)
    - 열거 타입의 정적 필드 중 생성자에서 접근할 수 있는 것은 상수 변수뿐이다. (아이템 24)
    - 정적필드가 초기화되기 전이므로, 자기 자신을 추가하지 못하게 하는 제약이 필요하기 때문
    - 열거 타입 생성자에서 같은 열거 타입의 다른 상수에도 접근 불가능 `public static final`

### 상수별 메서드 구현의 단점

열거 타입 상수끼리 코드를 공유하기 어렵다

**코드 예시**

급여명세서 요일 & 일당 계산 코드

```java
enum PayrollDay {
  MONDAY, TUESDAY, ..., SUNDAY;
  
  private static final MINS_PER_SHIFT = 8 * 60;
  
  int pay(int minutesWorked, int payRate) {
    int basePay = minutesWorked * payRate;
    
    int overTimePay;
    switch(this) {
      case SATUREDAY, SUNDAY:
        overTimePay = basePay / 2;
      default:
        overTimePay = minutesWorked <= MINS_PER_SHIFT ? 
          0 : (minutesWorked - MINS_PER_SHIFT) * payRate / 2;
    }
    return basePay + overTimePay;
  }
```

**간결하지만 위험한 코드**

- 휴가와 같은 새로운 값 추가 시, case 문에도 똑같이 추가해줘야 함

**상수별 메서드 구현 방법**

1. 잔업수당 계산 코드를 모든 상수에 중복해서 넣는다.
2. 계산 코드를 평일용/주말용으로 나눠 각각 help method 작성, 각 상수가 적절히 호출

→ 두 방식 모두 가독성이 떨어지고 오류 발생 가능성 높음

**상수를 추가할 때 전략을 선택하도록 만들기**

잔업수당 계산을 private 중첩 열거 타입으로 옮기고, PayrollDay 생성자에서 선택하도록 구현

→ 복잡하지만 안전하고 유연

```java
enum PayrollDay {
  MONDAY(WEEKDAY), ..., SUNDAY(WEEKEND);
  
  private final PayType payType;
  
  int pay(int minutesWorked, int payRate) {
    // 실제 동작은 위임
    return payType.pay(miniutesWorked, payRate);
  }
  
  // ------ 전략 열거 타입 ------
  enum PayType {
    WEEKDAY {
      int overTypePay(int minsWorked, int payRate) {
        // 평일 로직 구현
      }
    },
    WEEKEND {
      int overTimePay(int minsWorked, int payRate) {
        // 주말 로직 구현
      }
    };
    // 각자 구현할 메서드는 abstract로
    abstract int overTimePay(int mins, int payRate);
    
    int pay(int minsWorked, int payRate) {
      int basePay = minsWorked * payRate;
      return basePay + overTimePay(minsWorked, payRate);
    }
  }
}
```

**switch문은 상수별 동작 구현에 적합하지 않다!**

기존 열거 타입에 상수별 동작 혼합하는 경우에는 좋은 선택일 수 있음

```java
switch(op) {
  case PLUS: return Opeartion.MINUS;
  case MINUS: return Operation.PLUS;
  // ...
}
```

추가하려는 메서드가 **의미상 열거 타입에 속하지 않는다면**, 직접 만든 열거 타입이라도 이 방식을 적용하는게 좋다.

### 그래서 언제 써야 하는지?

필요한 원소를 **컴파일타임**에 다 알 수 있는 상수 집합이라면 항상 열거 타입을 사용하자