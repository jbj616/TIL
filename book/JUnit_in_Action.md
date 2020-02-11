## 책 제목 : JUnit in Action

참고 도서 : [JUnit in Action](https://www.aladin.co.kr/shop/wproduct.aspx?ItemId=12075966)



### 목차

- [01-Intro](#01.Intro)



---

### 01.Intro

- 단위 테스트 : 명확한 작업 단위의 동작을 검사
- 통합 테스트나 인수 테스트는 다양한 컴포넌트 사이의 상호작용을 검증



```java
public class Calculator {
    public double add(double a, double b) {
        return a + b;
    }
}
```

- 문제가 있을까?
  - add 연산은 아무리 단순해도 프로그램의 기능임 -> 간접적으로 다른 메서드에게 영향을 줄 수 있음.
  - 애플리케이션이 수정될 때마다 기능의 동작 여부를 체크할 필요가 있음.



#### 단위 테스트 프레임 워크

규칙

1. 단위 테스트는 다른 모든 단위 테스트들과 **독릭적으로** 실행
2. 테스트 각각의 오류를 식별하고 보고해야함
3. 어떤 테스트를 실행할지 선택하기 쉬워야함



#### JUnit의 설계 목표

- 유용한 테스트를 작성하는 데 보탬이 되어야함
- **시간이 지나도 가치가 변치 않는 테스트**를 작성하는 데 보탬이 되어야 함
- 코드 재사용을 통해 테스트 작성 비용을 낮추는 데 보탬이 되어야 함



Junit을 적용한 Calculator Test

```java
public class CalculatorTest {

    @Test
    public void testAdd() {
        Calculator calculator = new Calculator();
        double result = calculator.add(10, 50);
        assertEquals(60, result, 0);//허용 오차
    }
}
```

- 테스트 메서드는 test_명으로 작성 권장

- assertEquals 

  ```java
  static public void assertEquals(double expected, double actual, double delta)//delta = 오차 범위
  ```



### 02.Junit 핵심

테스트 메서드 조건

- `@Test` 에노테이션 부여
- public 메서드
- void 반환형 메서드



Junit이 `@Test` 메서드를 호출 -> 독립된 메모리 공간에서 실행 

- Why? 혹시 모를 부작용을 방지



다양한 assert 메서드들

|   Assert 메서드    |               목적                |
| :----------------: | :-------------------------------: |
| assertArraysEquals |          배열 일치 여부           |
|    assertEquals    |   객체 일치 여부(equals로 판단)   |
|     assertSame     | 동일한 객체인가 (하나의 객체인가) |
|     assertTrue     |               True                |
|   assertNotNull    |             not null              |



- 테스트 클래스 - @Test가 부여된 테스트를 하나 이상 포함한 클래스
- 스위트 - 테스트들의 집합. 
- 러너 - 테스트 스위트 실행 엔진. 



#### 파라미터화 테스트 실행

하나의 테스트를 여러 번 반복 실행하는 기능

```java
@RunWith(value = Parameterized.class)
public class ParameterizedTest {

    private double expected;
    private double valueOne;
    private double valueTwo;

    @Parameters
    public static Collection<Integer[]> getTestParameters() {
        return Arrays.asList(new Integer[][]{
            {2, 1, 1},
            {3, 2, 1},
            {4, 3, 1}
        });
    }

    public ParameterizedTest(double expected, double valueOne, double valueTwo) {
        this.expected = expected;
        this.valueOne = valueOne;
        this.valueTwo = valueTwo;
    }

    @Test
    public void sum() {
        Calculator calc = new Calculator();
        assertEquals(expected, calc.add(valueOne, valueTwo), 0);
    }

}
```

- `@RunWith(value = Parameterized.class)`  -> `@Parameters` 를 가진 메서드 필요 -> 파라미터로 받을 생성자 필요함 -> test 실행

