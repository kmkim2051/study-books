# [아이템 14] Comparable을 구현할지 고려하라

### Comparable 인터페이스의 compareTo

**Object의 equals와의 차이점** 

단순 동치성 비교 뿐 아니라 순서까지 비교, 제네릭

Comparable을 구현한 클래스의 인스턴스는 자연적인 순서가 있음을 의미

→ Arrays.sort(a) 와 같이 손쉽게 정렬 가능

검색, 극단값 계산, 자동 정렬 컬렉션 관리 또한 손쉽게 가능

**코드 예시**

```java
public class WordList {
  public static void main(String[] args) {
    // String이 Comparable을 구현한 덕분
    Set<String> s = new TreeSet<>();
    Collections.addAll(s, args);
    System.out.println(s);
  }
}
```

자바 플랫폼 라이브러리의 모든 값 클래스와 열거 타입이 Comparable을 구현함

→ 알파벳, 숫자, 연대와 같이 순서가 명확한 값 클래스는 **반드시** Comparable을 구현하자

### compareTo 메서드의 일반 규약

이 객체와 주어진 객체를 비교한다. 

- 주어진 객체보다 **작으면 음의 정수, 같으면 0, 크면 양의 정수**를 반환
- 비교할 수 없는 타입의 객체 → ClassCastException 발생

Comparable을 구현한 클래스는 모든 x, y, z에 대해

- x.compareTo(y) == -(y.compareTo(x))
    - x.compareTo(y) 는 y.compareTo(x) 가 예외를 던질 때에 한해 예외를 던져야 한다
    - **순서를 바꿔도 예상한 결과가 나와야 함**
- 추이성을 보장한다.
    - x.compareTo(y) > 0 && y.compareTo(z) > 0 → x.compareTo(z) > 0
- x.compareTo(y) == 0 이면, sgn(x.compareTo(z)) == sgn(y.compareTo(z))
    - **크기가 같은 객체들끼리는 어떤 객체와 비교해도 항상 결과가 같아야 함**
    - compare 결과가 0이다 → 동등한 객체 → 제 3의 객체와 비교해도 일관된 순서

**즉, compareTo로 수행하는 동치성 검사도 equals와 같이 반사성, 대칭성, 추이성을 충족해야 함**

