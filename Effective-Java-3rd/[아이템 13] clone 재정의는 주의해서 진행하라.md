# [아이템 13] clone 재정의는 주의해서 진행하라

### **Cloneable**

복제해도 되는 클래스임을 명시하는 용도의 믹스인 인터페이스

아래의 여러 문제에도 불구하고 Cloneable 방식은 널리 쓰이고 있음

### **문제**

- clone 메서드는 Cloneable이 아닌 Object 에 protected 로 선언되어 있음
→ Cloneable 인터페이스 구현만으로는 외부에서 clone 메서드를 호출할 수 없음
- 리플렉션으로 가능하지만, 100% 성공 보장하지 않음
    - 해당 객체가 접근이 허용된 clone 메서드를 제공한다는 보장이 없음

### **Cloneable 인터페이스의 역할**

Object 의 protected 메서드인 **clone**의 동작 방식을 결정

- Marker interface 로서, Object.clone() 이 필드 단위 복사를 수행하도록 허용하는 ‘신호 역할’만 수행

Cloneable을 구현한 클래스의 인스턴스에서 clone 호출

→ 객체의 필드를 하나하나 복사한 객체를 반환

→ 그렇지 않은 클래스에서는 CloneNotSupportedException 던짐

<aside>
💡

인터페이스를 이례적으로 사용한 예이니 따라하지는 말자.

</aside>

### 실무에서의 Cloneable

clone 메서드를 public 으로 제공하며, 사용자는 복제가 제대로 이루어지기를 기대

- 위험하고 모순적인 메커니즘 = 생성자 호출 없이 객체 생성 가능

### **clone 메서드의 허술한 일반 규약**

객체의 복사본을 생성해 반환한다. ‘복사’의 정확한 뜻은 구현 클래스에 따라 다를 수 있으며, 일반적인 의도는 다음과 같다.

- 어떤 객체 x에 대해 **x.clone() ≠ x 는 참**이다.
- **x.clone().getClass() == x.getClass()** 도 참이다.
    - 위의 두 요구를 반드시 만족해야 하는 것은 아니다.
- **x.clone().equals(x)** 일반적으로 참이지만, 필수는 아니다.
- 관례상, 이 메서드가 반환하는 객체는 **super.clone**을 호출해 얻어야 한다.
    - 이 클래스와 (Object 제외) 상위 클래스가 이 관례를 따른다면 다음은 참이다.
    - **x.clone().getClass() == x.getClass()**
- 관례상, 반환 객체와 원본 객체는 독립적이어야 한다.
    - 복제된 객체의 내부 상태가 원본 객체와 **서로 참조를 공유하지 않도록 분리**되어 있어야 함
    - 이를 만족하려면 super.clone으로 얻은 객체의 필드 중 일부를 반환 전에 수정해야 할 수도 있다.

강제성이 없다는 점만 빼면 생성자 체이닝(생성자에서 super() 호출)과 유사한 메커니즘

clone 메서드가 super.clone이 아닌, **생성자를 호출해 얻은 인스턴스를 반환**해도 컴파일러는 정상 동작

→ 하위 클래스에서 super.clone을 호출하면 의도하지 않은 클래스의 인스턴스가 만들어져 에러 발생

→ clone을 재정의한 클래스가 final이라면 하위 클래스가 없으니 관례를 무시해도 안전하지만, Cloneable을 구현할 이유 또한 없음

### 제대로 동작하는 Cloneable 구현

먼저 super.clone 을 호출한다. (원본의 완벽한 복제본)

1. 모든 필드가 기본 타입 또는 불변 객체 참조라면 여기서 끝
2. 불변 클래스는 굳이 clone 메서드를 제공하지 않는 것이 좋다

가변 상태를 참조하지 않는 클래스용 clone 메서드

```java
@Override
pubilc PhoneNumber clone() {
  try {
    return (PhoneNumber) super.clone();
  } catch (CloneNotSupportedException e) { 
		// 일어날 수 없음
    // CloneNotSupportedException 은 사실 unchecked 였어야 했다고 함
    throw new AssertionError(); 
    
  }
}
```

