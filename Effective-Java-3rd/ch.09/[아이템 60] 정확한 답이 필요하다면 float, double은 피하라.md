# [아이템 60] 정확한 답이 필요하다면 float, double은 피하라

### 과학과 공학 계산용으로 설계된 float, double

이진 부동소수점 연산에 쓰이며, 넓은 범위를 빠르게 근사치로 계산

→ 특히 금융과 같이, 정확한 결과가 필요한 경우 사용하면 안됨

0.1, 10^-2 등을 표현할 수 없기 때문

**예시**

- 1.03 달러에서 42센트를 쓰면?
    - `System.out.println(1.03 - 0.42);`
    - 정확히 0.61 이 아닌, 0.61000000000001을 출력
- 1달러로 10센트 사탕 9개를 사면?
    - `System.out.println(1.00 - 9 * 0.10);`
    - 0.1이 아닌, 0.9999999998을 출력

**이러한 문제를 해결하려면**

(금융 계산에는) BigDecimal, int, long을 써야 한다.

### BigDecimal

BigDecimal을 사용한 코드 → 속도가 느리고 불편하다

```java
final BigDecimal TEN_CENTS = new BigDecimal(".10");

int itemBought = 0;
BigDecimal funds = new BigDecimal("1.00");

for (BigDecimal price = TEN_CENTS; funds.compareTo(price) >= 0;
          price = price.add(TEN_CENTS)) {
     funds = funds.subtract(price);
     itemBought++;
}

sout(itemBought + " 개 구입");
sout("잔돈 : " + funds);
```

BigDecimal의 단점

- 기본 타입보다 쓰기가 훨씬 불편하고, 훨씬 느리다.

### 대안 int, long

값의 크기가 제한되고, 소수점을 직접 관리해야 한다.

```java
int itemBought = 0;
int funds = 100; // 1 dollar = 100 cent

for (int price = 10; funds >= price; price += 10) {
     funds -= price;
     itemBought++;
}

sout(itemBought + " 개 구입");
sout("잔돈 : " + funds);
```

### 정리

- 정확한 답이 필요하면 float, double은 피하라
- 불편함 및 성능 저하를 감수한다면 BigDecimal을 사용해 반올림 제어
- 성능이 중요, 소수점 추적 가능, 숫자 범위 제한 → int, long
- ~10^9 → int, ~10^18 → long, 10^18+ → BigDecimal