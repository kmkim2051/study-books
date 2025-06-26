# [아이템 37] ordinal 인덱싱 대신 EnumMap을 사용하라

```java
class Plant {
  enum LifeCycle { ANNUAL, PERENNIAL, BIENNAIL }
  
  
  final String name;
  final LifeCycle lifeCycle;
}
```

식물들을 배열 하나로 관리하고, 생애주기별로 묶는다.

- 생애주기별로 총 3개의 집합
- 정원을 돌며 각 식물을 해당 집합에 넣는다.

```java
Set<Plant>[] plantsByLife = 
  (Set<Plant>[]) new Set[Plant.LifeCycle.values().length];
for (int i = 0; i < plantsByLife.length; i++)
  plants[i] = new HashSet<>();
  
for (Plant p : garden) {
  plants[p.lifeCycle.ordinal()].add(p);
```

### ordinal을 배열 인덱스로 사용

- 제네릭과 호한되지 않음, 컴파일 깔끔하지 않음
- 배열 인덱스의 의미가 없어 출력 결과에 레이블을 달아줘야 함
- **정확한 정숫값을 사용한다는 것을 직접 보증해야 함**
    - Bound Exception 가능성

### Map으로 해결

열거 타입을 키로 사용하도록 설계한, 빠른 Map 구현체 `EnumMap` 

```java
Map<Plant.LifeCycle, Set<Plant>> plants = new EnumMap<>(Plant.LifeCycle.class)

for (Plant.LifeCycle lc : Plant.LifceCycle.values())
  plants.put(lc, new HashSet<>());
  
for (Plant p : garden)
  plants.get(p.lifeCycle).add(p);
```

짧고, 명료하고, 성능도 비등하다.

- 내부에서 배열을 쓰기 때문
- EnumMap 생성자에 한정적 타입 토큰을 넘겨, 런타임 제네릭 타입 정보 제공

`Stream` 을 사용한 코드 1

```java
Arrays.stream(garden).collect(groupingBy(p -> p.lifeCycle)));
```

→ 코드는 간결하지만 EnumMap의 공간, 성능 이점이 사라지는 문제

`Stream` 을 사용한 코드 2

```java
// mapFactory 매개변수에 원하는 맵 구현체를 명시해 호출
Arrays.stream(garden)
  .collect(groupingBy(p -> p.lifeCycle, 
    () -> new EnumMap<>(LifeCycle.class), toSet())))
```

**Stream과 EnumMap 사용 사례 차이**

- EnumMap을 사용하면 식물의 생애주기당 하나의 중첩 맵 생성
- Stream은 식물이 있을 때만 생성

**ordinal을 두번이나 사용한 예시**

```java
public enum Phase {
  SOLID, LIQUID, GAS;
  
  public enum Transition {
    MELT, FREEZE, BOIL, CONDENSE, SUBLIME, DEPOSIT;
    
    private static final Transition[][] TRANSITIONS = {
      {null, MELT, SUBLIME},
      {FREEZE, null, BOIL},
      {DEPOSIT, CONDENSE, null}
    };
    
    public static Transition from(Phase from, Phase to) {
      return TRANSITIONS[from.ordinal()][to.ordinal()];
    }
```

- 컴파일러는 ordinal과 배열 인덱스 사이 관계를 알 수 없음
- 열거 타입을 수정하면서 배열을 수정하지 않으면 ArrayBoundExeption, NPE, 에러 없이 이상한 동작
- 상태의 가짓수가 늘어날수록 제곱으로 커짐

**Map 2개를 중첩한 EnumMap으로 해결**

이전 상태에서 ‘이후 상태에서 전이로의 맵에 대응’시키는 맵

```java
public enum Phase {
  SOLID, LIQUID, GAS;
  
  public enum Transition {
    MELT(SOLID, LIQUID),
    FREEZE(LIQUID, SOLID),
    BOIL(LIQUID, GAS),
    // ...
    ;
    
    private final Phase from;
    private final Phase to;
    
    private static final Map<Phase, Map<Phase, Transition>> m =
      Stream.of(values())
      .collect(groupigBy(t -> t.from,
        () -> new EnumMap<>(Phase.class),
        toMap(t -> t.to, t -> t, (x, y) -> y, () -> new EnumMap<>(Phase.class())));
     
     public static Transition from(Phase from, Phase to) {
       return m.get(from).get(to);
     }
```

SOLID → LIQUID로 가는 현상은 무엇인가?

input(SOLID, LIQUID) → output(MELT)

- MELT(SOLID, LIQUID)
→ (SOLID, Map(LIQUID, MELT))

새로운 상태와 전이가 추가되면 알맞은 Map에만 넣어주면 됨