- 메서드 동작을 위해 클래스 선언에 Cloneable 구현 선언을 추가
- Object.clone은 Object를 반환하지만, PhoneNumber::clone은 PhoneNumber 반환
    - 자바의  공변 반환 타이핑에 의한 것으로, 권장하는 방식
        
        [공변](%5B%E1%84%8B%E1%85%A1%E1%84%8B%E1%85%B5%E1%84%90%E1%85%A6%E1%86%B7%2013%5D%20clone%20%E1%84%8C%E1%85%A2%E1%84%8C%E1%85%A5%E1%86%BC%E1%84%8B%E1%85%B4%E1%84%82%E1%85%B3%E1%86%AB%20%E1%84%8C%E1%85%AE%E1%84%8B%E1%85%B4%E1%84%92%E1%85%A2%E1%84%89%E1%85%A5%20%E1%84%8C%E1%85%B5%E1%86%AB%E1%84%92%E1%85%A2%E1%86%BC%E1%84%92%E1%85%A1%E1%84%85%E1%85%A1%202084d82fc33b8069af5ff8fd0af09b0c/%E1%84%80%E1%85%A9%E1%86%BC%E1%84%87%E1%85%A7%E1%86%AB%202084d82fc33b80588b15d3e537bff22c.md)
        

### 가변 객체 참조 clone 문제

**Stack 예시**

```java
public class Stack {
  private Object[] elements;
  // ...
  public Stack() {
    this.elements = new Object[DEFAULT_INITIAL_CAPACITY];
  }
  // ...
}
```

clone 메서드가 단순히 super.clone 결과를 그대로 반환한다면?

→ 복사본 Stack 인스턴스는 원본 Stack 인스턴스와 동일한 내부 배열 참조 (불변 해침)

Stack 클래스의 하나뿐인 생성자를 호출하면 (새로운 객체) 이러한 상황은 일어나지 않음

**가변 상태를 참조하는 클래스용 clone**

```java
@Override
public Stack clone() {
  try {
    Stack result = (Stack) super.clone();
    result.elements = elements.clone();
    return result;
  }
  //...
```

- elements.clone 결과를 형변환할 필요는 없음
    - 배열의 clone은 런타임과 컴파일 타임 모두 원본 배열과 똑같은 배열 반환
- elements가 final 이라면? 작동하지 않음
    - Cloneable 아키텍처는 ‘가변 객체 참조 필드는 final로 선언’ 용법과 충돌

**해시테이블용 clone**

잘못된 clone 메서드 - 가변 상태를 공유

```java
@Override
public HashTable clone() {
  try { 
    HashTable result = (HashTable) super.clone();
    result.buckets = buckets.clone();
    return result;
  }
  // ...
```

복제본은 별도의 버킷 배열을 소유 (but) 원본과 같은 연결 리스트(next)를 참조

→ 각 버킷을 구성하는 연결 리스트를 복사해야 함

**복잡한 가변 상태를 갖는 클래스용 재귀적 clone 메서드**

```java
// 이 엔트리가 가리키는 연결 리스트를 재귀적으로 복사
Entry deepCopy() {
  return new Entry(
    key, 
    value, 
    (next == null) ? null : next.deepCopy()
  );
}
// ...
@Override
public HashTable clone() {
  try {
    HashTable result = (HashTable) super.clone();
    result.buckets = new Entry[buckets.length];
    for (int i = 0; i < buckets.length; i++) {
      if (buckets[i] != null) 
        result.buckets[i] = buckets[i].deepCopy();
    return result;
		// ...
```

버킷이 너무 길지 않다면 잘 작동하나, 리스트 원소 수만큼 스택 프레임을 소비하여 스택 오버플로우를 일으킬 위험 존재

→ deepCopy를 반복문으로 변경

```java
Entry deepCopy() {
  Entry result = new Entry(key, value, next);
    
  for (Entry p = result; p.next != null; p = p.next) {
    // p.next 를 deep copy
    p.next = new Entry(p.next.key, p.next.value, p.next.next);
  }
  return result;
}
```

**복잡한 가변 객체를 복제하는 마지막 방법**

