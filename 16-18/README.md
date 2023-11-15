# 16-18

## 16장. 드디어, 추상화

1. Sum.plus()
2. Expression.times()
3. \$5 + \$5 에서 Money 반환하기

### Sum.plus()

```java
public void testSumPlusMoney() {
    Expression fiveBucks = Money.dollar(5);
    Expression tenFrancs = Money.franc(10);

    Bank bank = new Bank();
    bank.addRate("CHF", "USD", 2);

    Expression sum = new Sum(fiveBucks, tenFrancs).plus(fiveBucks);
    Money result = bank.reduce(sum, "USD");

    assertEquals(Money.dollar(15), result);
}
```

- 명시적 Sum 생성이 fiveBucks.plus(tenFrancs) 보다 의도를 조금 더 잘 드러냅니다.

```java
// Sum
public Expression plus(Expression addend) {
    return new Sum(this, addend);
}
```

- 코드 중복 (멀리서 추상 클래스의 외침이...)

중간정리

- TDD에서는 테스트 코드의 줄 수와 모델 코드 줄 수가 거의 비슷하게 끝납니다.
- TDD가 경제적이려면 코드 생산량이 2배 늘거나, 동일한 기능을 절반의 코드로 구현해야 합니다.
- 자신의 방법과 TDD가 어떻게 다른지 측정해야 합니다.
- 또한 디버깅, 통합 작업, 설명에 걸리는 시간 등을 모두 포함해야 합니다.
- 작성자: 단순하게 보면, 테스트 코드까지 포함하면 작성해야 할 코드의 양이 2배가 된 셈입니다. 따라서 TDD가 효과적이려면 2배의 코딩 속도를 내거나, 절반의 코드로 동일한 기능을 구현해야 합니다. 이를 만족해야만 TDD가 경제적임을 정당화할 수 있는 것이죠. 따라서 어떤 태스크를 TDD없이 수행해보고, 같은 태스크를 TDD를 적용해 한 번 더 수행해봅시다. 이때 시간 등을 측정하여, 2배 혹은 절반 지표를 만족하는지 평가해봅시다. 만약 그렇지 않다면, 저희가 TDD를 100% 발휘하는 것이 아니라는 의미입니다. 따라서 이를 어떤 부분이 다르거나 약점인지 파악해 개선할 기회로 삼으면 됩니다.

### Expression.times()

```java
public void testSumTimes() {
    Expression fiveBucks = Money.dollar(5);
    Expression tenFrancs = Money.franc(10);

    Bank bank = new Bank();
    bank.addRate("CHF", "USD", 2);

    Expression sum = new Sum(fiveBucks, tenFrancs).times(2);
    Money result = bank.reduce(sum, "USD");

    assertEquals(Money.dollar(20), result);
}
```

- JUnit을 잘 안다면 테스트가 코드보다 긴 현상을 고칠 수 있습니다. (29장의 픽스처 Fixture)

```java
// Expression
Expression times(int multiplier);
```

- 이미 덧셈쪽에서 인자들을 Expression 추상화했었죠. 여기에도 비슷하게 했습니다.

```java
// Sum
public Expression times(int multiplier) {
    return new Sum(augend.times(multiplier), addend.times(multiplier));
}

// Money
public Expression times(int multiplier) {
    return new Money(amount * multiplier, currency);
}
```

- 위의 추상화를 적용하기 위해 가시성을 높였습니다.
- 작성자: visibility를 public으로 바꿨다는 의미 같네요.

### \$5 + \$5 에서 Money 반환하기

```java
public void testPlusSameCurrencyReturnsMoney() {
    Expression sum = Money.dollar(1).plus(Money.dollar(1));
    assertTrue(sum instanceof Money);
}
```

- 이 리팩토링은 실험입니다. 되면 좋고 안 되면 그만둡시다.
- 반환값이 Money라는 것을 테스트합니다.
- 구현 중심이기 때문에 조금 지저분하지만, 저희가 원하는 변화를 적용할 수 있게 합니다.

```java
// Money
public Expression plus(Expression addend) {
    return new Sum(this, addend);
}
```

- 필요충분조건으로 인자가 Money인 경우에만 통화를 확인하는 깔끔한 방법이 없네요.
- 실험은 실패했습니다.
- 코드를 되돌리고, 어차피 지저분하다고 생각했던 테스트를 지우고 떠났습니다.

### 16장 정리

- 미래에 코드를 읽을 다른 사람을 염두한 코드
- TDD와 저희의 현재 개발 스타일을 비교하는 실험 방법
- 또 한 번 선언부에 대한 수정이 시스템 나머지 부분으로 퍼졌고, 컴파일러의 조언을 따랐습니다.
- 실험을 해봤는데 제대로 안 돼서 버렸습니다.

## 17장. Money 회고

즐거운 회고 시간!

### 다음에 할 일은 무엇인가

Sum.plus()와 Money.plus() 사이의 지저분한 중복

- Expression을 인터페이스 대신 클래스로 바꾼다면 공통 코드를 담을 수도 있습니다. 다만 일반적인 변경 방향은 아닙니다.

