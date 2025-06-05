# [아이템 10] equals는 일반 규약을 지켜 재정의하라

equals는 잘못 재정의하면 끔찍한 결과를 초래한다. 다음 상황 중 하나에 해당한다면 재정의하지 않는다.

### equals를 재정의하지 않는 경우

**각 인스턴스가 본질적으로 고유하다.**

값이 아닌 동작하는 개체를 표현하는 클래스

*Thread* 가 좋은 예로, Object의 equals 메서드는 이러한 클래스에 딱 맞게 구현됨

**인스턴스의 ‘논리적 동치성(logical equality)’을 검사할 일이 없다.**

*regex.Pattern* equals를 재정의해서 두 **Pattern의 인스턴스가 같은 정규표현식을 나타내는지** 검사하는 방법

설계자가 불필요하다고 판단했다면 Object의 기본 equals로 해결된다. **

**상위 클래스에서 재정의한 equals가 하위 클래스에도 딱 들어맞는다.**

대부분의 Set 구현체는 AbstractSet이 구현한 equals를, List는 AbstractList로부터, Map은 AbstractMap으로부터 상속받아 그대로 쓴다.

**클래스가 private이거나 package-private이고, equals 메서드를 호출할 일이 없다.**

만에 하나라도 위험을 회피하고 싶으면 equals 재정의하여 에러를 던지도록 구현

### equals를 재정의해야 하는 경우

객체 식별성(물리적)이 아니라 논리적 동치성을 확인해야 하는데, 상위 클래스의 equals가 논리적 동치성을 비교하지 못하는 경우

- 주로 값 클래스가 여기 해당 (Integer, String)

즉, 객체가 같은지가 아니라 **값이 같은지**가 중요한 경우

equals가 논리적 동치성을 비교하도록 재정의하면, 값 비교 뿐 아니라 Map 및 Set의 원소로 사용할 수 있다.

- 다만 싱글턴을 보장하는 인스턴스 통제 클래스라면 equals를 재정의하지 않아도 된다.

### equals 를 재정의할 때 따라야 하는 일반 규약

equals 메서드는 동치관계(equivalence relation)를 구현하며, 다음을 만족한다.

<aside>
💡

정리 편의를 위해 아래 나오는 x, y, z 는 모두 null이 아닌 모든 경우를 의미하고, 

equals 메서드는 = 으로 간소화하였음

</aside>

**반사성(reflexibity)** 

- x = x
- 자기 자신과 같아야 한다.

**대칭성(symmetry)** 

- x = y → y = x
- 두 객체는 서로에 대한 동치 여부에 똑같이 답해야 한다.
- 대소문자_무시_스트링 ↔ 일반_스트링
- List::contains 와 같은 메서드에서 어떤 결과가 나올 지 알 수 없음

**추이성(transitivity)**

 x = y , y = z →  x = z

**일관성(consistency)**

- x = y 반복 호출 시, 항상 true 또는 항상 false
- 두 객체가 같다면 (어느 하나 이상 수정되지 않는 한) 영원히 같아야 한다.
    - 불변이든 가변이든, equals의 판단에 신뢰할 수 없는 자원이 끼어들면 안된다.
    - 외부 상황에 따라 유동적으로 바뀌는 URL의 IP 주소 등이 해당

**Not Null**

- x = null 은 false
- 모든 객체는 null과 같지 않아야 한다.
    - 명시적 검사보다는, instanceof 묵시적 검사가 낫다.

### **동치관계란?**

- 집합을 서로 같은 원소들로 이루어진 부분집합으로 나누는 연산
    - "같다"고 여길 수 있는 기준을 세워서, 같은 것끼리 묶는 것
- 이 집합을 동치류(equivalence class; 동치 클래스) 라고 함
    - 같다고 판단된 객체들끼리 만든 하나의 묶음
    
    예를 들어 `equals()` 기준으로 아래 3개의 객체가 모두 같다면:
    
    ```java
    // Student(이름, 학번)
    // 학번을 기준으로 equals 재정의
    Student a = new Student("A", 1);
    Student b = new Student("B", 1);
    Student c = new Student("C", 1);
    ```
    
    - 이 셋은 모두 하나의 "동치류(equivalence class)"에 속해요.
    - `Objects.equals(a, b)`도 `true`, `Objects.equals(b, c)`도 `true`
- 유효한 equals 메서드는 모든 원소가 같은 동치류에 속한 어떤 원소와도 서로 교환할 수 있어야 함

### 예시로 알아보는 equals 규약 위배 (Point)

2차원에서의 점을 표현하는 Point, 색상이 추가된 ColorPoint

```java
public class Point {
  int x, y;
  
  @Override
  public boolean equals(Object o) {
    if (!(o instanceof Point))
      return false;
    Point p = (Point)o;
    return (p.x == x) && (p.y == y);
  }
  // ...
}
```

```java
// 확장된 Point
public class ColorPoint extends Point {
  Color color;
  // ...
}
```

equals 를 재정의하지 않는다면, 색상 정보를 무시한 채 비교를 수행

**대칭성을 위반하는 잘못된 코드**

```java
// 대칭성을 위반하는 잘못된 코드!
@Override
public boolean equals(Object o) {
  if (!(o instanceof ColorPoint))
    return false;
  return super.equals(o) && ((ColorPoint) o).color == color;
}
```

- p.equals(cp) == true
    - 색을 무시
- cp.equals(p) == false

**추이성을 위반하는 잘못된 코드**

```java
@Override
public boolean equals(Object o) {
  if (!(o instanceof Point))
    return false;
    
  // 일반 Point면 색상을 무시하고 비교 -> 다른 제3의 Point가 생기면 o.equals 무한루프
  if (!(o instanceof ColorPoint))
    return o.equals(this);
    
  return super.equals(o) && ((ColorPoint) o).color == color;
}
```

