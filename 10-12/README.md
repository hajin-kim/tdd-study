## 10장. 흥미로운 시간

> 할일 목록
**- 공용 times**
>

목표: Money를 나타내기 위한 단 하나의 클래스만을 남기기. (Dollar 와 Franc 클래스 제거)

```java
Franc
Money times(int multiplier) {
	return Money.franc(amount * multiplier);
}

Dollar
Money times(int multiplier) {
	return Money.dollar(amount * multiplier);
}
```

1. 이 둘을 동일하게 만들기 위해, 우선 팩토리 메서드를 인라인 시켜 다시 생성자를 반환 값으로 주도록 한다.

```java
Franc
Money times(int multiplier) {
	return new Franc(amount * multiplier, "CHF");
}

Dollar
Money times(int multiplier) {
	return new Dollar(amount * multiplier, "USD");
}
```

1. 각 Franc, Dollar 클래스에서는 인스턴스 변수 currency가 항상 ‘CHF’, ‘USD’이므로 다음과 같이 고칠 수 있다.

```java
Franc
Money times(int multiplier) {
	return new Franc(amount * multiplier, **currency**);
}

Dollar
Money times(int multiplier) {
	return new Dollar(amount * multiplier, **currency**);
}
```
2. 이제 times() 의 리턴 클래스로 Franc을 가질지 Money를 가질지가 정말 중요한 사실인지 생각해보자.
    - 고민 대신 우선 수정 후 테스트를 돌려 컴퓨터에게 물어보자.
    - Franc.times() 메서드가 Money를 반환하도록 고쳐보자.

```java
Franc
Money times(int multiplier) {
	return new **Money**(amount * multiplier, currency);
}
```
3. 컴파일러가 Money를 abstract이 아닌 concrete 클래스로 바꿔야 한다고 알려준다.

```java
class Money
    Money times(int amount) {
        return null;
}
```
4. 빨간 막대와 에러 메시지로 `“expected: <Money.Franc@31aebf> but was: <Money.Money@478a43>”` 를 얻었다. 더 나은 메세지를 보기 위해 toString() 을 정의하자.
    1. 테스트도 없이 코드를 작성해도 되나 싶지만, toString()은 디버그 출력에만 쓰이기에 이를 잘못 구현함으로써 얻게 될 리스크가 적다.
    2. 이미 빨간 막대 상태이기에 새로운 테스트를 작성하지 않는게 좋다.

```java
public String toString() {
      return amount + " " + currency;
}
```

5.  이제 에러 메시지로 `“expected:<10 CHF> but was:<10 CHF>”` 가 나온다.
    1. 답은 맞았는데 클래스가 다르다. 이 뜻은 equals() 구현에 문제가 있는걸 가르킨다.
    2. 이전에 우리는 화폐가 동일한지 알기 위한 equals() 구현을 클래스를 비교하는 방식으로 테스트했다.
    3. 하지만 우리가 정말 검사하고 싶은건 **클래스가 같은지가 아니라 currency가 같은지 여부다.**

```java
public boolean equals(Object object){
    Money money = (Money) object;
    return amount == money.amount && getClass().equals(money.getClass());
  }
```

- 빨간 막대 상태에서 테스트를 추가로 작성하고 싶지 않다.
- 하지만 테스트 없이는 모델 코드를 수정할 수 없다.
- 변경된 코드를 되돌려 다시 초록 막대 상태로 돌아가보자.

```java
  Money times(int multiplier){
    return new Franc(amount* multiplier, currency);
  }
```

- times의 리턴을 Money에서 Franc로 돌렸다.
- 우리 상황은 Franc(10, "CHF") 과 Money(10, "CHF")가 서로 같기를 바랬지만, 사실은 그렇지 않다고 보고된 것이다. 따라서 이걸 **그대로 테스트로 사용할 수 있다!**

```java
public void testDiffrentClassEquality(){
    assertTrue(new Money(10, "CHF").equals(new Franc(10, "CHF")));
  }
```

- 이는 예상대로 실패한다. equals() 코드는 클래스가 아닌 currency를 비교해야 한다.
- equals() 메서드를 수정해보자.

```java
Money
public boolean equals(Object object){
    Money money = (Money) object;
    return amount == money.amount && currency().equals(money.currency());
  }
```

- 이제 Franc.times() 의 리턴값을 Money로 바꾸어도 테스트는 여전히 통과한다.
- 이는 Dollar.times() 에도 동일하게 적용해 이의 리턴값도 Dollar 에서 Money 클래스로 바꿔주자.

```java
**Dollar**
Money times(int multiplier) {
    return new Money(amount* multiplier, currency);
  }
**Franc**
  Money times(int multiplier) {
    return new Money(amount* multiplier, currency);
  }
```

- 이제 각 times() 의 구현이 동일해졌으니, 상위 클래스로 끌어올린다.

```java
Money
 Money times(int multiplier) {
    return new Money(amount* multiplier, currency);
  }
```

이제 아무것도 하지 않는 Dollar, Franc 하위 클래스들을 제거할 수 있다.

정리