"끝났다"는 말을 믿지 않습니다.

- 테스트의 장점: 시스템이 클 수록 건드리는 부분이 견고함을 보장해줍니다. 그러면 안심하고 고칠 수 있습니다.
- 자동화된 프로그램: 작업이 끝난 후 SmallLint 같은 코드 감정(code critic) 툴을 돌려봅시다. 자잘한 사항을 신경쓰지 않아도 알아서 알려주므로, 그것들을 신경쓰며 개발해야 하는 스트레스를 받지 않게 됩니다.
- "어떤 테스트들이 추가로 더 필요할까?"
- 의도와 결과가 다른 테스트는 이유를 찾아내야 합니다.
- 실패해야 하는 테스트는 시스템의 제한사항 또는 해야 할 작업 등의 문서처럼 사용할 수 있습니다.
- 할 일 목록이 빌 때가 설계한 것을 검토(리팩토링)하기 좋은 시기입니다.

### 메타포

- Money 예제 결과가 과거의 작업물들과 달랐습니다.
- 생각한 메타포의 설계가 기존과 완전히 다른 방향으로 흘러갔습니다.

작성자: TDD의 프로세스 주기가 짧은 덕분에 코드, 설계, 아이디어 등에 대한 성찰과 새 방향 설정이 매우 빈번하고 쉽습니다. 이러한 점을 설명하려는 것이 아닌가 싶습니다.

<!-- TODO -->

### JUnit 사용도

- JUnit 로그를 기록해두었는데, 실행 버튼을 125번 눌렀고, 보통 1분에 한 번 실행했습니다.
- 조급하게 리팩토링하면 정말 가끔 발생하는 성공 혹은 실패를 봤습니다.

### 코드 메트릭스

- 코드와 테스트 사이에 대략 비슷한 양의 함수와 줄이 있습니다.
- 테스트 코드 분량은 공통된 테스트 픽스쳐를 뽑아내는 작업으로 줄일 수 있습니다.
- 회기성 복잡도(cyclomatic complexity)는 기존의 흐름 복잡도(flow complexity)와 같습니다. 테스트 코드에 분기나 반복문이 전혀 없기 때문에 테스트 복잡도가 1입니다. 흐름 제어 대신 다형성(polymporphism)을 사용했기 때문에 코드의 복잡도도 낮습니다.
- 함수 시그니처와 닫는 괄호도 포함된 줄 수입니다.
- 테스트의 줄 수가 과정되었는데, 픽스처 구축 코드가 없기 때문입니다.

### 프로세스

TDD의 주기

1. 작은 테스트를 추가합니다.
2. 모든 테스트를 실행하고, 실패하는 것을 확인합니다.
3. 코드를 수정합니다.
4. 모든 테스트를 실행하고, 성공하는 것을 확인합니다.
5. 중복을 제거하기 위해 리팩토링합니다.

리팩토링당 수정 횟수

- 수정: 메서드나 클래스 정의 변경
- TDD를 통해 각 리팩토링의 수정 횟수는 적게 유지됩니다.

작성자: 각 주기마다 수정 횟수가 적을수록 거대한 변경을 최소화할 수 있게 됩니다. 이로 인해 대규모 변경이 야기하는 온갖 문제가 확연히 줄어드는 것이 TDD의 이점인 것 같습니다.

<!-- TODO -->

### 테스트의 질

TDD로 만드는 테스트들은 시스템의 수명주기동안 지속돼야 할 정도로 유용하지만, 다음을 대체할 수는 없습니다.

- 성능 테스트
- 스트레스 테스트
- 사용성 테스트

테스트는 감시의 역할을 넘어서, 의사소통의 도구로 쓰일 수 있습니다.

TDD 테스트의 지표들

- 명령문 커버리지(statement coverage)
  - 테스트의 시작점입니다. TDD는 무조건 100%.
  - JProbe 등
- 결함 삽입(defect insertion)
  - 코드의 의미를(내용을) 변경했을 때 테스트가 실패하는지 봅니다.
  - `hashCode() { return 0; }` 기억하실까요? 0이 다른 값으로 바뀌어도 테스트는 여전히 성공할 것입니다.
  - 수동으로 할 수도, Jester를 쓸 수도

테스트 커버리지에 대해

- 테스트 커버리지를 늘리려면, 테스트 수를 늘리면 됩니다.
  - 전문 테스터는 TDD 테스터보다 10배 많은 테스트를 만들기도 합니다.
  - 32장을 참조해주세요.
- 또는 테스트의 수를 그대로 두고, 로직을 단순하게 만들면 됩니다.
- 저희는 후자를 선호합시다.

### 최종 검토

- 테스트를 확실히 돌아가게 만드는 세 가지 접근법
  - 가짜로 구현하기
  - 삼각측량법
  - 명백하게 구현하기
- 설계를 주도하기 위한 방법: 테스트 코드와 실제 코드 사이의 중복 제거하기
- 테스트 사이의 간격을 조절하는 능력: 필요에 따라 속도를 줄이거나 높이기

## 2부: xUnit 예시

