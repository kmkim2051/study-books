# [아이템 11] equals를 재정의하려거든 hashCode도 재정의하라

**equals를 재정의한 모든 클래스에서 hashCode도 재정의해야한다.**

→ 그렇지 않으면 hashCode 규약을 어기게 되어 HashMap, HashSet 등에서 문제 발생

### Object 명세

> 1. equals 비교 정보가 변경되지 않았다면, hashCode 메서드는 항상 일관된 값을 반환해야 함
2. equals 가 true 로 판단되면, hashCode 값은 같아야 함
3. equals 가 false 라도, hashCode도 달라야 하는 것은 아님 (다만 달라야 해시테이블 성능 좋아짐)
> 

hashCode 잘못 재정의 시 문제 발생하는 포인트는 2번, 즉 논리적으로 같은 객체는 같은 해시코드를 반환해야 함

```java
Map<Phone, String> m = new HashMap<>();
m.put(new Phone(123, "kim"));

// hashCode 문제 시 null이 반환됨
m.get(new Phone(123, "kim"));
```

먼저 다른 버킷에서 찾으려 하기 때문이고, 같은 버킷이라도 해시코드가 다르면 동치성 비교조차 하지 않음

최악이지만 적법한 hashCode 구현 (모든 객체가 동일 버킷에 담겨 linked-list 처럼 동작)

→ 좋은 해시함수라면 서로 다른 인스턴스에 다른 해시코드를 반환한다. (규약 3)

```java
@Override int hashCode() {
  return 123;
}
```

이상적인 해시함수는 서로 다른 인스턴스들을 32비트 정수 범위에 균일하게 분배해야 한다.

### 좋은 hashCode를 작성하는 요령

1. int result 선언 후 값 c로 초기화
    1. c는 해당 객체의 첫 핵심 필드를 2.a 방식으로 계산한 해시코드
2. 해당 객체의 나머지 핵심 필드 f 각각에 다음 작업 수행
    1. 해당 필드의 해시코드 c 계산
        1. 기본 타입 필드 → Type.hashCode(f) 수행
        2. 참조타입이고, 클래스가 필드의 equals를 재귀적으로 호출 → hashCode를 재귀적으로 호출
            1. 계산이 복잡해진다면 표준형으로 비교, null 이면 0을 대신 사용
        3. 필드가 배열이라면, 핵심 원소 각각을 필드처럼 적용
            1. 핵심 원소가 없다면 상수(보통 0), 모든 원소가 핵심 원소라면 Arrays.hashCode
    2. 2.a 의 c로 result 갱신
        1. result = 31 * result + c
    3. return result;

구현을 완료했다면, 동치 인스턴스에 대해 같은 값을 반환할지 자문하고 단위 테스트를 작성

**equals에 쓰이지 않은 필드는 반드시 제외해야 함**

*31 * result* 는 필드를 곱하는 순서에 따라 result 값을 달라지게 함 → 비슷한 필드가 여러개일 때 해시 효과 up

- 31 인 이유? 홀수 & 소수, (i << 5 - i)로 최적화 가능
- 짝수는 하위 비트에 영향을 덜 줘서 충돌 확률을 높이고 오버플로우시 정보 손실 가능

**전형적인 hashCode 메서드**

```java
@Override
public int hashCode() {
  int result = Short.hashCode(areaCode);
  result = 31 * result + Short.hashCode(prefix);
  result = 31 * result + Short.hashCode(lineNum);
  return result;
}
```

**Objects.hash 메서드**

파라미터를 위한 배열, 박싱/언박싱 등으로 성능이 살짝 아쉬움

```java
return Objects.hash(lineNum, prefix, areaCode);
```

**클래스가 불변이고, 해시코드 계산 비용이 크다면**

- 매번 새로 계산하기보다는 캐싱 방식을 고려
- 이 타입의 객체가 해시 키로 자주 사용된다면 → 생성 시 해시코드 미리 계산
    - 그렇지 않다면, 지연 초기화 → 스레드 안정성까지 고려해야 함
        
        ```java
        private int hashCode; // 기본값 0
        // volatile 로 가시성과 순서 보장 가능
        
        @Override
        public int hashCode() {
            // hashCode값을 다시 계산하지 않도록, result로 복사 후 검증
            int result = hashCode;
            
            if (result == 0) {
                result = Short.hashCode(areaCode);
                result = 31 * result + Short.hashCode(prefix);
                result = 31 * result + Short.hashCode(lineNum);
                hashCode = result;
            }
            return result;
        }
        ```
        

**해시코드 계산의 성능 개선을 위해 핵심 필드를 생락해서는 안된다.**

자바 2 이전의 String은 최대 16개의 문자만으로 해시코드를 계산했음

- URL처럼 계층적인 문자열의 경우 문제 발생 가능성 높음

**hashCode 생성 규칙을 사용자에게 자세히 알리지 말것**

클라이언트가 이 값에 의지하지 않고, 추후에 계산 방식을 변경할 수도 있음