- 두 times()를 일치시키기 위해 그 메서드들이 호출하는 다른 메서드들을 인라인 시킨 후 상수를 변수로 바꿔주었다.
- 단지 디버깅을 위해 테스트 없이 toString()을 작성하였다.
- Franc 대신 Money를 반환하는 변경을 시도한 뒤 그것이 잘 작동할지를 테스트가 말하도록 했다.
- 실험한 것을 뒤로 물리고 또 다른 테스트를 작성했다. 테스트를 작동해했더니 실험도 제대로 작동했다.

---

## 11장. 모든 악의 근원

> 할일 목록
**- Dollar / Franc 중복**
>

- 현재 Dollar 와 Franc 클래스에는 생성자만 존재한다.
- 단지 생성자 때문에 하위 클래스가 있을 필요가 없기에, 하위 클래스를 제거하자.
- 코드의 의미를 변경하지 않으면서도 하위 클래스에 대한 참조를 상위 클래스에 대한 참조로 변경할 수 있다.

```java
Money
static Money dollar(int amount){
    return new **Money**(amount, "USD");
  }
  static Money franc(int amount){
    return new **Money**(amount, "CHF");
  }
```

- 팩토리 메서드인 Money.franc() 와 Money.dollar() **가** 상위 클래스인 **Money를 반환**하도록 고쳤다.
- 이렇게 되면 Dollar에 대한 참조는 남아있지 않고, Dollar를 지울 수 있다.
- 하지만 Franc은 우리가 작성했던 테스트 코드에서 아직 참조한다.

### Franc을 참조하는 테스트 코드 제거

```java
public void testDifferentClassEquality(){
    assertTrue(new Money(10, "CHF").equals(new Franc(10, "CHF")));
  }
```

- 이 테스트를 지워도 될 정도로 다른 곳에서 동치성 테스트를 충분히 하는지 살펴보자.

```java
public void testEquality(){
    assertTrue(Money.dollar(5).equals(Money.dollar(5)));
    assertFalse(Money.dollar(5).equals(Money.dollar(6)));
    ~~assertTrue(Money.franc(5).equals(Money.franc(5)));
    assertFalse(Money.franc(5).equals(Money.franc(6)));~~
    assertFalse(Money.franc(5).equals(Money.dollar(5)));
  }
```

- 충분한 테스트인걸 확인했고, 이제 Franc을 참조하는 testDifferentClassEquality 을 지울 수 있다.
- 또한, 3,4번째 assertion은 1,2번째와 중복되므로 지워주었다.

---

- 클래스 대신 currency를 비교하도록 강요하는 테스트 코드는 여러 클래스가 존재할 때만 의미 있다.
- Franc 클래스를 제거하려는 중이기 때문에, Franc 이 있을 경우에 시스템이 작동하는지 확인하는 테스트는 도움이 안되고 오히려 짐이 된다.
- Franc과 함께 testDifferentClassEquality() 테스트를 지워주자.
- 이와 비슷하게 달러와 프랑에 대한 별도에 테스트들이 존재하는데, 화폐와 상관없이 현재는 로직상의 차이가 없다는 걸 알 수 있다. (클래스가 두 개였을 때는 차이가 있었다). 이에 testDifferentClassEquality() 를 지워도 동작에 대한 신뢰를 잃지 않을 것이다.
- 이제 클래스가 Money 하나뿐이다. 덧셈을 다룰 준비가 됐다!

정리

- 하위 클래스의 속을 들어내는 걸 완료하고, 하위 클래스를 삭제했다.
- 기존의 소스 구조에서는 필요했지만 새로운 구조에서는 필요 없게 된 테스트를 제거했다.

---

## 12장. 드디어, 더하기

> 할일 목록
$5 + 10CHF = $10 (환율이 2:1일 경우)
> 
> **$5 + $5 = 10**

- 전체 더하기 기능의 코드를 적기 어려우니, 좀 더 간단한 예인 **$5 + $5 = 10** 에서 시작해보자.

```java
public void testSimpleAddition(){
    Money sum = Money.dollar(5).plus(Money.dollar(5));
    assertEquals(Money.dollar(10), sum);
  }
```

- 테스트 코드를 작성하고, Money에 plus 메서드를 작성해보자.

```java
Money plus(Money addend){
    return new Money(amount + addend.amount, currency);
  }
```

- Money.dollar(10) 을 반환하도록 가짜 구현을 해도 되지만, 어떻게 구현할 지 명확하기에 위와 같이 같이 작성했다.

### 다중 통화 연산을 어떻게 표현해야 할까?

우리는 설계상 다중 통화 사용에 대한 내용을 시스템의 나머지 코드에게 숨기고 싶다.

- 모든 내부 값을 참조통화*(환전에서 기준이 되는 화폐)* 로 전환하는 방법이 있다.
    - 하지만 이는 여러 환율을 쓰기 쉽지 않다. 이에 편하게 여러 환율을 표현할 수 있으면서도 산술 연산 비슷한 표현들을 여전히 산술 연산처럼 다룰 수 있는 해법이 필요하다. ( 다른 통화간 + 의 산술 연산 표현 )
    - 가지고 있는 객체가 우리가 원하는 방식으로 동작하지 않을 경우엔, 그 객체와 외부 프로토콜이 같으면서 내부 구현은 다른 새로운 객체 (imposter, 사칭 사기꾼) 를 만들 수 있다.


