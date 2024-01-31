# 28장. 초록 막대 패턴

### 가짜로 구현하기(진짜로 만들기 전까지만)

- 실패하는 테스트를 만든 후 첫 번째 구현을 어떻게 하는게 좋을까?
    1. 상수를 반환하게 한다.
    2. 테스트가 통과할 시, 단계적으로 상수를 변수를 사용하는 수식으로 변형한다.


예제)

```jsx
1. return "1 run, 0 failed"
2. return "%d run, 0 failed" % self.runCount
3. return "%d run, %d failed" ^ (self.runCount, self failureCount)
```

- 왜 코드를 들어낼 걸 알면서도 가짜 구현을 해야할까?
    - 가짜 구현을 해봄으로써 테스트가 잘못 작성되었다는 것을 알 수 있다.
    - 막대가 초록색일 때, 우리는 확신을 갖고 거기부터 리팩토링해 갈 수 있다.
    - 다음 테스트 케이스를 구현할 때, 이전 테스트의 작동이 보장된다는 것을 알기 때문에 그 다음 테스트 케이스에도 집중할 수 있다.

- 가짜 구현 코드는 필요 없는 코드를 작성하지 말라는 법칙에 위배되지 않는다.
    - 리팩토링 단계에서 테스트 케이스와 코드 간의 데이터 중복을 제거하기 때문

예제)

```jsx
assertEquals(new MyDate("28.2.02"), new MyDate("1.3.02").yesterday());
```

```jsx
public MyDate yesterday() {
	return new MyDate("28.2.02");
}
```

- 테스트와 코드 간 중복을 제거하자.

```jsx
public MyDate yesterday() {
	return new MyDate(new MyDate("1.3.02").days() - 1);
}
```

- this가 `MyDate(”1.3.02”)`와 같기에, 데이터 중복을 마저 제거한다.

```jsx
public MyDate yesterday() {
	return new MyDate(this.days() - 1);
}
```

### 삼각측량

- 오로지 예가 두 개 이상일 때에만 추상화를 하자.

예제) 두 정수의 합을 반환하는 함수

```jsx
private int plus(int augent, int addend) {
	return 4;
}
```

삼각측량 사용:

```jsx
public void testSum() {
	assertEquals(4, plus(3,1));
	assertEquals(7, plus(3,4));
}
```

두번 째 예를 얻었을 때 plus()의 구현을 추상화할 수 있다.

```jsx
private int plus(int augent, int addend) {
	return augend + addend;
}
```

- 삼각측량의 명확한 규칙은 매력적이지만, 어떤 계산을 해야 올바르게 추상화할 것인지에 대해 정말 감잡기 어려울 때만 사용하자.
- 그 외의 경우, 명백한 구현이나 가짜로 구현에 의존하자.

### 명백한 구현

- 뭘 타이핑 해야할지 알고, 그걸 재빨리 할 수 있다면 그냥 해버리자.
- 하지만 만약 그 코드가 테스트를 통과시키는 가장 단순한 변경이 아니라면?
    - “제대로 동작하는”을 푸는 동시에 “깨끗한 코드”를 해결하려는 것은 한번에 하기엔 너무 많은 일이다.
    - 우선 “제대로 동작하는”으로 되돌아가 그걸 해결하고, 그 후에 “깨끗한 코드”를 해결하자.

### 하나에서 여럿으로

객체 컬렉션을 다루는 연산 구현하기
- 우선 컬렉션 없이 구현 후 컬렉션을 사용하게 하자.

예제) 숫자 배열의 합을 구하는 함수

```jsx
public void testSum() {
	assertEquals(5, sum(5));
}

private int sum(int value) {
	return value;
}
```

- 우선 `sum()`에 배열을 받아들이는 인자를 하나 추가하자.

```jsx
public void testSum() {
	assertEquals(5, sum(5, new int[] {5}));
}

private int sum(int value, int[] values) {
	return value;
}
```

- 이 단계는 **변화 격리하기**의 예로 볼 수 있다.
- 테스트 케이스에 인자를 추가하면 테스트 케이스의 영향을 주지 않으면서 자유로이 구현을 변경할 수 있다.

```jsx
private int sum(int value, int[] values) {
	int sum = 0;
	for (int i = 0; i < values.length; i++) 
		sum += values[i];
	return sum;
}
```

- 이제 컬렉션을 사용하고, 사용하지 않는 단일 인자를 삭제할 수 있다,