- 필수는 아니지만 꼭 지켜야 하는 규칙 (equals와의 관계)
    - (x.compareTo(y) == 0) == x.equals(y)
    - compareTo와 equals의 동치성 테스트 결과가 일치하도록 구현하는게 좋다.
        - 정렬된 컬렉션에 넣었을 시, 정의된 동작과 결과가 달라질 수 있음
        - 예시
        
        ```java
        BigDecimal("1.0") 
        BigDecimal("1.00")
        
        equals로는 다르지만(다른 객체)    -> HashSet에서는 구분된 객체  -> size = 2
        compareTo로는 동일하다 (대소비교) -> TreeSet에서는 동일 객체   -> size = 1
        ```
        
        - [https://docs.oracle.com/javase/8/docs/api/java/util/TreeSet.html](https://docs.oracle.com/javase/8/docs/api/java/util/TreeSet.html)
            
            > but a `TreeSet` instance performs all element comparisons using its `compareTo` (or `compare`) method
            > 
    - 이 권고를 지키지 않는 모든 클래스는 그 사실을 명시해야 한다.

### **비교 대상의 타입에 따른 equals와 compareTo의 차이**

- equals 메서드는 모든 객체에 대해 전역 동치관계를 부여함
    - 클래스 간 비교를 허용하긴 하지만, 일반적으로 동일 타입 간에만 비교하긴 함
- compareTo는 타입이 다른 객체를 신경쓰지 않아도 됨
    - 단순히 ClassCastException을 던져도 되며, 대부분 그렇게 함
- 다른 타입 사이의 비교도 가능하지만, 보통 공통 인터페이스를 매개로 이루어짐

### **compareTo 규약을 지키지 못하면, 비교를 활용하는 클래스와 어울릴 수 없음**

- 정렬된 컬렉션
    - TreeSet, TreeMap
- 검색과 정렬 알고리즘 활용하는 유틸리티 클래스
    - Collections, Arrays

### **확장 클래스 문제와 우회법**

equals와 마찬가지로, **확장 클래스에서 새로운 값 컴포넌트가 추가**되면 compareTo 규약을 지킬 방법이 없다.

우회법 또한 내부 인스턴스를 반환하는 ‘뷰 메서드 (asPoint)’ 제공 방식으로 동일하다.

### 작성 요령

작성 요령은 equals와 비슷하다.

**Comparable 은 타입을 인수로 받는 제네릭 인터페이스**
→ compareTo 메서드의 인수 타입은 컴파일 타임에 결정
→ 타입이 잘못되면 컴파일이 안되므로 타입 체크 및 형변환할 필요 없음

**null을 인수로 넣어 호출하면 NPE를 던져야 함**

compareTo(null) 은 NPE를 던져야 한다.

- compareTo를 호출하는 메서드에서 @NotNull 등의 애노테이션으로 명시하는 것이 좋을 듯

실제로도 null의 멤버에 접근하는 순간 예외가 던져질 것이다.

**compareTo는 각 필드가 동치인지 비교하는게 아닌, 순서를 비교한다. (큰지, 작은지, 같은지)**

**객체 참조 필드는 compareTo 메서드를 재귀적으로 호출하여 비교**

**Comparable 미구현 또는 비표준 비교는 비교자(Comparator)를 대신 사용**

직접 만들거나, 자바 기본 제공을 사용

[코드] 객체 참조 필드가 하나뿐인 비교자

```java
public final class CaseInsensitiveString implements Comparable<CaseInsensitveString> {
  public int compareTo(CIS cis) {
    return String.CASE_INSENSITVE_ORDER.compare(s, cis.s);
  }
  // ...
```

```java
// String.java
public static final Comparator<String>CASE_INSENSITIVE_ORDER
= new CaseInsensitiveComparator();
```

### compareTo의 이전 방식

책의 2판에서 권장한 내용

- 정수 기본 타입 필드 비교는 관계 연산자(>, <) 사용
- 실수 기본 타입 필드 비교는 정적 메서드 (Type.compare) 사용

자바 7부터는 박싱된 기본 타입 클래스들에 새로 추가된 정적 메서드 compare 사용 추천

- 일관된 결과 반환(-1, 0, 1) , null-safe
- 관계 연산자는 더이상 추천하지 않음

### 핵심 필드가 여러 개인 클래스의 비교

클래스에 핵심 필드가 여러 개라면, 어느 것을 먼저 비교하느냐가 중요 → 핵심 필드부터

**비교 결과가 0이 아니라면, 즉시 결과를 반환하자**

### 자바 8에 도입된 메서드 체이닝 방식

Comparator 인터페이스가, 일련의 비교자 생성 메서드(comparator construction method) 를 이용,

**메서드 체이닝 방식으로 비교자를 생성 가능**

<aside>
💡

간결하지만, 약 10% 정도의 성능 저하 발생 가능 (저자 피셜)

</aside>

```java
Comparator<PhoneNumber> CP 
  = comparingInt((PhoneNumber pn) -> pn.areaCode) // 타입 정보 넘겨줘야 함
      .thenComparingInt(pn -> pn.prefix)
      .thenComparingInt(pn -> pn.lineNum);

public int compareTo(PhoneNumber pn) {
  return CP.compare(this, pn);
}      
```

클래스 초기화 할 때, 비교자 생성 메서드 2개를 이용한다.

- comparingInt
    - 객체 참조를 int 타입 키에 매핑하는 **키 추출 함수**를 인수로 받아, 
    그 키를 기준으로 순서를 정하는 **비교자를 반환**하는 **정적 메서드**

```java
 public static <T> Comparator<T> comparingInt(ToIntFunction<? super T> keyExtractor) {
   Objects.requireNonNull(keyExtractor);
   
   return (Comparator<T> & Serializable)
        (c1, c2) -> Integer.compare(
                      keyExtractor.applyAsInt(c1),
                      keyExtractor.applyAsInt(c2));
 }
```

- thenComparingInt
    - Comparator의 **인스턴스 메서드**로, int 키 추출자 함수를 입력받아 다시 비교자를 반환
        - 첫 번째 비교자 적용 후 **새로 추출한 키로 추가 비교를 수행**
    
    ```java
    default Comparator<T> thenComparingInt(ToIntFunction<? super T> keyExtractor) {
        return thenComparing(comparingInt(keyExtractor));
    }
    ```
    

**객체 참조용 메서드**

- comparing
    - 키 추출자 하나의 인수를 받는 메서드
    - 키 추출자, 비교자 두 개의 인수를 받는 메서드
    
    ```java
    // 키 추출자 하나의 인수를 받는 메서드
    // c1.compare(c2)
    public static <T, U extends Comparable<? super U>> Comparator<T> comparing(
            Function<? super T, ? extends U> keyExtractor)
    {
        Objects.requireNonNull(keyExtractor);
        return (Comparator<T> & Serializable)
            (c1, c2) -> keyExtractor.apply(c1).compareTo(keyExtractor.apply(c2));
    }
    
    // 키 추출자, 비교자 두 개의 인수를 받는 메서드
    // comparator.compare(c1, c2)
    public static <T, U> Comparator<T> comparing(
            Function<? super T, ? extends U> keyExtractor,
            Comparator<? super U> keyComparator)
    {
        Objects.requireNonNull(keyExtractor);
        Objects.requireNonNull(keyComparator);
        return (Comparator<T> & Serializable)
            (c1, c2) -> keyComparator.compare(keyExtractor.apply(c1),
                                              keyExtractor.apply(c2));
    }
    ```
    
- thenComparing
    - 키 추출자, 비교자, (키 추출자+비교자) 의 세 가지 종류

### 값의 차를 기준으로 구현한 compare

추이성을 위반하는 코드

- 정수 오버플로를 일으키거나, 부동소수점 계산 방식 오류 발생 가능

```java
public int compare(Object o1, Object o2) {
  return o1.hashCode() - o2.hashCode();
}
```

대신 정적 compare 메서드나 Comparator 생성 메서드를 활용하자

```java
// 방법 1.
public int compare(Object o1, Object o2) {
  return Integer.compare(o1.hashCode(), o2.hashCode());
}

// 방법 2.
Comparator.comparingInt(o -> o.hashCode());
```