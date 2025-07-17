# [아이템 56] 공개된 API 요소에는 항상 문서화 주석을 작성하라

전통적으로 API 문서는 코드가 변경되면 항상 같이 수정해줘야 함

- 자바에서는 자바독(Javadoc)이라는 유틸리티가 이를 도와줌

### 공개된 요소에는 문서화 주석을 달자

- API를 올바르게 문서화하려면 **공개된 모든 클래스, 인터페이스, 메서드, 필드 선언**에 문서화 주석 달아야 함
    - 직렬화 가능 클래스라면 직렬화 형태 정보 포함
    - 문서화 주석이 없다면, 자바독도 단순 선언의 나열, 쓰기도 헷갈려서 오류 원인
- 기본 생성자는 문서화 주석 불가 → 공개 클래스에서는 사용하지 말자
- 유지보수를 생각한다면, 단순하게라도 비공개 요소에도 문서화 주석 추가

### 메서드용 문서화 주석

메서드용에는 해당 메서드와 **클라이언트와의** **규약**을 명료하게 기술함

- 상속용 클래스의 메서드가 아니라면, **how가 아니라 what을 기술**해야 함
- 호출하기 위한 **전제조건**, 수행 후 만족해야 하는 **사후조건** 나열해야 함
- 전제조건은 `@throws` 태그로 비검사 예외 선언하여 암묵적 기술
    - 비검사 예외 하나가 전제조건 하나와 연결
- `@param` 태그를 이용, 영향받을 매개변수에 기술 가능

부작용도 문서화해야 함

- 사후조건으로 명확히 나타나지는 않지만, 시스템 상태에 변화
    - 백그라운드 스레드를 시작시키는 메서드 등

메서드 계약 기술하려면

- 모든 매개변수에 `@param`
- void 아니라면 `@return`  (생략 가능)
- 발생 가능한 모든 예외에 `@throws`

관례상 `@param` 태그와 `@return` 태그 설명은 마침표 없이 명사구를 사용

- 드물게 산술 표현식도 사용
- 한글 작성 시 종결 어미로 끝날 경우 마침표 사용이 일관돼 보임

```java
/**
* Returns the element at the specified position in this list.
* 
* <p>This method is <i>not</i> .... 
*
* @param index index of element to return;
* @return the element at the specified position in this list
* @throws IndexOutOfBoundsException if the index is out of range
*         ({@code index < 0 || index >= this.size()})
*/
E get(int index);
```

- 문서화 주석에 HTML 태그 사용 → 자바독 유틸리티는 이를 HTML로 변환하므로, 최종 문서에 반영
- `{@code}` 태그
    - 태그로 감싼 내용을 코드용 폰트로 렌더링
    - 태그 내부 HTML 요소나 다른 자바독 태그 무시
    - 여러 줄 코드를 넣으려면 `<pre>{@code ...}</pre>`
- “this” list
this 는 호춛뢴 메서드가 자리하는 객체를 가리킴

### 상속용 클래스

자기 사용 패턴에 대해서 문서에 남겨야 함 → 재정의 하는 방법 명시

- 자바 8, `@implSpec` 태그로 문서화
- 일반적인 문서화 주석: 메서드와 클라이언트 사이 계약 명시
- `@implSpec` : 메서드와 하위 클래스 사이 계약 명시 → 메서드 상속 또는 super 키워드 사용 시

```java
/**
* Returns true if this collection is empty.
*
* @implSpec
* This implementation returns {@code this.size() == 0}
* // ...
*/
public boolean isEmpty() { ... }
```

**API 설명에 HTML 메타문자 넣기**

`<, >, &` 등의 메타 문자 넣으려면? `{@literal}` 태그로 감싼다.

**각 문서화 주석의 첫 번째 문장은 해당 요소의 요약 설명으로 간주된다.**

`Returns the element at the specified position in this list.` 가 이에 해당

요약 설명은 반드시 대상의 기능을 고유하게 기술해야 한다.

→ 한 클래스 내 요약 설명이 똑같은 멤버가 있어서는 안된다.

**마침표에 주의**

첫 문장의 마침표가 나오는 부분까지만 요약 설명이 된다.

