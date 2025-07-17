# [아이템 58] 전통적 for 문보다는 for-each를 사용

스트림이 제격인 작업이 있고, 반복이 제격인 작업이 있다. (아이템 45)

**전통적 for 문으로 컬렉션 순환하는 코드**

```java
for (Iterator<Element> i = c.iterator(); i.hasNext());
  Element e = i.next();
  // ...
}
```

**전통적 for문으로 배열 순환하는 코드**

```java
for (int i = 0; i < a.length; i++) {
  ... // a[i]로 무언가 한다.
}
```

→ while문보다는 낫지만 가장 좋은 방법은 아님!

- 반복자와 인덱스 변수는 코드를 지저분하게 함 (실제 필요한건 원소인데도)
- 쓰이는 요소 종류가 늘어나면 오류 가능성 높아짐

**이러한 문제들은 for-each문 사용으로 모두 해결**

```java
// e in elements 라고 읽으면 된다.
for (Element e : elements) {
  // ...
}
```

컬렉션이든 배열이든, 성능은 크게 차이나지 않는다.

컬렉션을 중첩 순회해야 한다면, for-each 문의 이점이 더욱 커진다.

```java
enum Suit { CLUB, DIAMOND, HEART, SPADE };
enum Rank { ACE, DEUCE, THREE, ...}

static Collection<Suit> suits = Arrays.asList(Suit.values());
static Collection<Rank> suits = Arrays.asList(Rank.values());

List<Card> deck = new ArrayList<>();
for (Iterator<Suit> i = suits.iterator(); i.hasNext();)
  for (Iterator<Rank> j = ranks.iterator(); i.hasNext();)
    deck.add(new Card(i.next(), j.next()); // i가 매번 불리는 버그
    // NoSuchElementException 발생
```

같은 컬렉션이라면, 예상치 못한 결과만 출력하고 종료

```java
enum Face {ONE, TWO, TRHEE}
Collection<Face> faces = EnumSet.allOf(Face.class);
// ONE-ONE, TWO-TWO, THREE-THREE
```

for-each문 도입으로 간단하게 해결된다.

```java
for (Suit suit : suits)
  for (Rank rank : ranks)
    deck.add(new Card(suit, rank));
```

### for-each 사용할 수 없는 세 가지 경우

아래 세 가지 상황은 일반 for문을 사용하자

- 파괴적 필터링
    - 컬렉션을 순회하며 선택된 원소를 제거해야 하는 경우 (remove 필요)
    - 자바 8, Collection의 removeIf로 회피 가능
- 변형
    - 원소의 값 일부 혹은 전체를 교체하는 경우 (반복자나 인덱스 필요)
- 병렬 반복
    - 여러 컬렉션을 병렬로 순회해야 하는 경우, 각 반복자와 인덱스 명시적 제어 필요

for-each문은 컬렉션, 배열, Iterable 인터페이스가 구현된 객체 순회 가능

```java
public interface Iterable<E> {
  Iterator<E> iterator();
}
```