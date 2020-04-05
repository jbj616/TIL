## Mock vs @Mock vs @MockBean

> https://www.baeldung.com/java-spring-mockito-mock-mockbean 번역

Spring Test Code를 작성하는 도중 Mock을 사용하므로써 단위테스트를 할 수 있어서 정말 새로운 코딩의 맛을 느꼈다. 하지만 사용하면서도 내가 제대로 사용하는 게 맞는 지 다른사람 예제를 통해서만 작성하기 때문에 힘들었다. 언제 @Mock을 사용하는지 @MockBean을 사용하는지 모르겠었다. 그래서 다음과 같은 자료 읽어보기로 했다.



### Mockito.mock()

`Mockito.mock()`메소드는, 우리가 클래스나 인터페이스의 목 객체를 생성할 수 있게 해준다. 생성된 mock를 사용해서 메서드의 반환값을 정의한다.

```java
@Test
public void givenCountMethodMocked_WhenCountInvoked_ThenMockedValueReturned() {
    UserRepository localMockRepository = Mockito.mock(UserRepository.class);
    Mockito.when(localMockRepository.count()).thenReturn(111L);
 
    long userCount = localMockRepository.count();
 
    Assert.assertEquals(111L, userCount);
    Mockito.verify(localMockRepository).count();
}
```

이 메소드는 사용하기 전에 다른 것이 필요하지 않다. 메소드를 사용하여 목 클래스 필드와 로컬 목을 만들 수 있다.



### Mockito @Mock

이것은 mock()의 약어이다. Test 클래스에서만 사용이 가능하며 mock() 메소드와 달리 활성화할 수 있는 어노테이션이 필요하다. 그것은 MockitoJUnitRunner 이며 MockitoAnnotations.initMock() 메소드를 명시적으로 호출하여 실행시킨다.

```java
@RunWith(MockitoJUnitRunner.class)
public class MockAnnotationUnitTest {
     
    @Mock
    UserRepository mockRepository;
     
    @Test
    public void givenCountMethodMocked_WhenCountInvoked_ThenMockValueReturned() {
        Mockito.when(mockRepository.count()).thenReturn(123L);
 
        long userCount = mockRepository.count();
 
        Assert.assertEquals(123L, userCount);
        Mockito.verify(mockRepository).count();
    }
}
```

@Mock은 코드를 더 읽기 쉽게 만드는 것 외에도 **필드 이름이 실패 메시지에 표시되므로 실패한 경우 문제 모의를 더 쉽게 찾을 수 있습니다.**

```java
Wanted but not invoked:
mockRepository.count();
-> at org.baeldung.MockAnnotationTest.testMockAnnotation(MockAnnotationTest.java:22)
Actually, there were zero interactions with this mock.
 
  at org.baeldung.MockAnnotationTest.testMockAnnotation(MockAnnotationTest.java:22)
```

또한 *@InjectMocks* 와 함께 사용하면 설정 코드의 양을 크게 줄일 수 있습니다.



### @MockBean

*@MockBean* 을 사용하여 Spring 애플리케이션 컨텍스트에 목 객체를 추가 할 수 있습니다. 목는 응용 프로그램 컨텍스트에서 동일한 유형의 기존 Bean을 대체합니다.

동일한 유형의 Bean이 정의되지 않은 경우 새 Bean이 추가됩니다. 이 주석은 특정 Bean (예 : 외부 서비스)을 모의화(?)해야하는 통합 테스트에 유용합니다.

이 주석을 사용하려면 *SpringRunner* 를 사용 하여 테스트를 실행해야합니다.

```java
@RunWith(SpringRunner.class)
public class MockBeanAnnotationIntegrationTest {
     
    @MockBean
    UserRepository mockRepository;
     
    @Autowired
    ApplicationContext context;
     
    @Test
    public void givenCountMethodMocked_WhenCountInvoked_ThenMockValueReturned() {
        Mockito.when(mockRepository.count()).thenReturn(123L);
 
        UserRepository userRepoFromContext = context.getBean(UserRepository.class);
        long userCount = userRepoFromContext.count();
 
        Assert.assertEquals(123L, userCount);
        Mockito.verify(mockRepository).count();
    }
}
```

 여기서는 주입 된 *UserRepository* 모의를 사용 하여 *count* 메소드를 스텁했습니다 *.* 그런 다음 애플리케이션 컨텍스트에서 Bean을 사용하여 실제로 모의 Bean인지 확인했습니다.



### 참고

JUnit를 사용할 때 적절한 방법

With Junit5:

```java

@ExtendWith(MockitoExtension.class)
public class MyTest {
….
}
```

With Junit4:

```java
public class MyTest {
@Rule
public MockitoRule rule = MockitoJUnit.rule();
….
}
```

