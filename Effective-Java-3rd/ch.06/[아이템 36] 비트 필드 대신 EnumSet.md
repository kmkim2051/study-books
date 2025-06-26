# [아이템 36] 비트 필드 대신 EnumSet을 사용하라

열거한 값들이 주로 집합으로 사용될 경우, 예전에는 **2의 거듭제곱을 할당한 정수 열거 패턴 사용**

```java
int STYLE_BOLD = 1 << 0;
int STYLE_ITALIC = 1 << 1;
int STYLE_UNDERLINE = 1 << 2;

public void applyStyles(int styles) { ... }

text.applyStyles(STYLE_BOLD | STYLE_ITALIC);

// 0110 -> ITALIC + UNDERLINE
// 0101 -> ITALIC + BOLD
```

합집합, 교집합을 효율적으로 수행할 수 있지만, 정수 열거 상수의 단점뿐 아니라 다양한 문제

### 비트 필드 방식의 문제

- 값을 그대로 출력하면 정수보다 해석하기 어렵다.
- 비트 필드에 적용된 원소를 순회하기도 까다롭다.
- 최대 몇 비트가 필요한지 미리 예측하여 타입을 선택해야 한다.

### 더 나은 대안 EnumSet

Set을 완벽히 구현하며, 타입 안전하고, 다른 Set 구현체와도 사용 가능

- 내부는 비트 벡터로 구현되어, 특정 사이즈 이하에서는 높은 성능
- removeAll, retainAll 같은 대량 작업 또한 최적화

```java
public class Text {
  public enum Style { BOLD, ITALIC, UNDERLINE }
  
  // 어떤 Set을 넘겨도 되지만, EnumSet이 최적
  public void applyStyles(Set<Style> styles) {...}
}

text.applyStyles(EnumSet.of(Style.BOLD, Style.UNDERLIN));
```

applyStyles가 `EnumSet<Style>` 이 아닌 `Set<Style>` 을 받은 이유? **이왕이면 인터페이스로 받는게 범용적**