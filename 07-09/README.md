# 07-09

## 07장. 사과와 오렌지

### 동기: Franc과 Dollar를 비교하면?

```java
public void testEquality() {
    assertFalse(new Franc(5).equals(new Dollar(5)));
}
```

- 테스트 실패, 즉 프랑과 달러가 같다고 주장하는 현상이 생깁니다.

### 대안: 클래스 비교

```java
public class Money {
    protected int amount;
    public boolean equals(Object object) {
        Money money = (Money) object;
        return amount == money.amount
            && getClass().equals(money.getClass());
    }
}
```

- 지저분합니다.
- 도메인 용어(통화 등)이 아닌 객체 용어(클래스)를 쓰고 있습니다.
- 다만 모티베이션이 없으니 일단 둡시다.

### 정리

- 결함을 테스트에 담았습니다.
- 완벽하지 않지만 그럭저럭 봐줄 만한 방법으로 테스트를 통과하게 만들었습니다.
- 더 많은 모티베이션이 있기 전에 더 많은 설계를 도입하지 않습니다.

## 08장. 객체 만들기

### 동기: 중복된 코드

```java
class Franc extends Money {
    Franc(int amount) {
        this.amount = amount;
    }
    Franc times(int multiplier) {
        return new Franc(amount * multiplier);
    }
}

class Dollar extends Money {
    Dollar(int amount) {
        this.amount = amount;
    }
    Dollar times(int multiplier) {
        return new Dollar(amount * multiplier);
    }
}
```

- `times(int)`: 중복된 코드가 있습니다.
- 반환 타입을 Money로 통일하면 더욱 눈에 띕니다.

### 생각해보기

- 두 하위 클래스에 역할이 많지 않아 그냥 제거하고 싶지만, 드라마틱한 변화는 TDD를 효과적으로 보여주기에 적절하지 않습니다.
- 아이디어: 하위 클래스에 대한 직접 참조를 제거하고 싶습니다.

### 대안: 팩토리 메서드

```java
class Money {
    protected int amount;

    public boolean equals(Object object) {
        Money money = (Money) object;
        return amount == money.amount
            && getClass().equals(money.getClass());
    }

    static Dollar dollar(int amount) {
        return new Dollar(amount);
    }
    static Franc franc(int amount) {
        return new Franc(amount);
    }
}

public void testMultiplication() {
    Dollar five = Money.dollar(5);
    assertEquals(new Dollar(10), five.times(2));
    assertEquals(new Dollar(15), five.times(3));
}
```

### 하위 클래스 참조 줄이기

```java
abstract class Money {
    protected int amount;

    abstract Money times(int multiplier);

    public boolean equals(Object object) {
        Money money = (Money) object;
        return amount == money.amount
            && getClass().equals(money.getClass());
    }

    static Money dollar(int amount) {
        return new Dollar(amount);
    }
    static Money franc(int amount) {
        return new Franc(amount);
    }
}

public void testMultiplication() {
    Money five = Money.dollar(5);
    assertEquals(Money.dollar(10), five.times(2));
    assertEquals(Money.dollar(15), five.times(3));
}
```

1. 테스트 선언부 타입을 `Money`로 변경했습니다.
2. Money에는 times()가 없습니다. 일단 구현하기엔 이르므로, Money를 추상 클래스로 변경한 후 times()를 추상 메서드로 선언합시다. 사실 추상 클래스는 처음부터 했어야 했을 것 같습니다.
3. 마지막으로 팩토리 메서드 반환 타입을 변경합니다.
4. 테스트에서 하위 클래스 참조를 완전히 제거합니다.

### 발견한 문제

- `testDollarMultiplication`에서 로직을 다 테스트해서, `testFrancMultiplication`이 의미가 없습니다. 다만 일단 둡시다.
- `times()` 구현이 여전히 중복됩니다.

### 정리

- times()의 두 변이형 메서드 서명부를 통일시켜서 중복 제거를 향해 한 단계 전진했습니다.
- 일단 메서드 선언부를 공통 superclass로 옮겼습니다.
- 팩토리 메서드를 도입하여, 테스트 코드에서 콘크리트 하위 클래스의 존재 사실을 분리했습니다.
- 몇몇 중복된 테스트가 남게 됐지만, 일단 그대로 두었습니다.

