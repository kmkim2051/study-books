# [아이템 35] ordinal 대신 인스턴스 필드를 사용하라

모든 열거 타입은 열거 타입에서 위치를 반환하는 `ordinal` 메서드를 제공한다. (0, 1, 2, …)

정숫값이 필요하면 ordinal을 사용하고 싶은 유혹 → 절대 쓰지 말자

```java
public enum Ensenble {
  SOLO, DUET, TRIO, ...
  public int numOfMusician() { return ordnial() + 1; }
}
```

동작은 하지만 유지보수하기 아주 끔찍한 코드

→ octec(8) 말고 double-quart(8)을 추가하고 싶다면? 

→ 중간에 숫자 하나를 건너뛰고 새로운 값을 추가하고 싶다면?

### 해결법

ordinal을 쓰지 말고, **인스턴스 필드**로 저장하자

```java
public enum Ensenble {
  SOLO(1), DUET(2), TRIO(3), ..., OCTET(8), DOUBLE_QUART(8), ...;
  
  private final int num;
  public int numOfMusician() { return num }
}
```

> “대부분 프로그래머는 이 메서드를 쓸 일이 없다. EnumSet, EnumMap 같은 범용 자료구조를 위한 설계”
>