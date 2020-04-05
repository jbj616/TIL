## Spring Security Architecture 

> https://spring.io/guides/topicals/spring-security-architecture/ 번역

이 가이드는 Srping Security의 입문서로 프레임워크 설계 및 기본 빌딩 블록에 대한 통찰력을 제공합니다. 우리는 애플리케이션 보안의 기본적인 부분만 다루지만 Spring Security를 사용하는 개발자들이 경험 한 혼란을 해결할 수 있도록 할 것입니다. 이를 위해 필터를 사용하는 웹 어플리케이션보다 일반적으로 메서드 주석을 사용하여 보안이 적용되는 방식을 살펴 봅니다. 보안 어플리케이션의 작동 방식과 사용자 지정 방법을 이해해야하거나 어플리케이션 보안에 대한 생각을 보우고자 하는 경우 이 가이드를 읽으세요



### Authentication and access control

애플리케이션 보안은 두가지 독립적인 문제`authentication(Who are you?), authorization(what are you allowed to do) `로 요약됩니다. 때때로 사람들은 authorization대신 access control이라고 하여 혼란을 야기할 수 있습니다. Spring Security는 authorization과 authentication을 분리하도록 설계된 아키텍쳐를 가지며 둘 다 전략과 확장성을 가지고 있습니다. 

#### Authentication

인증을 위한 주요 전략 인터페이스는 `AuthenticationManager` 한가지이다

```java
public interface AuthenticationManager {

  Authentication authenticate(Authentication authentication)
    throws AuthenticationException;

}
```

`AuthenticationManager`는 `authenticate()` 3가지 방법중 하나를 수행한다

1. 입력이 유효한 주체를 나타내는 지 확인 할 수 있는 경우 `Authentication`(보통 `authenticated=true`)를 반환한다

2. 입력이 유효하지 않을 경유엔 `AuthenticationExeception` 에러는 던진다
3. 결정할 수 없을 땐 `return null`

`AuthenticationException`은 런타임 에러이다. 이것은 애플리케이션의 목적이나 스타일에 따라 일반적인 방법으로 애플리케이션에서 처리된다. 다시 말해, 사용자 코드는 일반적으로 이를 catch해서 다루지 않는다. 예를 들어 web UI 는 인증을 실패했다는 페이지를 렌더링하고 백엔드 HTTP 서비스는 `WWW-Authenticate`컨텍스트에 따라 헤더가 있거나 없는 401 error를 보낸다.

가장 흔한 `AuthenticationManager` 구현체인 `ProviderManager` 는 `AutehnticationProvider`의 한 체인을 대표한다 예를 들면 `AuthenticationProvider`은 `AuthenticationManager`와 약간 같지만 query를 부를 수 있도록하는 메서드를 가지고 있다

```java
public interface AuthenticationProvider {

	Authentication authenticate(Authentication authentication)
			throws AuthenticationException;

	boolean supports(Class<?> authentication);

}
```

`supports()`메서드의 `Class<?>` 인수는 `Class<? extends Authentication>`입니다.(`authenticate()`메서드에 전달되는 것이 지원하는 지 여부만 묻습니다.) `ProviderManager`는 `AuthenticationProviders`의 한 체인에 위임함으로써 같은 애플리케이션에서 서로 다른 인증 메커니즘을 지원합니다.  만약 `ProviderManager`가 특정한 `Authentication`인스턴스 유형을 을 인지하지 못하면 그것은 건너 뛸 것이다.

`ProviderManager`은 선택적인 부모를 가지며 모든 provider은 `null`을 반환한다면 동의 할 수 있다. 만약 부모를 이용할 수 없다면 `null` `Authentication` 결과는 `AuthenticationException`이다

때때로 애플리케이션은 보호된 리소스들의 논리적 그룹을 가지고있다(`api/**`경로 패턴에 일치하는 모든 web 리소스) 그리고 각 그룹은 자신들만의 `AuthenticationManager`를 가지고있다. 종종 이들 각각은 `ProviderManager`를 가지며 부모를 공유합니다. 이 부모는 일종의 전역 리소스가 되어 모든 공급자의 대체 역할을 합니다.