- TDD를 통해 '실제' 소프트웨어를 만드는 발전된 예제
- 자기 참조(self-referential) 프로그래밍에 대한 전산학 실습

## 18장. xUnit으로 가는 첫걸음

Task: 테스트 프레임워크 만들기!

- 부트스트랩 문제: 테스트 프레임워크를 테스트하기 위한 테스트 케이스? (메타-테스트라고 합시다.)

테스트 프레임워크 개발을 위한 TODO

> - [ ] **테스트 메서드 호출하기**
> - [ ] 먼저 setUp 호출하기
> - [ ] 나중에 tearDown 호출하기
> - [ ] 테스트 메서드가 실패하더라도 tearDown 호출하기
> - [ ] 여러 개의 테스트 실행하기
> - [ ] 수집된 결과를 출력하기

- 물론 메타-테스트를 먼저 만듭니다.
- 저희가 만들 테스트 프레임워크는 원시적인 형태로 플래그 세우기 및 출력으로 결과를 뱉습니다.

### 첫 번째 메타-테스트

```python
test = WasRun("testMethod")
print test.wasRun
test.testMethod()
print test.wasRun
```

- 갑분Python2
- 테스트 먼저입니다. 아직 WasRun을 안 만들었죠.
- 기대: 첫 출력은 None, 두번째 출력은 1이 나와야 합니다.
- testMethod()는 하드코딩된 임시 테스트 케이스입니다. 추후 유저가 직접 작성할 수 있도록 일반화하기 전에, 일단 하드코딩된 작은 것부터 하는 것입니다.

WasRun 정의

```python
class WasRun:
    pass
```

- pass!

생성자와 `wasRun` 필드, `testMethod` 메서드 추가

```python
class WasRun:
    def __init__(self, name):
        self.wasRun = None

    def testMethod(self):
        pass
```

- Trivial: 테스트 코드를 먼저 작성하면 그에 맞는 스텁(signature 등)을 알아서 만들어주는 툴이 있으면 어떨까요?

테스트가 기대대로 되도록 `testMethod` 수정

```python
class WasRun:
    def __init__(self, name):
        self.wasRun = None

    def testMethod(self):
        self.wasRun = 1
```

- 초록 막대!
- 물론 리팩토링 할 게 많습니다.

### 두 번째: `testMethod()` 대신 `run()`을 인터페이스로 노출시키기

```python
test = WasRun("testMethod")
print test.wasRun
test.run()
print test.wasRun
```

```python
class WasRun:
    def __init__(self, name):
        self.wasRun = None

    def run(self):
        self.testMethod()

    def testMethod(self):
        self.wasRun = 1
```

중간정리

- 리팩토링 시, 두 부분을 서로 분리해서 각각 따로 작업할 수 있습니다. (run 메서드 추가와 testMethod 리팩토링은 독립적으로 진행될 수 있었겠죠.)
- 작업이 끝날 때 하나로 돌아가도 좋고, 혹은 분리된 상태로 놔둘 수도 있습니다.

### 세 번째: `testMethod()` 동적 호출하기

```python
class WasRun:
    def __init__(self, name):
        self.wasRun = None
        self.name = name

    def run(self):
        method = getattr(self, self.name)
        method()
```

- 아니 이게 머선
- 하나의 특별한 사례에 대해서만 작동하는 코드를, 상수(`testMethod`)를 변수로 변화시켜 일반화했습니다.
- 상수가 하드코딩된 값이었더라도 원리는 같습니다.
- 이제 WasRun은 독립된 두 가지 일을 합니다.
  - 테스트 결과를 기록하는 일
  - 테스트 메서드를 동적으로 호출하는 일

### 네 번째: `TestCase` 상속하기

```python
class TestCase:
    pass

class WasRun(TestCase): ...
```

- name을 먼저 올립니다.
- run()은 상위 클래스의 속성만 사용하므로 이것도 옮깁시다. 오퍼레이션은 항상 데이터 근처에 있으면 좋습니다.

```python
class TestCase:
    def __init__(self, name):
        self.name = name

    def run(self):
        method = getattr(self, self.name)
        method()
```

### 다섯 번째: 처음 만든 테스트 코드를 TestCase로 옮기기

```python
class TestCaseTest(TestCase):
    def testRunning(self):
        test = WasRun("testMethod")
        assert(not test.wasRun)
        test.run()
        assert(test.wasRun)

TestCaseTest("testRunning").run()
```

- 이제 결과를 눈으로 매번 확인하지 않아도 됩니다.

<!-- TODO -->

### 18장 정리

- 시행착오 끝에 자기 과신을 인지하고, 아주 자그마한 단계로 시작하는 법을 알아냈습니다.
- 일단 하드코딩한 다음, 그것을 변수로 대체하여 일반화하는 방식으로 기능을 구현했습니다.
- 플러거블 셀렉터(pluggable selector)를 사용하여 테스트 메서드를 동적으로 호출했습니다. 정적 분석이 어려워지므로, 쿨타임 4개월형에 처합니다.
- 저희의 테스트 프레임워크를 작은 단계로만 부트스트랩했습니다.
