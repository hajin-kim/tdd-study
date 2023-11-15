# 04-06

## 4. 프라이버시

Dollar의 amount 를 private 로 만들기

```
public void testMultiplication(){
	Dollar five = new Dollar(5);
	Dollar product = five.times(2);
	assertEquals(10, product.amount);
	product = five.times(3);
	assertEquals(15, product.amount)
}

```

Dollar.times() 연산은 호출을 받은 객체의 값에 인자로 받은 값 만큼 곱한 값을 갖는 "Dollar 객체" 를 반환하지만 테스트에서는 product.amount의 값에 대해서만 테스트를 한다.

```
public void testMultiplication(){
	Dollar five = new Dollar(5);
	assertEquals(new(Dollar(10)), five.times(2));
	assertEquals(new(Dollar(15)), five.times(3));
}

```

assert에서 Dollar 객체끼리 비교하도록 만든 다음, product 변수를 없애고 인라인 시켰다.

따라서 Dollar 내의 amount 변수를 private 으로 수정할 수 있게 되었다.

```
private int amount;

```

## 5. 솔직히 말하자면 (Frac-ly Speaking)

**테스트 코드의 작성 단계**

1. 테스트 작성
2. 컴파일 되게 하기
3. 실패하는지 확인하기 위해 실행.
4. 실행하게 만듦
5. 중복 제거

반드시 위 5단계 사이클이 모두 실행되어야하며, 다섯 번째 단계가 생략되어서도 안된다.

1. 테스트 작성
일단 Dollar 의 테스트를 복사해서 Franc 객체가 만들어졌을 때 통과하기를 기대하는 테스트를 만든다.

```
public void testFrancMultiplication(){
	Franc five = new Franc(5);
	assertEquals(new(Franc(10)), five.times(2));
	assertEquals(new(Franc(15)), five.times(3));
}

```

1. 컴파일 되게 하기

```
class Franc{
	private int amount;

	Franc(int amount){
		this.amount = amount;
	}

	Franc times(int multiplier){
		return new Franc(amount*multiplier)
	}

	public boolean equals(Object object){
		Franc franc = (Franc) object;
		return amount == franc.amount;
	}
}

```

위 단계에서는 equals, times 문 등이 Dollar 객체와 서로 중복이 되고 있다.
중복코드에 대한 내용은 TDD 5단계 중 마지막 다섯번째 단계를 통해 해결한다.

정리하자면,

- 큰 테스트를 공략할 수 없다. Divide and conquer 를 하자.
- 일단 중복코드를 신경쓰지 않고 때려 박는다.
- 다만 중복이 사라지기 전까지는 집에 가지 않는다.

## 6. 돌아온 '모두를 위한 평등'

5장에서 만든 Dollar 와 Franc 사이의 중복을 제거해야한다.

**방법**

1. ~~Dollar 와 Franc 둘 중 하나의 클래스가 다른 하나를 상속 받도록 만든다.~
2. 두 클래스의 공통 상위 클래스를 만든다.

당연하지만 두 번째 방법이 더 이상적이다.

1. Money class 만들기

```
class Money{}

```

1. Dollar class 에 상속 시키고 amount 를 Money class로 빼기

```
class Dollar extends Money{}

class Money{
	protected int amount;
}

```

1. Dollar 의 equals 부분에서 Money로 타입을 받아오기

```
class Dollar extends Money{

	public boolean equals (Object object){
		Money money = (Money) object;
		return amount == money.amount;
	}
}

```

1. 메서드를 Money class 로 옮기기

```
class Money{
	protected int amount;

	public boolean equals (Object object){
		Money money = (Money) object;
		return amount == money.amount;
	}
}

```

1. Franc class 끼리 비교하는 테스트 코드에 대해서 작성하기 (코드를 복붙 했었기 때문에 해당 부분이 생략되어있었다.)

```
public void testEquality(){
	assertTrue(new Dollar(5).equals(new Dollar(5)));
	assertFalse(new Dollar(5).equals(new Dollar(6)));
	assertTrue(new Franc(5).equals(new Franc(5)));
	assertFalse(new Franc(5).equals(new Franc(6)));
}

```

1. 코드에 중복이 있는걸 확인했으므로 Dollar class 가 했던 것과 같은 process로 Franc class 에서 equals 메소드 지우고 Money 상속시키기

```
class Franc extends Money{}

```

정리하자면,

- 공통된 코드를 첫번째 class Dollar에서 상위 class Money로 단계적으로 옮겼다.
- 두번째 class Franc 도 Money의 하위 클래스로 만들었다.
- 불필요한 구현을 제거하기 전에 두 equals() 구현을 일치시켰다.