![](https://github.com/spring-guides/top-spring-security-architecture/raw/master/images/authentication.png)

An `AuthenticationManager` hierarchy using `ProviderManager`



#### Customizing Authentication Managers

Spring Security는 애플리케이션에 설정되어있는 공통 인증 관리자 기능을 빠리게 얻을 수 있는 configuration 도우미를 제공한다. 가장 일반적으로 사용되는 도우미는 `AuthenticationManagerBuilder`인 메모리, JDBC, LDAP 사용자 세부 사항을 설정하거나 사용자 정의`UserDetailsService`를 추가하는 데 유용합니다. 다음 `AuthenticationManager` 부모를 구성하는 예시입니다

```java
@Configuration
public class ApplicationSecurity extends WebSecurityConfigurerAdapter {

   ... // web stuff here

  @Autowired
  public void initialize(AuthenticationManagerBuilder builder, DataSource dataSource) {
    builder.jdbcAuthentication().dataSource(dataSource).withUser("dave")
      .password("secret").roles("USER");
  }

}
```

이 예제는 웹 어플리케이션과 관련있지만 `AuthenticationManagerBuilder`의 사용은 더 광범위하게 적용할 수 있다(아래에 더 많은 정보가 있다). `AuthenticationManagerBuilder`은 `@Bean`에서 `@Autowired`이다 - 이것은 `AutenticationManager` 전역으로 만든 것이다. 만약 우리가 다음과 같은 방벅을 사용한다면 대조적이다

```java
@Configuration
public class ApplicationSecurity extends WebSecurityConfigurerAdapter {

  @Autowired
  DataSource dataSource;

   ... // web stuff here

  @Override
  public void configure(AuthenticationManagerBuilder builder) {
    builder.jdbcAuthentication().dataSource(dataSource).withUser("dave")
      .password("secret").roles("USER");
  }

}
```

`@Override` 를 사용한다면 `AuthenticationManagerBuilder`은 local로써 `AuthenticationManager`를 빌드할때 사용된다 이는 글로벌의 child(하위)이다. 스프링 부트에서는 `@Autowired` 에서 전역으로 다른 Bean을 만들 수 있지만 명시적으로 직접 노출하지 않으면 로컬 Bean으로 할 수 없습니다.

스프링 부트는 `AuthenticationManager` type 빈을 제공하여 선점하지 않으면 (단지 한 사용자)에게 default global를 제공한다. 이 default는 사용자 정의 전역이 필요하지 않으면 자체적으로 충분히 안전하므로 걱정 할 필요가 없다. 빌드하는 구성을 수행하는 경우 `AuthenticationManager`를 보호하려는 리소스에 대해 로컬로 수행 할 수 있으며 전역 기본값에 대해 걱정하지 않아도 된다

#### 권한 부여 또는 액세스 제어

인증이 성공하면 권한 부여로 넘어갈 수 있으며 여기서 핵심은 `AccessDecisioManager`이다. 프레임워크에 의해 제공되는 구현체가 3가지 있으며 이 세가지은 위임은 `AccessDecisionVoter`의 체인에 위임되었다 `ProviderManager`와 `AuthenticationProviders`과의 관계처럼 

`AccessDecisionVoter`은 `Authentication`와 `ConfigAttributes`와 장식된 보안된 Object를 고려한다

```java
boolean supports(ConfigAttribute attribute);

boolean supports(Class<?> clazz);

int vote(Authentication authentication, S object,
        Collection<ConfigAttribute> attributes);
```

이 `Object`는 완전히 `AccessDecisionManager`와 `AccessDecisionVoter` 의 서명에서 generic이다 - 그것은 사용자가 액세스 할 수 있다는 것을 (웹 리소스 또는 Java 클래스의 메소드 가장 일반적인 두 가지 경우입니다)를 나타냅니다. `ConfigAttributes`도 상당히 일반적이며 보안 개체에 액세스하는 데 필요한 권한 수준을 결정하는 메타 데이터가있는 보안 개체의 장식을 나타냅니다. `ConfigAttribute`는 인터페이스이지만 매우 일반적인 하나의 메서드 만 있고 문자열을 반환하므로 이러한 문자열은 리소스 소유자의 의도를 어떤 식으로든 인코딩하여 누가 액세스 할 수 있는지에 대한 규칙을 나타냅니다. 일반적인 `ConfigAttribute`는 사용자 역할의 이름 (예 : ROLE_ADMIN 또는 ROLE_AUDIT)이며 특수한 형식 (ROLE_ 접두사와 같은)을 갖거나 평가해야하는 표현식을 나타냅니다.

대부분의 사람들은 `AffirmativeBased`인 기본 `AccessDecisionManager`를 사용합니다 (유권자가 긍정적으로 돌아온 경우 액세스 권한이 부여됨). 모든 사용자 정의는 유권자에게 발생하는 경향이 있으며, 새로운 투표를 추가하거나 기존의 투표 방식을 수정합니다.

스프링 표현식 언어 (SpEL) 표현식인 `ConfigAttributes`를 사용하는 것이 매우 일반적입니다 (예 :`isFullyAuthenticated () && hasRole ( 'FOO')`). 이는 표현식을 처리하고 이에 대한 컨텍스트를 작성할 수있는 `AccessDecisionVoter`에서 지원됩니다. 처리 할 수있는 식의 범위를 확장하려면 `SecurityExpressionRoot`의 사용자 지정 구현과 때로는 `SecurityExpressionHandler`가 필요합니다.