1. 먼저 super.clone 을 호출하여 얻은 객체의 모든 필드를 초기 상태로 설정
2. 원본 객체의 상태를 다시 생성하는 **고수준 메서드** 호출
    1. 해시테이블 예시
    buckets 필드 초기화한 다음, 원본 테이블의 모든 key-value에 대해 복제본 테이블의 put(key, value)를 호출해 둘의 내용이 일치하도록 설정
    2. 고수준 API 는 간결하지만 저수준 구현에 비해 비교적 느리고, 필드 단위 객체 복사를 우회하므로 Cloneable 아키텍처와는 어울리지 않음
3. clone에서는 (생성자와 마찬가지로) 재정의될 수 있는 메서드를 호출하지 않아야 함
    1. 하위 클래스에서 재정의한 메서드 호출 시, 원본과 복제본의 상태가 달라질 가능성이 큼
    2. (2-a)의 put 메서드는 final 또는 private 이어야 한다.
4. public인 clone 메서드에서는 throws절을 없애야 사용하기 편하다
5. 상속용 클래스는 Cloneable을 구현해서는 안된다.
    1. protected + 예외 선언으로 구현을 하위 클래스에서 선택하도록 하기
    2. clone 메서드를 final 로 설정하고, 예외 던지기
6. Cloneable 구현한 thread-safe 클래스는 clone 메서드 역시 동기화해줘야 함
7. <요약>
    1. Cloneable 구현 클래스는 clone 재정의
    2. 접근 제한자는 public, 반환 타입은 클래스 자기 자신
    3. super.clone을 먼저 호출, 객체 참조는 내부 모두 깊은 복사
    4. 단, 일련번호나 고유 ID는 기본/불변 타입이라도 수정해줘야 함

### 복사 생성자와 복사 팩터리

이미 Cloneable을 구현한 클래스를 확장하는 경우가 아니라면, 대부분 나은 선택지

- 언어 모순적이고 위험한 객체 생성 메커니즘 사용하지 않음
- 엉성한 규약에 기대지 않고, final 용법과 충돌하지 않음
- 불필요한 예외를 던지지 않고, 형변환도 필요하지 않음
- 해당 클래스가 구현한 **‘인터페이스’타입의 인스턴스**를 받을 수 있음
    - 범용 컬렉션 구현체는 Collection 이나 Map 타입을 받는 생성자 제공
        
        ```java
        List<String> original = List.of("apple", "banana", "cherry");
        List<String> copy = new ArrayList<>(original);
        
        Map<String, Integer> original = Map.of("one", 1, "two", 2);
        Map<String, Integer> copy = new HashMap<>(original);
        ```
        

더 정확한 이름는 변환 생성자(conversion), 변환 팩터리

**복사 생성자**

자신과 같은 클래스의 인스턴스를 인수로 받는 생성자

- 서브클래싱을 고려하지 않는, 단일 클래스의 상태 복제에 사용

```java
public Yum(Yum yum) {...}
```

**복사 팩터리**

복사 생성자를 모방한 정적 팩터리

- 캐싱, 타입 다양성, 서브클래싱 등을 고려하거나 외부 노출 용도로 사용

```java
public static Yum newInstance(Yum yum) {...}
```

### 더 알아보기

**with 방식으로 일부 필드만 바꾼 새로운 인스턴스 반환 → 불변 객체 생성 시 용이**

record에서 구현한 예시

```java
public record Person(String name, int age) {

    public Person withName(String name) {
        return new Person(name, this.age);
    }

    public Person withAge(int age) {
        return new Person(this.name, age);
    }
}
// 사용 예시
Person p1 = new Person("Alice", 30);
Person p2 = p1.withAge(31);  // 이름은 그대로, 나이만 변경

System.out.println(p2); // Person[name=Alice, age=31]
```

lombok에서 제공하는 with

```java
public class Person {
    @With
    private final String name;
    @With
    private final int age;

    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }
}
// 자동 생성 메서드
public Person withName(String name) { ... }
public Person withAge(int age) { ... }

// 사용 예시
Person p1 = new Person("Alice", 30);
Person p2 = p1.withAge(31);

System.out.println(p2); // Person[name=Alice, age=31]
```