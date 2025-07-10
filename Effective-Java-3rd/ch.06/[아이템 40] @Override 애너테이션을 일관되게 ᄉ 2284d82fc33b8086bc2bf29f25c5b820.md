# [아이템 40] @Override 애너테이션을 일관되게 사용하라

### @Override

- 자바의 기본 제공 애너테이션 중, 보통의 프로그래머에게 가장 중요한 애너테이션
- 메서드 선언에만 달 수 있으며, 상위 타입의 메서드를 재정의했음을 의미
- 일관되게 사용하면 여러 악명 높은 버그들을 예방

**코드 예시**

```java
public class Bigram {
  private final char first;
  private final char second;
  
  // constructor
  public boolean eqauls(Bigram b) {
    return b.first == first && b.second == second;
  }
  public int hashCode() {
    return 31 * first + second;
  }
  
  public static void main(String[] args) {
    Set<Bigram> s = new HashSet<>();
    for (int i = 0; i < 10; i++)
      for (char ch = 'a'; ch <= 'z'; ch++)
        s.add(new Bigram(ch, ch));
    
    sout(s.size()); // 예상: 26, 실제: 260
  }
```

equals와 hashCode를 **재정의** 가 아닌 **다중정의** 를 해서 발생하는 버그

`@Override` 애너테이션을 추가하면, 컴파일 오류가 명시적으로 발생하고 수정할 수 있다.

```java
@Override
public boolean equals(Object o) { // paramter는 Object 타입
  if (!(o isntanceof Bigram))
    return false;
  Bigram b = (Bigram) o;
  return b.first == first && b.second == second;
}
```

### 결론

상위 클래스의 메서드를 재정의하려는 모든 메서드에 `@Override` 애너테이션을 달자.

- 예외는 단 하나: 추상 메서드 재정의하는 경우는 굳이 달지 안아도 된다.

IDE는 일관되게 사용하도록 부추기기도 하는데, 애너테이션 달지 않은 메서드가 실제로 재정의 하는 경우도 캐치

클래스뿐 아니라 인터페이스의 메서드를 재정의할 때도 사용

추상 클래스나 인터페이스에서는 상위 메서드를 재정의하는 모든 메서드에 애너테이션을 다는 게 좋다.

**예시**

Set 인터페이스는 Collection 인터페이스를 확장했지만 새로 추가한 메서드는 없다.

따라서 모든 메서드 선언에 `@Override` 애너테이션을 달아 실수로 추가한 메서드가 없음을 보장