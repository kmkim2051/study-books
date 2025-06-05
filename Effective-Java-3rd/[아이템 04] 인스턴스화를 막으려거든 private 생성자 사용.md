# [아이템 4] 인스턴스화를 막으려거든 private 생성자를 사용하라

단순히 정적 메서드와 정적 필드만을 담은 클래스 생성을 위함

- java.lang.Math, java.util.Arrays 등 기본 타입 값 또는 배열 관련 메서드
- java.util.Collections 처럼 특정 인터페이스를 구현하는 객체 생성 정적 메서드
- final 클래스 관련 메서드

**추상 클래스로 만드는 것으로는 인스턴스화를 강제할 수 없다.**

하위 클래스를 만들어 인스턴스화 하면 그만

### private 생성자를 추가하면 클래스의 인스턴스화를 막을 수 있다.

**예시**

```java
public class Util {
  private Util() {
    throw new AssertionError();
  }
}
```

꼭 Assertion 에러를 던질 필요는 없지만, 클래스 안에서 실수로라도 생성자를 호출하는 것을 막아준다.

### 상속을 불가능하게 하는 효과도 있다.

하위 클래스가 상위 클래스의 생성자에 접근할 길을 막아버린다.