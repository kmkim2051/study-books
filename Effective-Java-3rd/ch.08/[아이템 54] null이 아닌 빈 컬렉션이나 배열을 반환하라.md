# [아이템 54] null이 아닌 빈 컬렉션이나 배열을 반환하라

```java
// 컬렉션이 비었으면 null 반환 - 흔하지만 따라하지 말 것!!
public List<Cheese> getCheeses() {
  return cheesesInStock.isEmpty() ? null
    : new ArrayList<>(cheesesInStock);
}
```

재고가 없다고 특별취급할 이유는 없음

- null 반환한다면, 클라이언트에서 별도 null 처리 필요
- 객체가 0개일 가능성이 거의 없는 상황에서, 오랜 시간 뒤 오류 발생하기도 함

**빈 컨테이너보다는 null이 성능상 낫지 않나?**

- 분석 결과 성능 하락 원인이 아닌 이상, 아주 미미한 성능 차이
- 빈 컬렉션과 배열은 굳이 새로 할당하지 않고 반환할 수 있다.

**올바른 예**

```java
public List<Cheese> getCheeses() {
  return new ArrayList<>(cheesesInStock);
}
```

가능성은 적지만, 빈 컬렉션 할당이 성능 저하할 수 있음

→ 빈 **불변** 컬렉션을 반환

```java
return list.isEmpty() ? Collections.emptyList() : new ArrayList<>(list);
```

**배열도 마찬가지**

절대 null을 반환하지 말고, **길이가 0인 배열**을 반환

```java
return list.toArray(new Cheese[0]); // 반환 타입을 알려주는 역할
```

성능 저하 우려가 있다면, 길이 0짜리 배열을 미리 선언하고 반환해도 됨

→ 길이 0인 배열은 모두 불변

```java
private static final Cheese[] EMPTY_CHEESE_ARRAY = new Cheese[0];
// ...
  return stock.toArray(EMPTY_CHEESE_ARRAY);
```

항상 EMPTY ARRAY를 인수로 넘겨 toArray 호출

→ stock이 비었을 경우 언제나 EMTPY_ARRAY를 반환하게 된다.

toArray는 배열이 충분히 크면 배열 안에 원소를 담아 반환, 그렇지 않으면 T[]를 새로 만들어 원소를 담아 반환

- 단순 성능개선 목적이라면 추천하지 않는다. (오히려 성능 하락한다는 연구 결과도 있음)

### 정리

null 이 아닌, 빈 배열이나 컬렉션을 반환하라.