→ `{@Literal}` 로 감싸주자.

자바 10부터는 `{@summary ...}` 로 깔끔하게 처리 가능

**요약 설명이란 문서화 주석의 첫 ‘문장’이라고 하면 부자연스럽다.** 

- **메서드와 생성자의 경우 주어가 생략된 동사구여야 한다.**
한글은 주어 생략이 자연스러우므로 문장이라고 해도 자연스러움
    
    ```java
    ArrayList(int initCapacity): Constructs an empty list with the ...
    ```
    
- **클래스, 인터페이스, 필드의 경우 명사절이어야 한다.**
    
    ```java
    Instant: An instantaneous point on the time-line.
    Maht.PI: The double value ~...
    ```
    

**자바 9부터, 자바독 HTML 문서에 검색(색인) 기능이 추가**

`{@index}` 태그를 사용해 중요한 용어를 색인화 할 수 있다.

`This method complies with the {@index IEEE 754} standard.`

### **문서화 주석에서 제네릭, 열거 타입, 애너테이션은 특별히 주의**

제네릭 타입 또는 제네릭 메서드 문서화 시, 모든 타입 매개변수에 주석을 달아야 한다.

```java
/**
* @param <K> 이 맵이 관리하는 키의 타입
* @param <V> 매핑된 값의 타입
*/
public interface Map<K, V> {...}
```

열거 타입 문서화 시 상수들에도 주석을 달아야 한다.

```java
public enum OrchestraSection {
  /* Woodwinds, such as flute, clarinet, and oboe. */
  WOODWIND,
  /* Brass instruments, such as hrn, and trumpet */
  BRASS,
  // ...
}
```

 

애너테이션 타입 문서화 시 멤버들에도 모두 주석을 달아야 한다.

- 애너테이션 자체도 물론이다.
- 필드 설명은 명사구로, 애너테이션 타입 설명은 의미를 설명하는 동사구

```java
/*
* 이 애너테이션이 달리 메서드는 명시한 예외를 던져야만 성공하는
* 테스트 메서드임을 나타냄
*/
@Retention(RententionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface ExceptionTest {
  /**
  * 이 애너테이션을 단 테스트 메서드가 성공하려면 던져야 하는 예외.
  * (이 클래스의 하위 타입 예외는 모두 허용)
  */
  Class<? extends Throwable> value();
}
```

패키지를 설명하는 문서화 주석은 `package-info.java` 파일에 작성한다.

- 이 파일은 패키지 선언을 반드시 포함해야 하며, 관련 애너테이션을 필수로 포함할 수 있다.
- 모듈 시스템(자바 9)도 이와 비슷하다. `module-info.java`

### 스레드 안전 수준을 반드시 API 설명에 포함해야 한다.

클래스 혹은 정적 메서드가 스레드 안전이든 아니든, 스레드 안전 수준을 반드시 API 명세에 포함해야 한다.

- 직렬화 가능 클래스라면 직렬화 형태도 API 설명에 기술해야 한다.

**자바독은 메서드 주석을 ‘상속’시킬 수 있다.**

문서화 주석이 없는 API 요소 발견 → 자바독이 가까운 문서화 주석을 찾아준다.

- 상위 클래스보다 인터페이스를 먼저 찾는다. (클래스는 인터페이스의 문서화 주석을 재사용 가능)

**여러 클래스가 상호작용하는 복잡한 API라면, 전체 아키텍처 설명을 추가로 제공하자**

**자바독은 프로그래머가 문서를 올바르게 작성했는지 확인하는 기능 제공**

- `-Xdoclint` 기능 활성화
- 체크스타일과 같은 IDE 플러그인 사용하면 더 완벽하게 검사
- HTML 유효성 검사기로 체크하면 오류를 더 줄일 수 있음
- `-html5` 를 켜면 HTML5 버전으로 생성

### 정리

- 정말 잘 쓰인 문서인지 확인하려면, 자바독 유틸이 만든 웹페이지를 확인해보자.
- 공개 API라면 빠짐없이설명을 달고, 표준 규약을 지키며, HTML 태그를 잘 사용하자.