## 09장. 우리가 사는 시간

목적: 통화?

### 생각해보기

- 하위 클래스를 제거하기 위해 통화를 도입해봅시다.
- 통화 개념을 어떻게 ~~구현~~, 아니, 테스트할까요?
- 복잡한 객체 및 경량 팩토리(flyweight factories)를 쓸 수도 있겠지만, 일단은 문자열을 씁시다.

### 1: 통화 개념 테스트

```java
public void testCurrency() {
    assertEquals("USD", Money.dollar(1).currency());
    assertEquals("CHF", Money.franc(1).currency());
}
```

### 2: 통화 개념 구현

```java
abstract class Money {
    protected int amount;

    abstract Money times(int multiplier);
    abstract String currency();

    public boolean equals(Object object) {
        Money money = (Money) object;
        return amount == money.amount
            && getClass().equals(money.getClass());
    }

    static Money dollar(int amount) {
        return new Dollar(amount, "USD");
    }
    static Money franc(int amount) {
        return new Franc(amount, "CHF");
    }
}

class Dollar extends Money {
    Dollar(int amount) {
        this.amount = amount;
    }

    Money times(int multiplier) {
        return Money.dollar(amount * multiplier);
    }

    String currency() {
        return "USD";
    }
}

class Franc extends Money {
    Franc(int amount) {
        this.amount = amount;
    }

    Money times(int multiplier) {
        return Money.franc(amount * multiplier);
    }

    String currency() {
        return "CHF";
    }
}
```

### 3: 리팩토링

동일한 구조를 원하므로, 통화를 변수에 저장해 메서드에서는 그걸 반환하게만 합니다.

```java
public class Franc extends Money {
    private String currency;

    Franc(int amount) {
        this.amount = amount;
        this.currency = "CHF";
    }

    Money times(int multiplier) {
        return Money.franc(amount * multiplier);
    }

    String currency() {
        return currency;
    }
}

public class Dollar extends Money {
    private String currency;

    Dollar(int amount) {
        this.amount = amount;
        this.currency = "USD";
    }

    Money times(int multiplier) {
        return Money.dollar(amount * multiplier);
    }

    String currency() {
        return currency;
    }
}
```

이제 `currency` 변수와 `currency()`가 동일하므로 위로 올립시다(push up).

```java
abstract class Money {
    protected String currency;

    String currency() {
        return currency;
    }
}
```

둘의 생성자 구현이 달라집니다. 문자열을 정적 팩토리 메서드로 옮겨서, 둘의 생성자를 동일하게 만듭시다.

그러다 times() 구현이 깨져서, 빠르게 고쳤습니다.

이제 생성자가 동일해졌으니, 상위 클래스로 올립시다.

```java
Money(int amount, String currency) {
    this.amount = amount;
    this.currency = currency;
}

Franc(int amount, String currency) {
    super(amount, currency);
}

Dollar(int amount, String currency) {
    super(amount, currency);
}
```

- 하위 클래스 생성자 구현을 `super()`를 호출하도록 변경했습니다.

### 작은 단계를 밟아가면서 일하기

- 도중에 다른 리팩토링 아이디어가 생겼다면, 원칙적으로는 기다리는 것이 맞습니다.
- 다만 개인적으로 짧은 중단은 받아들입니다.
- 물론 nested 중단은 하지 않습니다. (짐 코플린 Jim Coplien의 가르침)
- TDD는 조종해가는 과정입니다. 보폭이 크다고 느끼면 줄이면 되고, 작다고 느끼면 늘리면 됩니다.

### 정리

- 큰 설계 아이디어를 다루다가 곤경에 빠져서, 일단 더 작은 작업을 수행했습니다.
- 구현에 차이가 있는 부분들은 호출자(팩토리 메서드)로 옮겨서, 두 생성자를 일치시켰습니다.
- times()를 위해 잠시 리팩토링을 중단했습니다.
- 비슷한 리팩토링을 한번의 큰 단계로 처리했습니다.
- 동일한 생성자들을 상위 클래스로 올렸습니다.