- 해법:  Money와 비슷하게 동작하지만 사실은 **두 Money의 합을 나타내는 객체를 만든다.** 비유를 들어 생각해보자. ****
- 메타포 1)
    - Money의 합을 마치 지갑처럼 취급해보자.
    - 한 지갑에는 금액과 통화가 다른 여러 화폐들이 들어갈 수 있다.
- 메타포 2)
    - (2 + 3) x 5 와 같은 수식
    - 우리 경우에는 ($2 + 3CHF) x 5가 되겠지만, 이렇게 하면 Money를 수식의 가장 작은 단위로 볼 수 있다.
    - 연산의 결과로 Expression들이 생기는데, 그 중 하나는 합 (sum)이 될 것이다.
    - 연산(포트폴리오의 값을 합산하는 것 등)이 완료되면, 환율을 이용해서 결과 Expression을 **단일 통화로 축약**시킬 수 있다.
    - 이 메타포를 테스트에 적용해보자. 마지막 줄은 기존과 비슷하지만 단일 통화로 축약된 reduced 를 적용해 `assertEquals(Money.dollar(10), reduced);` 로 끝날 것이다.


- reduce (축약된)란 이름의 Expression(식)은 Expression에 환율을 적용함으로써 얻어진다.
- 환율이 적용되는 곳은 은행이라고 생각할 수 있다. 이를 다음과 같이 쓸 수 있다.

```java
public void testSimpleAddition(){
    *//Money sum = Money.dollar(5).plus(Money.dollar(5));
     Money reduced = bank.reduce(sum, "USD"); //은행이 식에 환율을 적용
    assertEquals(Money.dollar(10), sum);
  }
```

- 하지만 왜 bank 가 reduce() 를 수행하는 책임을 맡아야 할까? (왜 축약이 수식이 아닌 은행의 책임이어야 한다고 생각했을까?)
    - Expression은 우리가 하려고 하는 일의 핵심에 해당한다.
    - 그리고 핵심이 되는 객체는 다른 부분에 대해서 될 수 있는 한 모르도록 해야 한다. 이는 핵심 객체를 오랫 동안 유연하게 유지하고, 재활용하거나 테스트하기 쉽게 한다.
    - Expression과 관련이 있는 오퍼레이션이 많을 거라고 상상할 수 있다. 만약 모든 오퍼레이션을 Expression에만 추가한다면 Expression은 무한히 커질 것이다.
    - 이는 지금 이렇게 진행하기로 한 결정에 충분한 이유가 된다고 판단, 만약 나중에 Bank가 필요 없게 된다면 축약을 구현할 책임을 Expression으로 옮겨도 된다.

이제 테스트를 작성해보자.

1. Bank 객체를 생성해준다.
2. **두 Money의 합은 Expression이어야 한다.**
3. $5는 팩토리 메서드로 간단하게 만들 수 있다.

```java
public void testSimpleAddition(){
  ~~Money sum = Money.dollar(5).plus(Money.dollar(5));~~  //기존 코드

	Money five = Money.dollar(5); //3. $5는 팩토리 메서드로 간단하게 만들 수 있다. 
	Expression sum = five.plus(five); //2. 두 Money의 합은 Expression이어야 한다.
  Bank bank = new Bank(); // 1. Bank 객체 생성
  Money reduced = bank.reduce(sum, "USD");  
  assertEquals(Money.dollar(10), reduced);
}
```

이제 컴파일이 되게 Expression 인터페이스를 만들자.

```java
interface Expression
```

1. Money.plus() 는 Expression을 반환해야 한다. 이는 Money가 Expression을 구현해야 한다는 뜻이다. 아래와 같이 작성해주자.

```java
*class* Money implements Expression {

  Expression plus(Money addend){
    return new Money(amount + addend.amount, currency);
  }
}
```

1. 이제 빈 Bank 클래스가 필요하다. 그리고 Bank 에는 reduce() 의 스텁이 있어야 한다.

```java
Bank 
Money reduce(Expression source, String to){
    ~~return null;~~
		return Money.dollar(10); //가짜 구현
  }
```

1. 마침내 컴파일이 되고, 빨간 막대를 볼 수 있다. 간단히 `return Money.dollar(10)` 로 가짜 구현을 하자.

다시 초록 막대로 돌아왔고, 리팩토링할 준비가 됐다.

---

**정리**

- 큰 테스트를 작은 테스트 ($5 + 10CHF에서 $5 + $5로) 줄여서 발전을 나타낼 수 있도록 했다.
- 우리에게 필요한 계산에 대한 가능한 메타포들을 신중히 생각해봤다.
- 새 메타포에 기반하여 기존의 테스트를 재작성했다.
- 테스트를 빠르게 컴파일 했다.
- 그리고 테스트를 실행했다.
- 진짜 구현을 만들기 위해 필요한 리팩토링을 기대했다.