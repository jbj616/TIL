## 책 제목 : JUnit in Action

참고 도서 : [JUnit in Action](https://www.aladin.co.kr/shop/wproduct.aspx?ItemId=12075966)



### 목차

- [01-Intro](#01.Intro)



---

###01.Intro

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