```jsx
public void testSum() {
	assertEquals(5, sum(new int[] {5}));
}

private int sum(int[] values) {
	int sum = 0;
	for (int i = 0; i < values.length; i++) 
		sum += values[i];
	return sum;
}
```

- 이 단계 역시 변화 격리하기의 예로, 코드를 고쳐서 테스트 케이스를 바꿔도 코드에 영향이 없도록 했다.

```jsx
public void testSum() {
	assertEquals(12, sum(new int[] {5,7}));
}
```

- 이제 계획대로 테스트 케이스를 개선할 수 있다.

# 29장. xUnit 패턴

### 단언(assertions)

- 테스트가 잘 작동하는지 검사하는 방법
- boolean 수식을 작성해 프로그램이 자동으로 코드 동작 여부 판단을 수행하도록 하게 하자.
    - 판단 결과가 boolean 값이어야 한다. 참 값: 모든 테스트 통과를 의미, 거짓 값은 예상치 못했던 일이 발생했음을 의미한다.
    - 이 boolean 값은 컴퓨터에 의해 검증되어야 한다. 다양한 형태의 assert() 메서드를 호출해 이 값을 얻어낸다.

→ 단언은 구체적이어야 한다.

예제)

- `assertTrue(rectangle.area ≠ 0)` 는 0이 아닌 아무 값이나 반환하게 만들어도 조건을 만족하므로 유용하지 않다.
- 면적이 50이어야 한다면, 단언을 `assertTrue(rectangle.area() == 50)` 과 같이 구체적으로 적어주자.

예제 2) 청약됨(Offered) 또는 시행중(Running) 이라는 상태(Status)를 가질 수 있는 계약(Contract) 에 대한 테스트.

```jsx
Contract contract = new Contract(); //default로 Offered status
contract.begin(); //Running으로 status 변경
assertEquals(Running.class, contract.status.class);
```

- status에 대한 현재 구현에 너무 의존적인 테스트인걸 볼 수 있다.
- status가 boolean 값으로 표현되도록 구현이 바뀌더라도 테스트가 통과할 수 있어야한다.
- status가 바뀐다면 시행일이 언제인지 알아낼 수 있을 것이기에, 이를 반영해 테스트를 수정하자.

```jsx
assertEquals(..., contract.startDate()); //status가 Offered라면 예외
```

### 픽스처

→ 여러 테스트에서 공통으로 사용하는 객체들을 생성하는 방법

1. 각 테스트 코드에 있는 지역 변수를 인스턴스 변수로 바꾸고
2. setUp() 메서드를 재정의하여 이 메서드에서 인스턴스 변수들을 초기화한다.
- 우리는 모델 코드 (테스트 대상이 되는, 실제 애플리케이션 production code) 에서 중복을 제거하길 원한다. 그렇다면 테스트코드에서도 중복을 없애야한다.

→ 객체들을 세팅하는 코드가 여러 테스트에 걸쳐 중복된다면?

단점

- 동일 코드 반복 작성 시간 소요
- 인터페이스를 수동으로 변경할 경우, 여러 테스트를 고쳐주어야 한다.

장점

- 객체 세팅 코드들이 단언이 적힌 메서드에 포함될 경우, 테스트 코드를 그냥 위에서 아래로 읽어내려 갈 수 있다.
- 하지만, setUp 메서드로 분리된다면 테스트의 나머지 부분을 작성하기 전에, 그 메서드가 자동으로 호출된다는 점과 객체들이 어떻게 초기화되었는지를 기억해야 한다.

두 가지 방법

1. 픽스처를 생성하는 코드를 테스트 메서드에 포함시키기
2. 공통의 픽스처 생성 코드를 setUp() 메서드로 추출하기 (프레임워크에 의해 각 테스트 메서드 호출 직전에 호출)

예제

1. 테스트 메서드에 포함

```jsx
EmptyRectangleTest
public void testEmpty() {
	Rectangle empty = new Rectangle(0,0,0,0);
	assertTrue(empty.isEmpty());
}

public void testWidth() {
	Rectangle empty = new Rectangle(0,0,0,0);
	assertTrue(0.0, empty.getWidth(), 0.0);
}
```

1. setUp 메서드 사용

```jsx
EmptyRectangleTest

private Rectangle empty;

public void setUp() {
	empty = new Rectangle(0,0,0,0);
}

public void testEmpty() {
	assertTrue(empty.isEmpty());
}

public void testWidth() {
	assertTrue(0.0, empty.getWidth(), 0.0);
}
```

