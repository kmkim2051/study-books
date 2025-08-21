# [아이템 88] readObject 메서드는 방어적으로

가변 Date 필드를 이용하는 불변 클래스

```java
public final class Period {
  private final Date start, end;
  
  public Period(Date start, Date end) {
    this.start = new Date(start.getTime());
    this.end = new Date(end.getTime());
    // ... 유효성 검사
  }
  
  public Date start() { return new Date(start.getTime()); }
  // ...end()
  // ...toString
}
```

### Period 클래스 직렬화

단순 Serialzable 추가로는 충분하지 않음 → 불변식 보장할 수 없다.

**문제 - `readObject` 는 일종의 또다른 public 생성자**

- 매개변수로 바이트 스트림을 받는 생성자라고 볼 수 있음 → **유효성 검사, 방어적 복사** 필요
- 악의적인 의도로 생성한 바이트 스트림을 건넬 경우 문제 발생

```java
public class BogusPeriod {
  private static final byte[] form = {
    //... 잘못된 바이트 조합
  };
  
  public static void main(String[] args) {
    Period p = (Period)deserialize(form);
    sout(p);
  }
  
  static Object deserialize(byte[] sf) {
    try {
      return new ObjectInputStream(new ByteArrayInputStream(sf)).readObject();
    } // ...
  }
}
```

- 바이트 스트림 포맷은 [자바 객체 직렬화 명세 Serialization, 6] 참고

**이 문제를 고치려면?**

defaultReadObject 호출 후 유효성 검사 해야 함 → 문제 있다면 InvalidObjectException 

```java
private void readObject(ObjectInputStream s) throws IOException, ClassNotFoundException {
  s.defaultReadObject();
  
  // 유효성 검사 
  if (start.compareTo(end) > 0)
	  // ...
```

**여전히 존재하는 문제**

정상 인스턴스의 바이트 스트림 끝에 **private Date로의 참조를 추가**하면, 가변 인스턴스 생성 가능

```java
// 가변 공격의 예
public class MutablePeriod {
  public final Period;
  public final Date start;
  public final Date end;
  
  public MutablePeriod {
    try {
      ByteArrayOutputStream bos = ...
      ObjectOutputStream out = ...
      // 정상 Period 인스턴스 직렬화
      out.writeObject(new Period(new Date(), new Date()));
      
      // 악의적인 이전 객체 참조 추가 (자바 객체 직렬화 명세 참고)
      byte[] ref = { 0x71, 0, 0x7e, 0, 5}; // 참조 #5
      bos.write(ref); // 시작 필드
      ref[4] = 4; // 참조 4
      bos.write(ref); // 종료 필드
      
      // Period 역직렬화 후 Date 참조 훔침
      ObjectInputStream in = new ObjectInputStream(new ByteArrayInputStream(bos.toByteArray()));
      period = (Period)in.readObject();
      start = (Date)in.readObject();
      end = (Date)in.readObject();
      // ...
 }
 
 // 공격 모습
 public static void main(String[] args) {
   MutablePeriod mp = new MutablePeriod();
   Period p = mp.period;
   Date pEnd = mp.end;
   
   pEnd.setYear(78); // 불변식 깨진 데이터
```

문제의 근원 - Period의 readObject 메서드가 방어적 복사를 충분히 하지 않음

**→ 객체 역직렬화할 때 클라이언트가 소유해선 안되는 객체 참조를 갖는 필드를 모두 방어적 복사해야 한다**

(불변 클래스 안의 모든 private 가변 요소)

**방어적 복사와 유효성 검사를 모두 수행하는 readObject 메서드**

```java
private void readObject(ObjectInputStream s) throws IOException, ClassNotFoundException {
  s.defaultReadObject();
  
  // 가변 요소 방어적 복사
  start = new Date(start.getTime());
  end = new Date(end.getTime());
  
  // 불변식 검사
  if (start.compareTo(end) > 0)
   // throw...
```

- 방어적 복사를 유효성 검사보다 앞서 수행
- Date의 `clone` 메서드는 사용하지 않는다.
→ 두 조치 모두 공격 방지에 필요
- start, end 필드의 final 한정자 제거해야함
    - 아쉽지만 공격을 피하는게 더 큰 장점

### **기본 readObject를 써도 되는 상황?**

transient 필드를 제외한 모든 필드의 값을 매개변수로 받아,

**유효성 검사 없이 필드에 대입하는 public 생성자를 추가해도 괜찮은 경우**

- 아니라면
    - 커스텀 readObject 메서드
    - 직렬화 프록시 패턴

**readObject와 생성자의 또 다른 공통점 (final 아닌 직렬화 클래스의 경우)**

재정의 가능 메서드를 직/간접적으로든 호출해서는 안된다.

→ 하위 클래스의 상태가 완전히 역직렬화되기 전에 하위 클래스에 재정의된 메서드가 호출된다.

### 정리

readObject 작성 시, public 생성자만큼 신중하게 작성하자

- private 객체 참조 필드는 방어적 복사
- 불변식 어긋나면 InvalidObjectException
- 역직렬화 후 그래프 전체 유효성 검사 필요 시 ObjectInputValidation 인터페이스 사용
    
    ```java
    public interface ObjectInputValidation {
        public void validateObject() throws InvalidObjectException;
    }
    
    // 역직렬화 시 자동 호출되는 메서드
    private void readObject(ObjectInputStream ois) 
            throws IOException, ClassNotFoundException {
        ois.defaultReadObject();
        
        // 검증을 위해 등록
        ois.registerValidation(this, 0); // 우선순위 0
    }
    
    // 검증 로직
    @Override
    public void validateObject() throws InvalidObjectException {
        if (name == null || name.trim().isEmpty()) {
            throw new InvalidObjectException("이름이 null이거나 비어있습니다");
        }
        if (age < 0 || age > 200) {
            throw new InvalidObjectException("나이가 유효하지 않습니다: " + age);
        }
    }
    ```
    
- 재정의 가능 메서드는 호출하지 말자