## 책 제목 : JUnit in Action

참고 도서 : [JUnit in Action](https://www.aladin.co.kr/shop/wproduct.aspx?ItemId=12075966)



### 목차

- [01-Intro](#01Intro)
- [02-JUnit핵심](02Junit 핵심)
- [03-JUnit마스터하기](03.JUnit 마스터하기)



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



### 03.JUnit 마스터하기

#### 예제를 통한 애플리케이션 구현

1. 인터페이스 설계

   ```java
   public interface Request {
       String getName();
   }
   
   public interface Response {
   
   }
   
   public interface RequestHandler {
   
       Response process(Request request) throws Exception;
   }
   
   public interface Controller {
   
       Response processRequest(Request request);
   
       void addHandler(Request request, RequestHandler requestHandler);
   }
   ```

   - RequestHandler : 궃은 일을 도맡아 처리해주는 목적

     > 제어 구조 역전
     >
     > 이벤트가 발생했을 때 처리할 핸들러 객체를 등록해두고, 그 이벤트가 발생하면 등록되어 있던 객체의 특정 메서드를 호출하는 방식.
     >
     > 제어 구조의 역전을 통해 개발자들은 프레임워크가 관리하는 이벤트 생명주기에 대해 신경쓸 필요 없이 원하는 이벤트 처리를 위한 자기만의 핸들러를 끼워 넣을 수 있음.

2. 기반 클래스  구현

   ```java
   public class DefaultController implements Controller {
   
       private Map requestHandlers = new HashMap();
   
       protected RequestHandler getHandler(Request request) {
           if (!this.requestHandlers.containsKey(request.getName())) {
               String message = "Cannot find handler for request name " + "[" + request.getName() + "]";
               throw new RuntimeException(message);
           }
           return (RequestHandler) this.requestHandlers.get(request.getName());
       }
   
       @Override
       public Response processRequest(Request request) {
           Response response;
           try {
               response = getHandler(request).process(request);
           } catch (Exception e) {
               response = new ErrorResponse(request, e);
           }
           return response;
       }
   
       @Override
       public void addHandler(Request request, RequestHandler requestHandler) {
           if (this.requestHandlers.containsKey(request.getName())) {
               throw new RuntimeException(
                   "A request handler has already been registred for request name [" + request.getName() + "]");
           } else {
               this.requestHandlers.put(request.getName(), requestHandler);
           }
       }
   }
   ```

   - 요청에 대한 핸들러 반환을 위한 `getHandler()` 구현
   - 요청에 적절한 핸들러 전달 후 응답값 반환 하는 `processRequest()` 구현

3. 컨트롤러 테스트

   ```java
   private DefaultController controller;
   
   @Before
   public void instantiate() throws Exception {
       controller = new DefaultController();
   }
   ```

   - `@Before` 테스트 메서드 사이에서 호출 됨. (테스트 후 `@After`)
     - 테스트 메서드를 합치지 않고  테스트 사이에서 픽스쳐를 공유할 수 있도록 하기 위함.

4. 핸들러 추가하기

   RequestHandler가 작동하는 지 테스트

   ```java
   private class SampleRequest implements Request {
   
       public String getName() {
           return "test";
       }
   }
   
   private class SampleHandler implements RequestHandler {
   
       @Override
       public Response process(Request request) throws Exception {
           return new SampleResponse();
       }
   }
   
   private class SampleResponse implements Response {
   }
   ```

   - 요청 응답 핸들러에 대한 객체를 만든다 (inner 객체)

   ```java
   @Test
   public void testAddHandler() {
     Request request = new SampleRequest();
     RequestHandler = new SampleHandler();
     controller.addHandler(request, handler);
     RequestHandler handler = controller.getHandler(request);
     assertSame("Handler we set in controller should be the same requestHandler we get", handler, requestHandler);
   }
   ```

   - request, handler를 생성 후 request 요청시 같은 핸들러 인지 테스트

   1. 정해진 상태로 테스트를 초기화
   2. 테스트 대상 메서드 호출
   3. 결과를 확인

5. 요청 처리하기

   ```java
   @Test
   public void testProcessRequest() {
      	Request request = new SampleRequest();
     	RequestHandler = new SampleHandler();
     	controller.addHandler(request, handler);
       Response response = controller.processRequest(request);
       assertNotNull("Must not return a null response", response);
       assertEquals(new SampleResponse(), response);
   }
   ```

   - 반환된 response null 체크하기

   상태 초기화 중복 제거

   ```java
      	Request request = new SampleRequest();
     	RequestHandler = new SampleHandler();
     	controller.addHandler(request, handler);
   ```

   ->객체 인스턴스로 초기화

   ```java
   @Before
       public void instantiate() throws Exception {
           controller = new DefaultController();
           request = new SampleRequest();
           requestHandler = new SampleHandler();
           controller.addHandler(request, requestHandler);
   
       }
   ```

6. testProcessRequest 개선하기

   ```java
   private class SampleResponse implements Response {
   
           private static final String NAME = "Test";
   
           public String getName() {
               return NAME;
           }
   
           public boolean equals(Object object) {
               boolean result = false;
               if (object instanceof SampleResponse) {
                   result = ((SampleResponse) object).getName().equals(getName());
               }
               return result;
           }
   
           public int hashCode() {
               return NAME.hashCode();
           }
       }
   ```

   - equals 구현후 assertEquals 적용하기

7. 예외 처리 테스트 하기

   ```java
       @Test(expected = RuntimeException.class)
       public void testGetHandlerNotDefined() {
           SampleRequest request = new SampleRequest("restNotDefined");
   
           controller.getHandler(request);
       }
   
       @Test(expected = RuntimeException.class)
       public void testAddRequestDuplicateName() {
           SampleRequest request = new SampleRequest();
           SampleHandler handler = new SampleHandler();
   
           controller.addHandler(request, handler);
       }
   ```

   > 예외 테스트도 읽기 쉽게 말들어라 + 가능한 모든 예외를 테스트하라
   >
   > expected 파라미터만으로도 개발자가 발생시키려는 예외가 무엇인지 명백히 해야함

8. 타임아웃 테스트

   ```java
       @Test(timeout = 130)
       @Ignore(value = "Ignore for now util we decided a decent time-limit")
       public void testProcessMultipleRequestsTimeout() {
           Request request;
           Response response = new SampleResponse();
           RequestHandler handler = new SampleHandler();
           for (int i = 0; i < 99999; i++) {
               request = new SampleRequest(String.valueOf(i));
               controller.addHandler(request, handler);
               response = controller.processRequest(request);
               assertNotNull(response);
               assertNotSame(ErrorResponse.class, response.getClass());
           }
       }
   ```

   - timeout = 130 : 130밀리초 
   - `@Ignore` 테스트를 건너 뛸때 - 반드시 이유를 명시해라!!



#### Hamcrest 라이브러리

```java
public class HamcrestTest {

    private List<String> values;

    @Before
    public void setUpList() {
        values = new ArrayList<String>();
        values.add("x");
        values.add("y");
        values.add("z");
    }

    @Test
    public void testWithoutHamcrest() {
        assertTrue(values.contains("one") || values.contains("two") || values.contains("three"));

    }

    @Test
    public void testWithHamcrest() {
        assertThat(values, hasItem(anyOf(equalTo("one"), equalTo("two"), equalTo("three"))));
    }

    //error 설명이 더 자세할 뿐만 아니라 코드 자체의 가독성이 뛰어남.
}
```

- 직관적이여서 보고 사용하는 데 쉽다.
- assert 실패시 설명이 디테일 함.

![image-20200211145154362](./img/junit-in-action_0.png)