대칭성은 지켜주지만, 추이성은 깨버린다.

```java
cp1 = ColorPoint(1, 2, RED)
p = Point(1, 2)
cp2 = ColorPoint(1, 2, BLUE)
```

| 비교식 | 결과 | 이유 |
| --- | --- | --- |
| cp1.equals(p) | true | 색상 무시하고 비교 |
| p.equals(cp2) | true | 색상 무시하고 비교 |
| cp1.equals(cp2) | false | 일반 Point와 비교시 무시했던 색상이 비교됨 |

### 그렇다면 해법은?

이 현상은 객체 지향 언어의 동치관계에서 나타나는 근본적인 문제

**구체 클래스를 확장해 새로운 값을 추가하면서 equals 규약을 만족시킬 방법은 존재하지 않는다.**

**리스코프 치환 원칙 위배**

```java
@Override
public boolean equals(Object o) {
  if (o == null || o.getClass() != getClass())
    return false;
  Point p = (Point) o;
  return p.x == x && p.y == y;
}
```

getClass() 방식은 같은 구현 클래스의 객체와 비교할 때만 true를 반환한다. 

하지만 클래스가 다르면 x, y값과 무관하게 false를 반환하므로, **Point의 하위 클래스가 Point의 역할로서 비교되어야 할 때 리스코프 치환 원칙을 위배한다.**

**우회 방법 - 컴포지션**

Point를 상속하는 대신, private 필드로 두고 일반 Point를 반환하는 view method를 public으로 추가

```java
public class ColorPoint {
  private final Point;
  private final Color;
  // ...
  public Point asPoint() {
    return point;
  }
  
  @Override
  public boolean equals(Object o) {
    if (!(o instanceof ColorPoint))
      return false;
    ColorPoint cp = (ColorPoint) o;
    return cp.point.equals(point) && cp.color.equals(color);
    // ...
}
  
```

<aside>
💡

java.sql.Timestamp 는 java.util.Date 를 확장 후 nanaseconds를 추가하였다.

→ 대칭성 위배. 컬렉션에 섞어 쓰면 문제 발생 가능

</aside>

### 양질의 equals 구현 방법

1. == 연산자를 사용해, 입력이 자기 자신의 참조인지 확인한다.
    1. 이는 단순한 성능 최적화 용도
2. instanceof 연산자로 입력이 올바른 타입인지 확인한다. 그렇지 않다면 false를 반환한다.
    1. 올바른 타입은 해당 클래스인 것이 보통, 가끔 특정 구현 인터페이스가 될 수 있다.
    2. 어떤 인터페이스는 자신을 구현한 서로 다른 클래스끼리도 비교할 수 있도록 수정하기도 한다.
        1. 이러한 인터페이스라면 equal에서 해당 인터페이스를 사용해야 한다. (Set, List, Map 등)
3. 입력을 올바른 타입으로 형변환한다.
4. 입력 객체와 자기 자신의 대응되는 핵심 필드들이 모두 일치하는지 하나씩 확인한다.

```java
@Override
public boolean equals(Object o) {
  // 1
  if (this == o) {
    return true;
  }
  // 2
  if (!(o instanceof Point))
    return false;
  // 3
  Point p = (Point) o;
  // 4
  return p.x == x && p.y == y;
}
```

| 필드 | 비교 방법 | 비고 |
| --- | --- | --- |
| 기본 타입
(float, double 제외) | == |  |
| 참조 타입 | 각각의 equals |  |
| float, double | Float.compare(f1, f2)

Double.compare(d1, d2) | Float.NaN, -0.0f 등
특수한 부동소수 값을 다루기 위함

.equals는 오토박싱을 수반할 수 있어 성능상 이슈 |
| null 을 정상으로 취급하는 필드 | Objects.equals()로
NPE 예방 |  |
| 비교 복잡한 클래스 | 표준형을 저장하여 
표준형끼리 비교 | 불변 클래스에 효과적
가변 객체라면 값이 바뀔 때 마다 표준형을 갱신해야 함 |
- 필드 비교 순서가 equals 성능을 좌우하기도 함 → 가능성이 크거나 비용이 싼 필드를 먼저 비교
- 동기화용 lock 필드 같이 논리 상태가 의미 없는 필드는 비교하지 않음
- 파생 필드가 객체 전체의 상태를 대표하는 경우, 모든 필드를 비교할 필요 없이 캐시 값만 비교

**equals 를 다 구현했다면 대칭성, 추이성, 일관성을 자문해보자. (+단위테스트)**

전형적인 equals 의 예

```java
public final class PhoneNumber {
  private final short areaCode, prefix, lineNum;
  // ...
  private static final rangeCheck(int val, int max, String arg) {
    // ...
  }
  @Override
  public boolean equals(Object o) {
    if (o == this)
      return true;
    if (!(o instanceof PhoneNumber))
      return false;
    PhoneNumber pn = (PhoneNumber) o;
    return pn.lineNum == lineNum && pn.prefix == prefix && pn.areaCode == areaCode;
  }
  // ...
}    
```

### 마지막 주의사항

- equals 를 재정의 할 땐 **hashCode** 도 반드시 재정의하자.
- 너무 복잡하게 해결하지 말자.
    - 필드들의 동치성만 검사해도 어렵지 않게 규약 준수 가능
    - 별칭은 굳이 비교하지 않는다. (예) File의 경우 Symbolic link
- Object 외 타입을 매개변수로 받는 equals는 선언하지 말자.
    - overriding가 아니라 overloading이 되어, 하위 클래스에서 @Override 시 긍정 오류