# 13-15

## 13. 진짜로 만들기

- 현재 코드 상황

```c
class Bank{
	Money reduce(Expression source, String to){
		return Money.dollar(10);
	}
}

class Money{
	Expression plus(Money addend){
		return new Money(this.amount + addend.amount, this.currency);
	}
}

```

- Bank의 reduce를 가짜 구현으로 만들어두었다.

```c
public void testSimpleAddition(){
	Mony five = Money.dollar(5);
	Expression sum = five.plus(five);
	Bank bank = new Bank();
	Money reduced = bank.reduce(sum, "USD");
	assertEquals(Money.dollar(10), reduced);
}

```

- 위는 가짜 구현을 통과하는 테스트코드이다.

우선 Money.plus 메서드가 Expression(Money) 가 아닌 Expression(Sum)을 반환해야하기 때문에 이를 테스트코드에 추가한다. (두 Money의 합은 Sum 이여야하기 때문)

```c
public void testSimpleAddition(){
	Mony five = Money.dollar(5);
	Expression sum = five.plus(five);
	Sum sum = (Sum) result;
	assertEquals(five, sum.augend);
	assertEquals(five, sum.addend);
}

```

위 코드를 통과하기 위해 augend, addend 필드와 생성자를 가지고 Expression 을 implements 한 Sum class가 우선 필요하다.

```c
class Sum implements Expression {
	Money augend;
	Money addend;

	Sum(Money augend, Money addend){
		this.augend = augend;
		this.addend = addend;
	}

}

```

또한 Money.plus() 메서드에서 Money가 아닌 Sum 을 return 하도록 바꾸고 reduce 메소드에도 이를 추가한다.

```c
class Money{
	Expression plus(Money addend){
		return new Sum(this, addend);
	}
}

```

```c
class Bank{
	Money reduce(Expression source, String to){
		Sum sum = (Sum) source;
		int amount = sum.augend.amount + sum.added.amount;
		return new Money(amount, to);
	}
}

```

위와 같은 코드는 다음 이유로 지저분하다.

- 모든 Expression 에 대해서 캐스팅이 성공해야만한다.
- public 필드와 그 필드들에 대해 두 단계에 걸친 레퍼런스가 있다.

이 문제들을 해결하기 위한 테스트코드를 짜보자면 아래와 같다.

```c
public void testReduceMoney(){
	Bank bank = new Bank();
	Money result = bank.reduce(Money.dollar(1), "USD");
	assertEquals(Money.dollar(1), result)
}

```

이러한 점들을 아래와 같이 고칠 수 있다.

```c
class Bank{
	Money reduce(Expression source, String to){
		if(source instanceof Money) return (Money) source;
		Sum sum = (Sum) source;
		return sum.reduce(to)
	}
}

class Sum{
	public Money reduce(String to){
		int amount = augend.amount + added.amount;
		return new Money(amount, to);
	}
}

```

여기서 sum에서 reduce(String)을 구현하고 있으므로 Money에서도 이 메소드의 구현을 만들어 reduce()를 Expression interface에 정의할 수 있다.

이 점을 이용해서 아래와 같이 코드를 리펙토링 할 수 있다.

```java
class Money{
	public Money reduce(String to){
		return this;
	}
}

class Expression{
	Money reduce(String to);
}

class Bank{
	Money reduce(Expression source, String to){
		return source.reduce(to)
	}
}

```

## 14. 바꾸기

```c
public void testReduceMoneyDifferntCurrency(){
	Bank bank = new Bank();
	bank.addRate("CHF", "USD", 2);
	Money result = bank.reduce(Money.franc(2), "USD");
	assertEquals(Money.dollar(1), result);
}

```

이제 위의 테스트코드와 같이 환율에 대한 조건을 추가하고 환전을 해야한다.
아무래도 환율에 대한 일은 Bank 가 처리해야할 것이다.

```c
class Bank{
	Money reduce(Expression source, String to){
		return source.reduce(this, to)
	}

	int rate(String from, String to){
		return(from.equals("CHF") && to.equals("USD")
			? 2
			: 1;
	}
}

class Expression{
	Money reduce(Bank bank, String to);
}

class Sum{
	public Money reduce(Bank bank, String to){
		int amount = augend.amount + added.amount;
		return new Money(amount, to);
	}
}

class Money{
	public Money reduce(Bank bank, String to){
		int rate = bank.rate(currency, to);
		return new Money(amount/rate, to);
	}
}

```

따라서 Bank에 환율을 가져오는 메서드를 만들어두고 Money에서 Bank를 참조해 reduce한다.