- 장단점을 잘 따져보고 둘 다 시도해보자.

### 외부 픽스처

- 픽스처 중 외부 자원이 있을 경우 어떻게 해제해야 할까?

  → tearDown() 메서드를 재정의하여 이곳에서 자원을 해제하자.

    - 테스트의 목적 중 하나는 테스트가 실행되기 전과 실행된 후의 외부 세계가 동일하게 유지되도록 만드는 것이기 때문이다.


예제

```jsx
testMethod(self):
	file = File("foobar").open();
	try: 
		..run the test..
	finally:
		file.close()
```

- 만약 파일이 여러 테스트에서 사용되었다면, 공통 픽스처의 일부로 만들 수 있을 것이다.

```jsx
setUp(self):
	self.file = File("foobar").open()
testMethod(self):
	try:
		..run the test..
	finally:
		self.file.close()
```

- 하지만 finally 절의 중복은 설계에서 우리가 놓치고 있는 점을 알려준다.
- xUnit은 각 테스트가 끝난 후에 tearDown()이 호출되는 것을 보장한다. 이는 테스트 메서드에서 무슨 일이 발생하던 상관없이 호출되기에, 아래와 같이 적을 수 있다. (setUp() 실패 경우 제외)

```jsx
setUp(self):
	self.file = File("foobar").open()
testMethod(self):
	..run the test..
tearDown(self):
		self.file.close()
```

### 테스트 메서드

- 테스트 케이스를 어떻게 표현할 것인가? → ‘test’로 시작하는 이름의 메서드로 나타내자.
- 만약 이런 테스트가 수백, 수천개라면?

관례

- 동일한 픽스처를 공유하는 모든 테스트는 동일한 클래스의 메서드로, 다른 종류는 다른 클래스에 존재한다.
- 관습에 의해 메서드 이름은 ‘test’로 시작한다.
    - xUnit 프레임워크가 이 패턴을 자동으로 인식하고 주어진 클래스에 대한 테스트 슈트를 생성하기 때문이다.
- 메서드 이름은 나중에 아무것도 모르는 사람이 와도 테스트 작성의 이유를 알 수 있도록 단서를 주어야 한다. (예시: testAsssertPosInfinityNotEqualsNegInfinity)
- 테스트 메서드는 의미가 그대로 드러나는 코드로, 읽기 쉬워야한다.
    - 테스트 코드가 점점 길어지고 복잡해진다면, 가장 짧은 테스트를 작성하자.
- 원하는 테스트에 대한 짧은 아웃라인을 작성해보자.



### 예외 테스트

- 예외가 발생하는 것이 정상인 경우에 대한 테스트를 작성하는 법
    - 예상되는 예외를 잡아서 무시하고,
    - 예외가 발생하지 않은 경우에 한해 테스트가 실패하게 만들자.

예제

```jsx
public void testRate() {
	exchange.addRate("USD", "GBP", 2);
	int rate = exchange.findRate("USD", "GBP");
	assertEquals(2,rate);
```

예외가 제대로 발생하는지 테스트 해보자.

```jsx
public void testMissingRate() {
	try {
		exghange.findRate("USD", "GBP");
		fail();
	} catch (IllegalArgumentException expected) {
	}
}
```

- findRate() 가 예외를 던지지 않는다면 테스트를 실패했음을 알려주는 fail() 이 호출된다.
- 원하는 정확한 종류의 예외만을 잡아내야 한다는 점에 유의하자.

### 전체 테스트

- 모든 테스트를 한번에 실행하려면?
    - 모든 테스트 슈트에 대한 모음을 작성하자. (각각의 패키지에 대해 하나씩, 그리고 전체 애플리케이션의 패키지 테스트를 모아주는 테스트 슈트)
    - TestCase의 하위 클래스 하나를 패키지에 추가하고, 그 클래스에 대해 테스트 메서드를 하나 추가한다고 가정했을 때, 다음에 테스트를 전부 돌릴 때 그 테스트 메서드도 실행되어야 한다.
    - 이를 위해 각각의 패키지는 AllTests 라는 이름의 클래스를 가지고 있어야 하고, 이 클래스는 TestSuite 인스턴스를 반환하는 suite() 라는 이름의 정적 메서드를 가지고 있어야 한다.  (지금은 IDE에서 지원한다.)