이제 환율을 적용하는 과정을 만들었으므로 환율을 저장하는 부분을 만들어야한다.

```java
public void testArrayEquals(){
	assertEquals(new Object[] {"abc"}, new Object[] {"abc"});
}

```

위와 같이 서로 다른 두개의 해시 테이블은  내부 원소가 같더라도 서로 다르다고 판단하기 때문에 대로 사용할 수 없다. 하지만 주석을 보니 배열에 대해서 assertArrayEquals를 사용하면 된다고 하는 것 같기도....?

따라서 키를 위한 객체를 따로 만들어주어야한다.

```java
@AllArgConstructor
@EqualsAndHashCode
private class Pair{
	private String from;
	private String to;
}

```

이제 환율을 저장하고 설정하는 부분을 Bank에 만들어야한다.

```java
class Bank{
	private Hashtable rates = new Hashtable();

	Money reduce(Expression source, String to){
		return source.reduce(this, to)
	}

	int rate(String from, String to){
		Integer rate = (Integer) rates.get(new Pair(from, to));
		return rate.intValue();
	}

	void addRate(String from, String to, int rate){
		rates.put(new Pair(from, to), new Integer(rate));
	}
}

```

이때, USD 에서 USD 로 의 변경을 요청하는 경우 헨들링이 추가로 필요하다.
따라서 아래와 같이 테스트코드를 먼저 작성해준다.

```java
public void testIdentityRate(){
	assertEquals(1, new Bank().rate("USD", "USD"));
}

```

그리고 아래와 같이 헨들링하는 코드를 추가해준다.

```java
class Bank{
	private Hashtanle rates = new Hashtable();

	Money reduce(Expression source, String to){
		return source.reduce(this, to)
	}

	int rate(String from, String to){
		if(from.equals(to)) return 1;
		Integer rate = (Integer) rates.get(new Pair(from, to));
		return rate.intValue();
	}

	void addRate(String from, String to, int rate){
		rates.put(new Pair(from, to), new Integer(rate));
	}
}

```

## 15. 서로 다른 통화 더하기

이제 $5+10CHF 를 테스트를 추가할 준비가 되었다.
테스트는 아래와 같이 작성될 것이다.

```java
public void testMixedAddition(){
	Expression fiveBucks = Money.dollar(5);
	Expression tenFrancs = Money.franc(10);
	Bank bank = new Bank();
	bank.addRate("CHF", "USD", 2);

	Money result = bank.reduce(
		fiveBucks.plus(tenFrancs, "USD")
	);

	assertEquals(Money.dollar(10), result);

}

```

- 첫 번째로 Sum.reduce에서 인자를 축약하지 않기 때문에 에러가 발생한다.
기존 아래와 같던 코드를

```java
class Sum{
	public Money reduce(String to){
		int amount = augend.amount + added.amount;
		return new Money(amount, to);
	}
}

```

다음과 같이 변경해주면서 해결한다.

```java
class Sum{
	public Money reduce(String to){
		int amount = augend.reduce(bank, to).amount + added.reduce(bank.to).amount;
		return new Money(amount, to);
	}
}

```

- 이제 원래 Expression이였어야 하는 Money 부분을 조금씩 변경할 수 있다.

```java
class Sum implements Expression {
	Money augend;
	Money addend;

	Sum(Money augend, Money addend){
		this.augend = augend;
		this.addend = addend;
	}

}

//위 코드를 다음과 같이 Expression 들로 변경한다.
class Sum implements Expression {
	Expression augend;
	Expression addend;

	Sum(Expression augend, Expression addend){
		this.augend = augend;
		this.addend = addend;
	}
}

```

```java
class Money{
	Expression plus(Money addend){
		return new Sum(this, addend);
	}
}
//위 코드를 다음과 같이 Expression 들로 변경한다.
class Money{
	Expression plus(Expression addend){
		return new Sum(this, addend);
	}
}

```

또한 15장 시작에 작성했던, 테스트코드를 실행하면 fiveBucks.plus를 실행할 때, Expression에 plus() 가 정의되지 않았다는 문제가 발생한다.

따라서 Expression에 plus를 정의해주고 이를 상속하는 Money와 Sum에 구현을 적는다.

```java
class Expression{
	Expression plus(Expression addend);
}

class Money{
	public Expression plus(Expression addend){
		return new Sum(this, addend);
	}
}

//일단 sum의 plus는 할 일 목록에 적어둠
class Sum{
	public Expression plus(Expression addend){
		return null;
	}
}

```